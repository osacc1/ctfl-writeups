# Fall Challenge 4: locked_out

**Category**: crypto  
**Points**: 150 (3 flags)  
**Description**:  
Some guy working for ottersecurity accidentally encrypted all his passwords using his company's "state of the art" encryption toolkit. Can you get his passwords back before he gets fired?

---

# Approach

We are given the source code for an encryption toolset and a command line history of it being used. Each flag corresponds to a different function from the code, invoked in the command history.

This challenge focuses on encoding formats and chaining together encoding/decoding operations. https://cyberchef.io is very useful for this.

# Flag 1

```bash
$ ./otterset -e jumble $FLAG1
```

Inspecting the `jumble` function, we can see that it encodes the input a number of times in different formats. Specifically hexadecimal, octal, decimal, base32, and base64. The code is obfuscated, and instead of attempting to trace the code I took a more vibes-based approach.

The function applies a base64 encode right at the end, so I started with a base64 decode. From there I used my knowledge of what the formats look like to decide what to apply next. For example, after base64 decoding the data it looks like this:

```
34 65 20 34 37 20 35 35 20 ...
```

Out of the available encoding methods, I determined this was hexadecimal because it consists only of pairs (decimal and octal can have triplets) and digits (base32 and base64 contain all letters of the alphabet). Using a similar process of elimination, I went step by step until I reached the flag.

The decoding operations applied to the original encrypted data are below. Note that this is the reverse of the order the encodings were performed in. 

```
encrypted data -> base64 -> hex -> hex -> base64 -> hex -> base64 -> hex -> flag
```

# Flag 2

```bash
$ ./otterset -e mixor $FLAG2
```

The `mixor` function starts with this conspicuous hint:

```cpp
// TODO: can be easily reversed with a hash cracker/rainbow table (fix that)
cout << "Using key with unique hash: " << MD5(key) << "\n";
```

Using a hash cracker such as https://crackstation.net we can retrieve the key: `P@ssw0rd`

The rest of the function body consists of obfuscated encoding like we saw in the `jumble` function, with the addition of the `xor` operator using the key. I took the same vibes-based approach as the last time. When the output didn't match any of the encoding formats, I `xor`'d the output with the key  until I was back in familiar waters. The result is as follows:

```
encrypted data -> base64 -> xor -> base64 -> xor -> decimal -> hex -> xor -> base64 -> base32
```

# Flag 3

```bash
./otterset -e twist $FLAG3
```

Comparing the `twist` function's signature with the variables it is actually called with reveals that the parameter names are misleading:

```cpp
// Actual variables passed:
//
//            flag_1        flag_2        flag_3         key_1
void twist(string in, string key_1, string key_2, string key_3)
{
    // State-of-the-art triple-parallel key encryption
    string out = _xor(in, key_1);
    out = hexadecimal(out);
    out = _xor(out, key_2);
    out = base_32(out);
    out = _xor(out, key_3);
    out = base_64(out);
    cout << "Encrypted data: " << out << "\n";
}

...

// Function call in main():
twist(flag_1, flag_2, flag_3, key_1);
```

Rewriting `twist` with accurate variable names gives us the following psuedo-function:

```cpp
1 twist(flag_1, flag_2, flag_3, key1) {
2     out = xor(flag_1, flag_2) //              |
3     out = hex(out)            //              v
4     out = xor(out, flag_3)    // flag_3 = xor(out, out)
5     out = base32(out)         //                   ^
6     out = xor(out, key1)      //                   |
7     out = base64(out)         //                   | 
8 }
```

Now, we don't have `flag_3` yet, so we can't perform the `xor` on line 4. The solution relies on the fact that `xor` is reversible, as in `A XOR B = C` and `A XOR C = B`. Using this fact, we can rewrite line 4 as `flag_3 = xor(out, out)` where the result of lines 2-3 are `xor`'d with the reverse of lines 5-7.

Because we have `flag_1`, `flag_2`, and `key1` already, the only information we're missing is `flag_3`. Working from both sides of line 4, we get the following:
```
topside:     flag_1 -> xor with flag_2 -> hex encode
bottomside:  encrypted data -> base64 decode -> xor with key1 -> base32 decode
flag:        xor(topside, bottomside)
```

# Retrospect

For flags 1 and 2, I first tried to reverse engineer the logic, however I found that it was much faster to decode the encrypted data step by step by recognizing the encoding formats. Here's a reference table of what the formats used in this challenge typically look like, adopted (stolen) from Ella's slides:

```
Binary:        01110010 01100001 01110111 01110010 00100000 01011000 01000100
Octal:         162 141 167 162 40 130 104
Decimal:       114 97 119 114 32 88 68
Hexadecimal:   72 61 77 72 20 58 44
Base32:        OJQXO4RALBCA====
Base64:        cmF3ciBYRA==
```

Flag 3 is a great example of why you should explore the entire source code before attempting to find a solution. Without knowing what variables the function was actually being passed, I wouldn't have been able to solve it. Working the logic out on paper was also extremely helpful.