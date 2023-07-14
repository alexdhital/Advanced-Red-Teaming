# AD powershell Module

### Get information about current domain, its child domain, its forest, current domain's domain controller and other info
```Get-ADDomain``` 
### Gets same info as above for another domain or forest in this case is forest.
```Get-ADDomain -Identity moneycorp.local``` 
### Gets min/max password age, min password length, Clear text password, complexicity, etc.
```Get-ADDefaultDomainPasswordPolicy``` 
### Gets info like Max ticket age, max service age, max renew age, etc. Here in attacks like golden ticket we may want to keep max ticket or max renew age as default shown in the output to bypass defense.(Some detection tools check this if max ticket or renew age is greater than default domain policy).
```Get-ADDefaultDomainKerberosPolicy``` 
### password policy for chosen domain
```Get-ADDefaultDomainPasswordPolicy -Identity "moneycorp.local"``` 
### kerberos policy for chosen domain forest in this case
```Get-ADDefaultDomainKerberosPolicy -Identity "moneycorp.local"```
### Get the name and ip address of the domain controller
```Get-ADDomainController``` 
### Gets the ip address and domain controller of another domain in this case forest
```Get-ADDomainController -DomainName moneycorp.local -Discover``` 

# User Enumeration

### Get all users with their default properties
```Get-ADUser -Filter *```
### Get all users with all properties
```Get-ADUser -Filter * -Properties *```
### Gets information about specific user with all its properties
```Get-ADUser -Identity student1 -Properties *```
### Get usernames of all domain users and their logoncount and passwordlastset
```Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset}}}```
### Search for particular string in a user's attribute
```Get-ADUser -Filter 'Description -like "*pass*"' -Properties Description | select name,Description```

# Get Domain Admins
```
Import-Module ActiveDirectory
$domainAdmins = Get-ADGroupMember -Identity "Domain Admins" | Get-ADUser
$domainAdmins

```

# Computer Enumeration

### Get all computer objects
```Get-ADComputer -Filter * | select Name```
### Get all computer objects with all properties
```Get-ADComputer -Filter * -Properties *```
### Get computer objects with specific os
```Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name,OperatingSystem```
### Get domain computer and check if it can be reached or not
```Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}```

# Group Enumeration

### Get all groups in current domain
```Get-ADGroup -Filter * | select Name```
### Get all groups with all properties
```Get-ADGroup -Filter * -Properties *```
### Get all groups of target domain
```Get-ADGroup -Filter * -SearchBase "DC=moneycorp,DC=local"```
### Get all groups containing word admin in group name
```Get-ADGroup -Filter 'Name -like "*admin*"' | select Name```
### Get all members of Domain Admins group
```Get-ADGroupMember -Identity "Domain Admins" -Recursive```
### Get group membership for a user
``` Get-ADPrincipalGroupMembership -Identity student1 ```
### List all local groups on a machine(needs administrator priv on non-dc machines)
```Get-ADLocalGroup -Server dcorp-dc```
### Get members of the above local group "Administrators" on a machine(needs administrator priv on non-dc machines)
```Get-ADGroupMember -Identity "Administrators" -Server "dcorp-dc" | Select-Object Name, SamAccountName, ObjectClass```

# Logon Enumeration
This requires administrative rights on target computer

### Get actively logged users on a computer
```
$computerName = "dcorp-adminsrv"
$eventID = 4624  # Event ID for successful user logon

$loggedOnUsers = Get-WinEvent -ComputerName $computerName -FilterHashtable @{
    LogName = 'Security'
    ID = $eventID
} | ForEach-Object {
    $properties = $_.Properties
    $username = $properties[5].Value
    $domain = $properties[6].Value
    $user = "$domain\$username"
    $user
}

Write-Host "Actively logged on users on $computerName:"
$loggedOnUsers

```
### Get locally logged users on a computer
```
$computerName = "dcorp-adminsrv"

$loggedOnUsers = Get-WmiObject -Class Win32_ComputerSystem -ComputerName $computerName |
    Select-Object -ExpandProperty UserName |
    ForEach-Object {
        $username = $_ -replace '.*\\'
        Get-ADUser -Filter "SamAccountName -eq '$username'" -Properties DisplayName |
            Select-Object -ExpandProperty DisplayName
    }

Write-Host "Locally logged on users on $computerName:"
$loggedOnUsers

```
### Get last logged user on a computer

```
$computerName = "dcorp-adminsrv"

$lastLoggedOnUser = Get-WmiObject -Class Win32_ComputerSystem -ComputerName $computerName |
    Select-Object -ExpandProperty LastLoggedOnUser

Write-Host "Last logged-on user on $computerName: $lastLoggedOnUser"

```

# Forest Enumeration

### Get details about current forest
```
Get-ADForest
```
### Get details about another forest
```
Get-ADForest -Identity eurocorp.local
```
### Get all domains in the current forest
```
(Get-ADForest).Domains
```
### Get all domains in another forest
```
(Get-ADForest -Identity eurocorp.local).Domains
```

### Get all global catalogue for current forest
```
Get-ADForest | select -ExpandProperty GlobalCatalogs
```