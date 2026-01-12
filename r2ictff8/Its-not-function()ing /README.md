## Overview

The file is 64-bit arm64 binary file so we can't run the program nor conduct dynamic analysis on it

<img width="987" height="60" alt="image" src="https://github.com/user-attachments/assets/d8cb4d32-f274-4573-a5cf-45603b5f878e" />

So we run it on ghidra to understand the flow

## Program Flow

### 1) Entry function

This function works as a main function. 

<img width="458" height="365" alt="image" src="https://github.com/user-attachments/assets/58c2e89c-fc8a-47f2-b53a-29c38486a74b" />

It will takes user input and compare it with `nope` in this line of code:
```c
iVar1 = _strcmp(acStack_34,"nope");
```

Then it will print out `Nice try` if its identical and `Wrong key if different`.

The problam we see here, there's no function or any encoded text that that will print out flag.

### 2) _secret_function

As I scrolled up in symbol tree window in ghidra, there's suspicious function that's never called. 

<img width="269" height="259" alt="image" src="https://github.com/user-attachments/assets/0c479dd7-c5d9-492a-aa2c-cc84406b4b6b" />

