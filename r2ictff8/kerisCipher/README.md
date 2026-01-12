## Task

We found a small "authenticator" binary used in a strange toolchain. It only prints Correct! when the secret token is right. Recover the flag.

## Overview

The program prompt user to enter the flag and probably did some kind of compare mechanism

<img width="700" height="375" alt="image" src="https://github.com/user-attachments/assets/fee75606-41d6-41b2-b82a-fe84e823dafc" />

## Program Flow

### 1) Flag format check

The program will check the length of the input to meets desired criteria: 
  - Length: `0x1e (30)`
  - Starts with: `flag{`
  - Ends with: `}`

<img width="484" height="252" alt="image" src="https://github.com/user-attachments/assets/1fae06d6-0798-4bf6-b174-5da78867e3af" />

So the flag format will be:

    flag{XXXXXXXXXXXXXXXXXXXXXXXX}

### 2) Copy 24-byte 
This code copies the inner part of the flag:

<img width="184" height="55" alt="image" src="https://github.com/user-attachments/assets/7e7de46e-8ac9-4399-806a-b061e3ea6b8b" />

These three variables together hold 24 bytes (3 × 8 bytes).
They come from `acStack_89`, which was the input.
So the input will be like:
    
    flag[XXX] -> 24 bytes -> (local_108, uStack_100, local_f8)

### 3) Build shuffle table

First this initializes an array:

<img width="312" height="255" alt="image" src="https://github.com/user-attachments/assets/12e8441b-9c32-489b-b07d-597a08523b71" />

Then this loops shuffles it:

<img width="401" height="201" alt="image" src="https://github.com/user-attachments/assets/4e7df80b-3abc-4943-baf9-45e67ea4e395" />

This is [Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle) using a [xorshift](https://en.wikipedia.org/wiki/Xorshift) PRNG (Pseudo-Random Number Generator)
So now `local_e8` is a fixed permutation of `[0..23]`

### 4) XOR-based encryption with permutation

key is defined:

<img width="219" height="260" alt="image" src="https://github.com/user-attachments/assets/86ef33f5-ef30-468e-a807-3cece0990621" />

Encryption loop:

<img width="418" height="228" alt="image" src="https://github.com/user-attachments/assets/d8a9a940-ee88-460b-93c0-622de43fc95c" />

The same logic as:

```python
for i in 0..23:
    index = local_e8[i]
    key = key[i mod 16] XOR rolling_value
    rolling_value += 7
    
    output[i] = flag[index] XOR key
```

So the data is reordered by the shuffle table then XOR-encrypted by a stream cipher.
The result is stored in `local_c8`

### 5) TEA encryption

The encrypted bytes are treated as 6 blocks of 64-bit:

<img width="273" height="71" alt="image" src="https://github.com/user-attachments/assets/81133ece-ad4b-46b7-bff1-c14d8f915f06" />

Target ciphertext:

<img width="234" height="117" alt="image" src="https://github.com/user-attachments/assets/c382ca84-8411-4f65-aef0-f4e8e109493a" />

Encryption loop:

<img width="409" height="184" alt="image" src="https://github.com/user-attachments/assets/9f4b6549-d4ad-4189-9bf6-528e4cefcd82" />

This is [TEA](https://en.wikipedia.org/wiki/Tiny_Encryption_Algorithm) block cipher witth:

```c
k0 = 0xC5EB2561
k1 = 0xBFBF8C08
k2 = 0xDE38DC58
k3 = 0x21323A44
```
32 rounds.

### 6) Final check

After encrypting each 64-bit block:

<img width="363" height="31" alt="image" src="https://github.com/user-attachments/assets/e31842be-fdb0-43c8-8cd2-8835b67276ec" />

If all 6 blocks match:
```c
puts("Correct!");
```
Else:
```c
puts("Nope.");
```

### Summary
The binary check the flag like this:

    flag
    -> extract 24 bytes
    -> shuffle
    -> XOR stream cipher
    -> TEA encrypt
    -> compare to hardcoded ciphertext

To solve, i need to:

    ciphertext
    -> TEA decrypt
    -> XOR decrypt
    -> unshuffle
    -> get flag

## Solutions

