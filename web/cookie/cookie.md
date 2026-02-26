## Overview
<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/0d4d7057-e042-4f60-9728-6bb9aa60a757" />

This task provided link to the webpage with the button to become an admin

## Initial Analysis
<img width="600" height="463" alt="image" src="https://github.com/user-attachments/assets/96f904e1-b065-4b15-b2ce-3973a6951865" />

Open the website. The `Go to Admin Page` button redirect to the admin page but we dont have permission to do so. So what we can do, we use inspect `element` > `storage` > `cookies`.

<img width="1185" height="310" alt="image" src="https://github.com/user-attachments/assets/b391d141-b492-4264-8a62-dbb44bb11e4c" />

There’s a value in the cookies table. Its looks like base64 encoded text, so we will use [Cyberchef](https://gchq.github.io/CyberChef/) to solve it. 

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/bcaa519d-10bc-4c15-a8b0-1e8664c59a63" />

The decoded text shows its JWT (JSON Web Access Token ). It starts with the header, then the payload and the signature. The common attack we can use in JWT vulnerability is [JWT “none” algorithm Attack](https://portswigger.net/kb/issues/00200901_jwt-none-algorithm-supported).

So we can try to change the ‘alg’ to ‘none’ and then we can finally change the ‘role’ to ‘admin’. We also need to delete the signature at the last token because we already set the algorithm to none. The server follows the format Header.Payload.Signature.
So we need to adjust our new token by adding dot.

## Solution
To encode it back, we use the base64url encoding as i already tried the base64 and it failed. Noted that we need to to encode it separately.

<img width="493" height="726" alt="image" src="https://github.com/user-attachments/assets/057e397d-fad8-4e6c-b050-5b93c520409d" />

After get the encoded text, format it with the JWT format with dot in between payload and header and at the last token. Delete the extra padding ‘=’.

<img width="625" height="155" alt="image" src="https://github.com/user-attachments/assets/775d5d81-2f45-4b9e-a0c6-0519344e0532" />

Now we can put in the cookies and try to reload.

<img width="544" height="246" alt="image" src="https://github.com/user-attachments/assets/79871cb8-da1a-4458-b60c-c198e7421db7" />

## Flag
We already got the admin privilege and can go to admin page and gain the flag.

<img width="602" height="224" alt="image" src="https://github.com/user-attachments/assets/e3b11e71-0161-4093-a3cc-b132121e4843" />

### Notes

