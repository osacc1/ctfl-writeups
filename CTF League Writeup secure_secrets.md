# Fall Challenge 3: secure_secrets

**Category**: rev  
**Points**: 175  
**Description**:  
Oh boy, I sure hope nobody can crack my super secure and secret password vault!! I made sure to throw away the source code after compiling it, so clearly nobody can find the password inside.

---

# Approach

For this challenge we are given an ELF executable that prompts the user for a password. Once the correct password is entered, the program reads the flag from an inaccessible server-side file.

To solve this challenge I used Ghidra to decompile and reverse engineer the executable to figure out how it validates the password. 

# Process

My approach was to work backwards from the win condition, trying to understand what each section of logic was doing then labeling functions and variables accordingly.

The first thing I did was find the main function by identifying what the `entry` function calls. My next goal was to find the win condition. There is one `if` statement in the main function, that calls two functions. Examining both reveals that one prints the flag, while the other prints a fail message. From this I was able to label this section of logic.

```cpp
  isCorrectPassword = check_password(password);
  if (isCorrectPassword == '\0') {
    fail();
  }
  else {
    win();
  }
```

Next, I examined the `check_password` function. That random string looked suspicious, so I labeled it `key1`. I labeled the boolean it returns `isCorrectPassword` because we know if this function returns `true`, the win condition is met.

The `string_length_equal` and `strings_not_equal` were fairly straight forward to reverse engineer, so I have emitted their explanations. Note that `isPasswordLengthZero` is reused in the `if` statement to store the result of `strings_not_equal`.

After labeling everything we can, we are left with the following function:

```cpp
bool check_password(undefined8 password)

{
  char isPasswordLengthZero;
  bool isCorrectPassword;
  string mystery [32];
  undefined8 local_28;
  char *key1;
  
  key1 = "t_[lVLbGEDzID";
  local_28 = 0xd;
  isPasswordLengthZero = string_length_equals(password,0xd);
  if (isPasswordLengthZero == '\0') {
    mystery_function(mystery,password);
    isPasswordLengthZero = strings_not_equal(mystery,key1);
    isCorrectPassword = isPasswordLengthZero == '\0';
    std::__cxx11::string::~string(local_48);
  }
  else {
    isCorrectPassword = false;
  }
  return isCorrectPassword;
}
```

From this we can see that `mystery` must equal `key1` to validate the password.

Before we can understand what's happening here, we need to figure out what that `mystery_function` is doing. I have labeled the variables that can be derived from the functions logic, as well as the `construct_string` function.

```cpp
string * mystery_function(string *result,ulong password)

{
  byte *char;
  ulong string_len;
  string key2 [47];
  undefined1 local_29;
  undefined1 *local_28;
  ulong i;
  byte password_char;
  
  std::__cxx11::string::string(result);
  local_28 = &local_29;
  construct_string(key2,"7k5819Q4602x7",&local_29);
  FUN_0010307c(&local_29);
  i = 0;
  while( true ) {
    string_len = std::__cxx11::string::size();
    if (string_len <= i) break;
    char = (byte *)std::__cxx11::string::operator[](password);
    password_char = *char;
    std::__cxx11::string::size();
    char = (byte *)std::__cxx11::string::operator[]((ulong)key2);
    std::__cxx11::string::operator+=(result,*char ^ password_char);
    i = i + 1;
  }
  std::__cxx11::string::~string(key2);
  return result;
}
```

With the variables named, we are able to see that the `while` loop iterates over the length of `password` and `xor`s it with `key2`. We can now update our `check_password` logic:

```cpp
...
perform_xor(xor_result,password);
isPasswordLengthZero = strings_not_equal(xor_result,key1);
isCorrectPassword = isPasswordLengthZero == '\0';
...
```

Now we can see that we need a password that, when `xor`'d with `key2` equals `key1`. Because `xor` is a symmetric operation, to get such a password we can `xor` both keys.

`t_[lVLbGEDzID XOR 7k5819Q4602x7 = C4nTgu3sstH1s`

Entering this password gives us the flag, completing the challenge!

# Retrospect

The majority of the code in this challenge turned out to be irrelevant red herrings. By working backwards and only focusing on the logic directly related to the previous step, you can avoid confusing tangents (unlike me). Below is a graph of the important functions:

```
main -> check_password +-> perform_xor -> construct_string
                       +-> strings_not_equal
                       +-> string_length_equals
```

And the logic of `main`:
```cpp
undefined8 main(void)

{
  char isCorrectPassword;
  string password [32];
  string password_copy [40];
  
  welcome();
  get_input(password);
  isCorrectPassword = check_password(password);
  if (isCorrectPassword == '\0') {
    fail();
  }
  else {
    win();
  }
  std::__cxx11::string::~string(password);
  return 0;
}
```

My main struggle with this challenge was interpreting the decompiled C++ bultins. Once I got past their unsightlyness I found that what they were doing was actually quite simple. 