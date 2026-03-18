# Bluemoon-2021 Lab Walkthrough (Kali Linux)
This document outlines the penetration‑testing exercise performed against the Bluemoon: 2021 virtual machine from VulnHub. The objective was to practice reconnaissance, exploitation, and privilege escalation techniques in a controlled environment using Kali Linux.

Lab Setup
- Target Hostname: BlueMoon
- Target IP: 192.168.56.101
- Operating System: Debian GNU/Linux 10
- Attacker Host: Kali Linux (192.168.56.102)
- Difficulty Level: Beginner (Boot2Root challenge)
- Goal: Obtain root access and capture the flag

1️ Reconnaissance
- Ran netdiscover to identify active hosts on the subnet.
- Verified attacker IP with ifconfig.
- Used nmap -sn 192.168.56.0/24 to perform a ping sweep and confirm the target machine’s presence.

2️ Service Enumeration
A detailed scan with nmap -sC -sV -Pn revealed:
- FTP (21/tcp) – vsftpd 3.0.3
- SSH (22/tcp) – OpenSSH 7.9p1
- HTTP (80/tcp) – Apache 2.4.38

3️ Web Exploration
- Used dirb with a custom wordlist to brute‑force directories.
- Discovered /hidden_text via gobuster.
- The hidden page contained a QR code image.
- Decoding the QR revealed FTP credentials:
userftp : fttp@ssword

4️ FTP Access
- Connected to FTP service and downloaded two files: information.txt and p_lists.txt.
- information.txt referenced a user named robin.
- p_lists.txt contained a custom password list for brute‑forcing.

5️ User Shell via SSH
- Used Hydra to brute‑force SSH with the provided wordlist:
hydra -l robin -P p_lists.txt ssh://192.168.56.101
- Valid credentials discovered:
robin : k4rv3ndh4nh4ck3r
- Logged in successfully with ssh robin@192.168.56.101.

6️ Privilege Escalation
Robin → Jerry
- Exploited feedback.sh with sudo rights, injecting /bin/bash.
- Switched to user jerry.
- Upgraded shell using Python:
python3 -c 'import pty; pty.spawn("/bin/bash")'
- Found jerry was part of the docker group.
Jerry → Root
- Verified Docker images were available.
- Mounted host filesystem inside a container:
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
- Escalated privileges to root.
- Retrieved final flag:
Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}
