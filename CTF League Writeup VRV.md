# Fall Challenge 5: VRV

**Category**: pwn  
**Points**: 150 (3 flags)  
**Description**:  
My flawless masterpiece of an application for my sowtware engineering class is finally complete! All hail the glorious memory safe language, C!

---

# Approach

The application being exploited in this challenge is recipe builder. The three flags all require modifying a `Recipe` struct in unintended ways through unsafe input. Below is a table of the `Recipe` struct in-memory that will prove very useful.

```
+-----------+--------------------------+-----------+
| offset    | field                    | size      |
+-----------+--------------------------+-----------+
|   0 - 31  | char name[32];           | 32  bytes |
|  32 - 45  | const char author[14];   | 14  bytes |
|  46 - 49  | int32_t cook_time        | 4   bytes |
|  50 - 179 | char cuisine[28];        | 28  bytes |
|  78 - 205 | char ingredients[4][32]; | 128 bytes |
| 206 - 213 | int64_t rating;          | 8   bytes |
| 214 - 215 | int8_t ingredient_count; | 1   byte  |
+-----------+--------------------------+-----------+
```

The first issue I faced when approaching this challenge is the main menu. Each action is labeled with a number, but the program converts the user input to hexadecimal using `strtol()`. This means to interact with the menu you must convert to action number to hexadecimal, like so:

```
+-----+-----+--------------------------+
| int | hex | menu action              |
+-----+-----+--------------------------+
| 16  | 10  | Enter recipe information |
| 31  | 1f  | Add ingredient           |
| 33  | 21  | Remove ingredient        |
| 42  | 2a  | Print recipe             |
| 50  | 32  | Quit                     |
+-----+-----+--------------------------+
```

# Flag 1

The source code reveals that flag 1 will be printed along with the recipe if the `author` field is `"?"`.

```c
void print_recipe(struct Recipe* r){
    ...
    if(strcmp(r->author, "?") == 0){
        print_file("flag1.txt", 32);
    }
    ...
}
```

Problematically, the program does not allow the user to change the `author` field. However, the code that gets user input for the `name` field is exploitable.

```c
// signature:
// void get_input(char* buffer, int size)
get_input(r->name, 34);
```

The `get_input()` function relies on the standard library function `fgets()`, which is vulnerable to a buffer overflow if the input is larger than the passed buffer size. This is exactly the case here, as the size passed is 34 (bytes), while the `name` field is 32 bytes. This means that when prompted to enter the recipe name, we can enter a string of 34 characters (each character is 1 byte) and 2 bytes will overflow into the next field, which is `author`.

To exploit this, I entered the following 33 characters (the last character of a char string is always null).

```
         name                      author      \ 
       32 bytes                   14 bytes      \
-------------------------------|-------------|   \   rest of memory
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?                /     not shown
---------------------------------|              / 
    get_input buffer                            \ 
          34 bytes                               \
```                                                  
                                                      
Now that we've managed to sneak in that '?' character, we can print the recipe which reveals flag 1.

> [!TIP]
> Use Python to craft your evil strings, much easier than using your fingers:   
> `print('A'*32 + '?')`

# Flag 2

To get our second flag, the following condition must be satisfied:

```c
if(r->cook_time == 1162559811){
    print_file("flag2.txt", 32);
}
```

The `Recipe` struct defines `cook_time` as an `int32_t`, which is 4 bytes (32 bits). If we try entering `1162559811` as the cook time, only the first three characters are accepted (remember the last character is null). In order to satisfy the `if` statement, we need to find some char string that C will interpret as that number.

To do this, we can first convert `1162559811` to hexadecimal, then convert the result to ASCII like so:

```
1162559811 -> 45 4b 41 43 -> EKAC
```

Because the `int32_t` is stored with the least-significant bit (the end) at the lowest memory address (this is called little-endian), we must reverse the resulting string. This gives us `CAKE`, which is suspiciously food related.

Now we know what we need to set `cook_time` to, but we can't just input `CAKE` because the program converts user input to an integer using `strtol()`. To get around this, we must find another buffer overflow.

We can't overflow `name` again because the exploit we found only overflows 2 bytes and the `cook_time` field starts at the 46th byte.

There happens to be a bug in the `remove_item` function that allows us to decrement `item_count` past `0` to `-1`. Because `add_ingredient` uses `item_count` to `strcpy` the inputted new ingredient, it is vulnerable to a buffer overflow. When `item_count` is set to `-1`, adding a new ingredient with a length of 32 bytes will overwrite the 32 bytes that come before the `ingredients` field. To find what this will overwrite we can subtract the 32 bytes from the `ingredients` field's offset, which can be found in the `Recipe` struct table. From this we can see that we will overwrite the data from offset `46` to `78`, which includes the fields `cook_time` (4 bytes) and `cuisine` (28 bytes).

Perfect! Now the exploit is as simple as removing an ingredient, causing `item_count` to be `-1` then adding an ingredient with the value `CAKE`. Print the recipe and you have your second flag.

```
\   cook_time      cuisine                     ingredients[0]            \   
 \  4 bytes        28 bytes                       32 bytes                \
 /     ---|---------------------------|-------------------------------|   /
/      CAKE                                                              / 
\      -------------------------------|                                  \ 
 \             ingredients[-1]                                            \
  \                32 bytes                                                \
```

# Flag 3

The third flag requires the following conditions to be satisfied:

```c
if(r->rating == 2397971403551886657 && r->ingredient_count == 67){
    print_file("flag3.txt", 32);
```

We can use the same strategy from flag 2 to find that we need to set `rating` to `AMAZING!` and `ingredient_count` to `C`. These fields occupy a combined 9 bytes directly following the `ingredients` field. Since there is no (intended) way to arbitrarily set either field from the program, we will need to cause a buffer overflow in `ingredients` again, but in the other direction this time.

Inspecting the `add_ingredient` function reveals this suspicious section:

```c
char new_ingredient[0x32];
printf("\nEnter a new ingredient: ");
get_input(new_ingredient, sizeof(new_ingredient));
strcpy(r->ingredients[r->ingredient_count], new_ingredient);
```

`new_ingredient` is defined with a size of `0x32`. The `0x` prefix hints that it's a hexadecimal number, meaning the buffer is actually 50 bytes long! `ingredients` are supposed to be 32 bytes long, giving us 18 bytes of overflow; more than enough to overwrite `rating` and `ingredient_count`.

Set the first three `ingredients` to whatever your heart desires. Now craft an evil string of 32 bytes to fill the final `ingredients` field, add your `AMAZING!` 8 bytes to overwrite `rating`, and add a final byte `C` to set the `ingredient_count` to a very humorous number. Now discover an off-by-one error and decrement the count by one. The final flag will now appear when printing the recipe. yay!

```
\            ingredients[4]        rating  ingredient_count
 \              32 bytes           8 bytes 1 byte
 / -------------------------------|-------||
/  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxAMAZING!C
\  -------------------------------------------------|
 \               new_ingredient
  \                 50 bytes
```

# Retrospect

Writing out the space that each `Recipe` field in memory was incredibly helpful in this challenge and saved me lots and lots of time. Once I understood how the memory was laid out, the rest was relatively easy (although it did take me a while to find the bug in `remove_item`).

Once again never underestimate the power of writing things down on paper.

An understanding of the C standard library functions used in this challenge was also critical. I found reading documentation for functions I wasn't familiar with, specifically `strtol`, very helpful.

Never underestimate the power of reading the docs.
