---
layout: post
title: TryHackMe | Net Sec Challenge
date: 2024-06-07 19:19 +0100
categories: [TryHackMe, Medium]
tags: [brute-force, challenge, ctf, ftp, infosec, nmap, ssh, net-sec-challenge, telnet, writeup]
toc: true
media_subpath: '/assets/img/posts/tryhackme-net-sec-challenge'

---

![Desktop View](thumbnail.png){: width="710" height="710" }


***

## Introduction

***

**Name: [Net Sec Challenge](https://tryhackme.com/r/room/netsecchallenge)**

**Creator: strategos**

**Release Date: 12-10-2021** 

**Difficulty: Medium**

This machine presents a series of questions that build upon knowledge gained in the network security module. Said questions test proficiency in Nmap, Telnet, and Hydra.

***

## Questions

***

### Question 1

We must identify the highest open port number, less than 10,000. We can accomplish this using the `-p-` flag.

```console
neighboursec@kali:~$ nmap -p- 10.10.204.203
Nmap scan report for 10.10.204.203
Host is up (0.017s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
8080/tcp  open  http-proxy
10021/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.09 seconds
```

Plain as day, the highest open port number less than 10,000 is port 8080. The meticulous hacker will note there is an unknown service running on port 10021, and they are right to do so as this is a critical part of later challenges.

### Question 2

To answer this question, we can simply refer back to the scan we previously performed, where we found port 10021 open. As it's both an uncommon port, and above 10,000 it fits the bill, and is the answer to this question.

### Question 3

Again, refer to the previous scan, where we can see there are 6 ports open on TCP. 

### Question 4

There is a flag hidden in the header of the HTTP server running on port 80. Though we could uncover the flag using other tools, TryHackMe suggests we use Telnet. 

```console
neighboursec@kali:~$ telnet 10.10.204.203 80 
Trying 10.10.204.203...
Connected to 10.10.204.203.
Escape character is '^]'.
GET / HTTP/1.0

HTTP/1.0 200 OK                                                                                                                                                                                                                             
Vary: Accept-Encoding                                                                                                                                                                                                                       
Content-Type: text/html                                                                                                                                                                                                                     
Accept-Ranges: bytes                                                                                                                                                                                                                        
ETag: "229449419"                                                                                                                                                                                                                           
Last-Modified: Tue, 14 Sep 2021 07:33:09 GMT                                                                                                                                                                                                
Content-Length: 226                                                                                                                                                                                                                         
Connection: close                                                                                                                                                                                                                           
Date: Wed, 05 Jun 2024 18:39:43 GMT                                                                                                                                                                                                         
Server: lighttpd THM{FLAG-REDACTED}                                                                                                                                                                                                      
                                                                                                                                                                                                                                            
<!DOCTYPE html>                                                                                                                                                                                                                             
<html lang="en">                                                                                                                                                                                                                            
<head>                                                                                                                                                                                                                                      
  <title>Hello, world!</title>                                                                                                                                                                                                              
  <meta charset="UTF-8" />                                                                                                                                                                                                                  
  <meta name="viewport" content="width=device-width,initial-scale=1" />                                                                                                                                                                     
</head>                                                                                                                                                                                                                                     
<body>                                                                                                                                                                                                                                      
  <h1>Hello, world!</h1>                                                                                                                                                                                                                    
</body>                                                                                                                                                                                                                                     
</html>                                                                                                                                                                                                                                     
Connection closed by foreign host.     

```

Note that you will have to type `GET / HTTP/1.0` followed by the enter key, to receive a response from the host.

### Question 5

Another flag hides in the SSH server header. We can perform an Nmap scan on port 22 with the `-sV` flag to find it.

```console
neighboursec@kali:~$ nmap -sV -p 22 10.10.204.203                                                                                                                                                                    
Nmap scan report for 10.10.204.203                                                                                                                                                                                                          
Host is up (0.015s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     (protocol 2.0)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.94SVN%I=7%D=6/5%Time=6660B305%P=x86_64-pc-linux-gnu%r(NU
SF:LL,29,"SSH-2\.0-OpenSSH_8\.2p1\x20THM{FLAG-REDACTED}\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.27 seconds

```
### Question 6

Similarly to the previous question, we can perform an Nmap scan with the `-sV` flag to determine the service version running on port 10021.

```console
neighboursec@kali~$ nmap -sV -p 10021 10.10.204.203
Nmap scan report for 10.10.204.203
Host is up (0.015s latency).

PORT      STATE SERVICE VERSION
10021/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.23 seconds
```
Aha, the service version running on this port is `vsftpd 3.0.3`.

### Question 7

This question has a tad few more steps involved in finding our flag, but it is nothing to be feared. Firstly, we'll create a wordlist with the usernames eddie and quinn, like so:

```console
neighboursec@kali~$ echo -e  "eddie\nquinn" >>  usernames.txt 
```

Now we want to use hydra to brute force the logins for these users on the FTP server, we'll use the usernames.txt file we just created for our list of users, and rockyou.txt as our password dictionary.

```console
hydra -L usernames.txt -P /usr/share/wordlists/rockyou.txt ftp://10.10.204.203:10021                                                                                                                                                    
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-05 20:41:57
[DATA] max 16 tasks per 1 server, overall 16 tasks, 28688798 login tries (l:2/p:14344399), ~1793050 tries per task
[DATA] attacking ftp://10.10.204.203:10021/
[10021][ftp] host: 10.10.204.203   login: eddie   password: PASSWORD-REDACTED
[10021][ftp] host: 10.10.204.203   login: quinn   password: PASSWORD-REDACTED
1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-06-05 20:42:30

```
Passwords in hand, we can now login to the server to find our flag.

```console
neighboursec@kali:~$ ftp eddie@10.10.204.203 10021                                                                                                                                                                                                          
Connected to 10.10.204.203.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||30016|)
150 Here comes the directory listing.
226 Directory send OK.
ftp> 
```
Hm, it appears the flag is not in Eddie's directory. No matter, let's try logging in as Quinn instead.

```console
neighboursec@kali:~$ ftp quinn@10.10.204.203 10021                                                                                                                                                                                                           
Connected to 10.10.204.203.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||30411|)
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002           18 Sep 20  2021 ftp_flag.txt
226 Directory send OK.
ftp> get ftp_flag.txt
local: ftp_flag.txt remote: ftp_flag.txt
229 Entering Extended Passive Mode (|||30357|)
150 Opening BINARY mode data connection for ftp_flag.txt (18 bytes).
100% |***********************************************************************************************************************************************************************************************|    18       10.04 KiB/s    00:00 ETA
226 Transfer complete.
18 bytes received in 00:00 (0.93 KiB/s)
```

Perfect, we have the flag.

### Question 8 

In this last question, we must perform an Nmap scan that goes undetected by the IDS to get the final flag. Sometimes, this challenge does not work from a local machine, it didn’t work on mine. Therefore, you may need to switch to an attackbox.

![Desktop View](Screenshot_20240605_212503.png){: width="701" height="483" }

Though scans performed with fragmented packets and other evasion techniques go undetected, only a null scan works and unveils the flag.

```console
neighboursec@kali:~$ sudo nmap -sN 10.10.204.203 
Nmap scan report for ip-10-10-204-203.eu-west-1.compute.internal (10.10.204.203)
Host is up (0.00044s latency).
Not shown: 995 closed ports
PORT     STATE         SERVICE
22/tcp   open|filtered ssh
80/tcp   open|filtered http
139/tcp  open|filtered netbios-ssn
445/tcp  open|filtered microsoft-ds
8080/tcp open|filtered http-proxy
MAC Address: 02:5D:7F:94:CD:45 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 93.02 seconds
```

![Desktop View](Screenshot_20240605_214201.png){: width="673" height="434" }

**After a successful scan, we get the flag and complete The Net Sec Challenge!**

***

