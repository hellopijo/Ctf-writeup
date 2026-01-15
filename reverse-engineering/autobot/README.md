## Overview

The file is 64-bit arm64 binary file so we can't run the program nor conduct dynamic analysis on it

<img width="540" height="126" alt="image" src="https://github.com/user-attachments/assets/8b7f98e7-2c5a-4ba2-b6ea-e616f3ba5e4f" />

So we run it on binary ninja to understand the flow

## Program flow

<img width="540" height="378" alt="image" src="https://github.com/user-attachments/assets/328bb7da-f897-4cf3-86c5-d48f1d111ca1" />

If we clean up the decompile code we got:

```c
{
    char input[0x40];
    const char expected[0x0f] = {
        0xb6, 0x25, 0x6c, 0x8b, 0xfa, 0xa1, 0x70, 0x4f,
        0x4f, 0x56, 0x2d, 0x24, 0x63, 0x7a, 0xa0
    };

    printf("Enter flag: ");
    fgets(input, sizeof(input), stdin);

    /* remove trailing newline */
    input[strcspn(input, "\n")] = '\0';

    if (strlen(input) != 0x0f) {
        puts("Wrong");
        return 0;
    }

    for (int i = 0; i < 0x0f; i++) {
        char transformed = sub_100003db8(input[i], (uint8_t)i);
        if (transformed != expected[i]) {
            puts("Wrong");
            return 0;
        }
    }

    puts("Correct");
    return 0;
}
```

A brief about this code:
 - first, it will check the length of user input to be exactly `0x0f` (15)
 - it loop through every character in user input and store sub_100003db8() result in `transformed`
 - if `transformed` is different with expected[] --> print wrong


<img width="540" height="148" alt="image" src="https://github.com/user-attachments/assets/96eff137-0ca8-4384-bad4-7a54158a5748" />


Cleanup and we got:

```c
uint8_t transform(uint8_t c, uint8_t idx)
{
    uint8_t x = c ^ (idx - 0x5b);

    return ((x >> 5) | (x << 3)) + idx * 7;
}
```

So whats this function does is:
 - XOR the input character with (idx - 0x5b)
 - performs an 8-bit left-rotation by 3  --> `((x >> 5) | (x << 3))` this is classic `ROL 3`
 - add idx*7

## Solution

Our goal is to reverse the logic in the `transform()` function to `expected[]` array.

```c
encoded = [0xb6, 0x25, 0x6c, 0x8b, 0xfa, 0xa1, 0x70, 0x4f, 0x56, 0x2d, 0x24, 0x63, 0x7a,0xe9,0xa0]

flag = ""
for idx, val in enumerate(encoded):
        x = (val - idx * 7) & 0xff
        x = ((x >> 3) | (x << 5)) & 0xff
        c = x ^ ((idx + 0xa5) & 0xff)

        flag += chr(c)

print(flag)
```

## Flag

<img width="406" height="59" alt="image" src="https://github.com/user-attachments/assets/f8017fef-192e-40d3-a6a3-b63e589695ac" />
