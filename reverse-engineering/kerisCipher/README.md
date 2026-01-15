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
### Python Script

```python
from pwn import *

# --- 1. Constants ---
target_blocks = [
    (0x9fa1d1b7, 0x4bd067a5),
    (0xd70102,   0x192cdd9e),
    (0x28a9f4f9, 0xd65b2be4)
]
K = [0xc5eb2561, 0xbfbf8c08, 0xde38dc58, 0x21323a44]
local_118 = [0x61, 0x25, 0xeb, 0xc5, 0x08, 0x8c, 0xbf, 0xbf, 0x58, 0xdc, 0x38, 0xde, 0x44, 0x3a, 0x32, 0x21]

def get_shuffle_map():
    uVar4 = 0x1337c0de
    perm = list(range(24))
    for i in range(23, 0, -1):
        # bitwise ops are cleaner with pwn's masked integers if needed, 
        # but standard python is fine here
        uVar4 = ((uVar4 << 13) ^ uVar4) & 0xFFFFFFFF
        uVar4 = (uVar4 ^ (uVar4 >> 17)) & 0xFFFFFFFF
        uVar4 = ((uVar4 << 5) ^ uVar4) & 0xFFFFFFFF
        idx = uVar4 % (i + 1)
        perm[i], perm[idx] = perm[idx], perm[i]
    return perm

def decrypt_block(v0, v1):
    delta = 0x61c88647
    sum_val = (-0x3910c8e0) & 0xFFFFFFFF
    for _ in range(32):
        v1 = (v1 - ((v0 << 4) + K[2] ^ (v0 >> 5) + K[3] ^ sum_val + v0)) & 0xFFFFFFFF
        v0 = (v0 - ((v1 << 4) + K[0] ^ (v1 >> 5) + K[1] ^ sum_val + v1)) & 0xFFFFFFFF
        sum_val = (sum_val + delta) & 0xFFFFFFFF
    return v0, v1

# --- Solving Logic ---
log.info("Starting decryption...")

# Step A: Decrypt Blocks using p32 (instead of struct.pack)
decrypted_bytes = b""
for v0, v1 in target_blocks:
    d0, d1 = decrypt_block(v0, v1)
    decrypted_bytes += p32(d0) + p32(d1) # p32 = pack 32-bit little endian

# Step B: Reverse Stream Cipher
bVar8 = 0x5a
unxored = []
for i in range(24):
    bVar10 = local_118[i & 0xf] ^ bVar8
    unxored.append(decrypted_bytes[i] ^ bVar10)
    bVar8 = (bVar8 + 7) & 0xFF

# Step C: Un-shuffle
perm = get_shuffle_map()
flag_middle = [0] * 24
for i in range(24):
    flag_middle[perm[i]] = unxored[i]

# Final Output using log.success
flag = "flag{" + "".join(chr(c) for c in flag_middle) + "}"
log.success(f"Found flag: {flag}")
```

## Flag

<img width="785" height="106" alt="image" src="https://github.com/user-attachments/assets/13152c65-d831-4d02-9e95-3553a39adfb5" />
