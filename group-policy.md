# Group Policy objects

GPO are applied on organizational units. Group policy objects are used to manage and configure user and computer settings across all user and computers objects in a network at once. With a GPO, sysadmins can manage and configure applications, software operations, and user settings throughout an entire organization. 

- Group Policy Objects can be abused for various attacks like privesc, backdoors, persistance, etc.

# Get list of GPO in current domain(PowerView)
```
Get-DomainGPO
```
# Get list of GPO for your own computer
Here, seeing exact policy for any other computer is not possible even with admin privilege
```
Get-DomainGPO -ComputerIdentity dcorp-student1
```
- Use rsop.msc on command line to see exact set of policy setting for current machine.

# Get GPO which uses restricted groups useful for extracting interesting users
Restricted Groups are mostly used to manage local groups on workstations, member servers, and domain controllers. One of the features within GPOs is the ability to configure Restricted Groups. This allows administrators to control the membership of local groups on target computers through GPO settings.
```
Get-DomainGPOLocalGroup
```
- After getting the groups, query the members of these groups for interesting users. 

# Get users who are in local group of a machine using GPO
```
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
```
# Get machines where the given user is member of using GPO
```
Get-DomainGPOUserLocalGroupMapping -Identity student1 -Verbose
```
# Get Organizational Units in a domain
```
Get-DomainOU
```
# Get Organizational UNits in a domain using AD module
```
Get-ADOrganizationalUnit-Filter*-Properties*
```
# List all computers which is part of certain OU
```
(Get-DomainOU -Identity <OU-Name>).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```

# Since we might be interested to find out what are the GPO applied to certain OUs
- First find all the Ous => ```Get-DomainOU | select name```
- Then extract the cn from gplink, this is the policy name {...-...-...-...} => ```Get-DomainOU -Identity <OU-name>```
- See the GPO for that OU => ```Get-DomainGPO -Identity '{0D1C3BF-1F556-AF64-D999877JU}'```
- It is not possible to see exact GPO settings from the command line
