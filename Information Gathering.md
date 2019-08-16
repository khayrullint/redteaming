# External scans
## nmap scans
Command | Description
------- | -----------
-sn | ping scan (disable port scan, assumes all hosts up)
-sV | version detection
-sC | run common scripts
-T4 | default timing: https://nmap.org/book/performance-timing-templates.html
-A | Aggressive and advanced version of -sV
--max-retries 1 | set the number of retries of connections to the port. Significantly increases scan time

## DNS Enumeration
Example: HTB\Bank machine
Tools:
* nslookup
* dnsrecon
* dig

## Wordlists
https://github.com/danielmiessler/SecLists

## [Tools - Web] Droopescan
https://github.com/droope/droopescan
CMS scanner
