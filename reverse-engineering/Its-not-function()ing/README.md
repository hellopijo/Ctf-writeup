## Overview

The file is 64-bit arm64 binary file so we can't run the program nor conduct dynamic analysis on it

<img width="540" height="336" alt="image" src="https://github.com/user-attachments/assets/e6e08d7f-fbbf-4c13-9c0b-9e20caea5fdb" />


So we run it on binary ninja to understand the flow

## Program Flow

__Main function__

<img width="540" height="300" alt="image" src="https://github.com/user-attachments/assets/b4b1f25d-acf3-40d7-ab3a-47767658acb8" />


It will takes user input and compare it with `nope` in this line of code:
```c
if (_strcmp(var_40, "nope") != 0)
```

Then it will print out `Nice try` if its identical and `Wrong key if different` and exit with value `0`.

The problam we see here, there's no function or any encoded text that that will print out flag. So there's another function that will print the flag but never been called. Then, i found another function called `_secret_function()`.

<img width="540" height="298" alt="image" src="https://github.com/user-attachments/assets/3b916997-773e-42b6-90ac-e9bf43c01aee" />

This is the cleanup version of this functions:

```c
    char buf[0x14 + 1];  // 20 bytes + null terminator

    strncpy(buf, "64%! '0!=0=<110;3942", 0x14);
    buf[0x14] = '\0';    // ensure null termination

    _decode(buf, 0x14);

    printf("FLAG: %s\n", buf);
```

So whats the function does is:
- copy encoded string into buf
- pass buf as arguments `_decode()` to modify the content of buf
- print out the modified content of buf which clearly a flag


<img width="522" height="300" alt="image" src="https://github.com/user-attachments/assets/f2f07cf0-421d-4000-995a-6fa32b6b5bf9" />

Clean it up and we get:

```c
void _decode(char *buf, int len)
{
    for (int i = 0; i < len; i++) {
        buf[i] ^= 0x55;
    }
}
```

This code very simple to understand, its executing XOR operation on every character with `0x55` to decode the buf content which is `64%! '0!=0=<110;3942`

## Solution

To solve it, we need to use the same logic as `_decoded()` function. So i write a simple python script:

<img width="540" height="310" alt="image" src="https://github.com/user-attachments/assets/9685c0fd-e83f-4a1c-984d-56d18f2e3212" />

## Flag

<img width="540" height="145" alt="image" src="https://github.com/user-attachments/assets/2c86d2d9-b393-4baa-9c30-79a757c361e4" />
