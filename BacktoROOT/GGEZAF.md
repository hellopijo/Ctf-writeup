# GGEZAF

In this challenge, ive given this IP `45.32.121.222`. My end goal is to gain root of this target server.

## Reconnaissance 

I run nmap to scan all open port for this server using command `nmap -p- –min-rate 5000 -Pn 45.32.121.222` and found 2 services open which ftp/21 and ssh/22

<img width="749" height="256" alt="image" src="https://github.com/user-attachments/assets/296d3ca5-f8f2-4161-866e-d64ee2cdb125" />

FTP (File Transfer Protocols) - FTP Server stores file and let user upload/download files over a network.
SSH (Secure Shell) - Protocol to let user securely connect to remote server/computer over internet.

## Exploit

Got into ftp server (username:anonymous, password: <blank>) and found creds.txt using `ls` command and download it using `get`

<img width="749" height="383" alt="image" src="https://github.com/user-attachments/assets/0d43e246-2774-45f8-ac95-4e241a28057f" />

Read the creds.txt and found a credential

<img width="753" height="222" alt="image" src="https://github.com/user-attachments/assets/8eaf94d4-ba4c-451f-81be-d6bc3299d430" />

Use the credential for ssh to the server and found info.txt file that shows “privesc to root to get the flag”

<img width="753" height="436" alt="image" src="https://github.com/user-attachments/assets/954d81ba-2ea5-4251-989b-4d20f0d231a1" />

`privesc` in this context means privilege escalation. We need to find a way to get the same privilege as root.

Run `sudo -l` to see whats privilege given for this user1337. 
We can see we can use `cat` and `ls` command

<img width="753" height="99" alt="image" src="https://github.com/user-attachments/assets/b0999cdf-11e8-4d30-be2a-debcf09d9418" />

We cant use cd. So we use `sudo ls root` to list up whats there then `sudo cat root/root.txt` to read the flag.

<img width="998" height="142" alt="image" src="https://github.com/user-attachments/assets/2efea104-6a11-4acb-bdaa-f7fbc11d3406" />


