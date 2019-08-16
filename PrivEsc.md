#Tools

## Windows
Watson: https://github.com/rasta-mouse/Watson
Sherlock: https://github.com/rasta-mouse/Sherlock (Example: HTB\Optimum)
PowerUp
https://github.com/frizb/Windows-Privilege-Escalation/blob/master/README.md

## Linux

### Enumeration scripts

- LinEnum.sh
- linuxprivchecker.py
- unixprivesc.sh

### Dirty Cow
Example: https://www.youtube.com/watch?v=XYXNvemgJUo&t=1430s

### Available sudo commands
user@server: sudo -l

### Checking mysql permissions
If we have creds for mysql we can run shell with mysql priveleges:
  @victimm > mysql -u root -p
  mysql > \! /bin/sh
  $ id

### Locate all binaries with setuid flag
find / -perm 4000 2>/dev/null | xargs ls -la

### Command hijacking
If we found command executed (by the script, for example) with higher privileges and the full path is not defined, we can run the file with the same name but from different location (HTB - Lazy).

### Editing /etc/passwd
If /etc/passwd is available for editing, we could simply add new privileged user.
Guide: https://www.hackingarticles.in/editing-etc-passwd-file-for-privilege-escalation/
