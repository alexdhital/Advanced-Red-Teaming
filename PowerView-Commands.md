# Powerview Domain Enumeration

### Get iformation about current domain, its child domain, its forest, current domain's domain controller and other info.
```Get-NetDomain``` 
### Gets same info as above for another domain or forest here in this case is forest.
```Get-NetDomain -Domain moneycorp.local```
### Gets info like Kerberos policy, System ACcess, Version, Registry Values, Unicode, etc.
```Get-DomainPolicy``` 
### Gets min/max password age, min password length, Clear text password, complexicity, etc.
```(Get-DomainPolicy)."System Access"```
### Gets info like Max ticket age, max service age, max renew age, etc. Here in attacks like golden ticket we may want to keep max ticket or max renew age as default shown in the output to bypass defense.(Some detection tools check this if max ticket or renew age is greater than default domain policy).
```(Get-DomainPolicy)."Kerberos Policy"``` 
### Gets policy for another domain, forest in this case.
```Get-DomainPolicy -domain moneycorp.local```
### same as above for password policy of another domain.
```(Get-DomainPolicy -domain moneycorp.local)."System Access"``` 
### Get the name and ip address of the domain controller
```Get-NetDomainController``` 
### Gets the ip address and domain controller of another domain in this case forest
```Get-NetDomainController -Domain moneycorp.local``` 

# User Enumeration

### Get all domain users with their default properties
```Get-DomainUser```
### Get information of a domain user with default properties
```Get-DomainUser -Identity student1``` 
### Get usernames of all domain users and their logoncount
```Get-DomainUser | select samaccountname, logoncount```
### Get information about a domain user with all properties
```Get-DomainUser -Identity student1 -Properties *```
### Search for particular string in a user's attribute
```Get-DomainUser -LDAPFilter "Description=*pass*" | select name,description"```

# Get Domain Admins
```
Get-NetGroupMember -GroupName "Domain Admins"
```

# Computer Enumeration

### Get all computer objects with all properties
```Get-DomainComputer```
### Get only the names of computers in a domain
```Get-DomainComputer | select Name```
### Get computer objects with specific os
```Get-DomainComputer -OperatingSystem "*server 2022*"```
### Get domain computer and check if it can be reached or not
```Get-DomainComputer -Ping```

# Group Enumeration

### Get all groups in current domain
```Get-DomainGroup | select Name```
### Get all groups in current domain with all properties
```Get-DomainGroup```
### Get all groups of target domain
```Get-DomainGroup -Domain moneycorp.local```
### Get all groups containing word admin in group name
```Get-DomainGroup *admin*```
### Get members of Domain Admins group
```Get-DomainGroupMember -Identity "Domain Admins" -Recurse```
### Get members of Enterprise Admins group
```
 Get-DomainGroupMember -Domain moneycorp.local -Identity "Enterprise Admins" -Recurse
```
### Get group membership for a user
```Get-DomainGroup -UserName "student1"```
### List all local groups on a machine(needs administrator priv on non-dc machines)
```Get-NetLocalGroup -ComputerName dcorp-dc```
### Get members of the above local group "Administrators" on a machine(needs administrator priv on non-dc machines)
``` Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators```

# Logon Enumeration
This requires administrative rights on target computer

### Get actively logged users on a computer
```Get-NetLoggedOn -ComputerName dcorp-adminsrv```
### Get locally logged users on a computer
```Get-LoggedonLocal -ComputerName dcorp-adminsrv```
### Get last logged user on a computer
``` Get-LastLoggedOn -ComputerName dcorp-adminsrv ```


# Forest Enumeration

### Get details about current forest
```
Get-Forest
```
### Get details about another forest
```
Get-Forest -Forest dhital.local
```
### Get all domains in the current forest
```
Get-ForestDomain
```
### Get all domains in another forest
```
Get-ForestDomain -Forest eurocorp.local
```
### Get all global catalogue for current forest
```
Get-ForestGlobalCatalog
```
### Get all global catalogue for another forest
```
Get-ForestGlobalCatalog-Foresteurocorp.local
```