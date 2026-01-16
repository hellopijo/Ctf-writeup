## Overview

This challenge provide me with pcap file that has been captured. We will open it in wirehsark for further investigation.

<img width="440" height="328" alt="Screenshot from 2025-12-27 14-58-28" src="https://github.com/user-attachments/assets/f7b0abb9-3976-4d80-9d00-4950a3a9ca60" />

## Initial Analysis

To clean up the noise, I go to `statistic > protocol hierarchy`

<img width="523" height="490" alt="Screenshot from 2025-12-27 15-01-26" src="https://github.com/user-attachments/assets/4df41b30-94fc-4939-bc11-96257ad8f4ab" />

This result tell us that this pcap file capture http files. HTTP can be considered insecure in today’s world because it lacks the security features provided by its successor, HTTPS.

Because this PCAP file captures HTTP traffic, it may contain readable data or files that are useful for solving this challenge.

So we moved to `file` at the top-left menu, `export objects > http`

<img width="530" height="245" alt="Screenshot from 2025-12-27 15-03-34" src="https://github.com/user-attachments/assets/b12fd051-1dc8-4305-a119-17d70e933ef8" />

The lists provided was all the captured file via http and we can access all of them by clicking `save`

This zip file obviously contain the flag in it

<img width="324" height="74" alt="Screenshot from 2025-12-27 15-05-32" src="https://github.com/user-attachments/assets/39717ff4-e601-41b2-a1e8-2c289ea9063c" />

But it require password to access it.

<img width="341" height="204" alt="Screenshot from 2025-12-27 15-06-19" src="https://github.com/user-attachments/assets/d4f6050e-8bbe-4bf7-b82f-c9b05ce81952" />

So we continue searching for any hints that probably the passwords. I found credential i need in login.php.

<img width="522" height="373" alt="Screenshot from 2025-12-27 15-07-07" src="https://github.com/user-attachments/assets/d429f069-3a7b-47b7-8bc1-5591f48d7b08" />

Extract the zip file and we got the flag.txt

<img width="309" height="114" alt="Screenshot from 2025-12-27 15-08-48" src="https://github.com/user-attachments/assets/c3ed85b5-d2a4-411d-b216-d77a042a18dc" />

## Flag

<img width="1001" height="198" alt="Screenshot from 2025-12-27 15-10-08" src="https://github.com/user-attachments/assets/38999b2d-a875-44bc-9bd9-0b3c199fe1d8" />

### Notes
Wireshark can capture HTTP traffic because it operates at the packet level and understands the HTTP protocol. Since HTTP data is unencrypted, Wireshark can reassemble TCP streams, parse HTTP headers, and extract transmitted files directly from the captured packets.


