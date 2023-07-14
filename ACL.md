# Access Control Lists
It is security mechanism which is used to control access to resources such as files, folders, registry keys, printers, network shares and other active directory objects. ACLs define who can perform specific action on these resources and what level of access they have/can perform. ACLs are applied to these resources. ACLs consists of entries called ACE(Access Control Entries) that defines permission granted or denied to specific user, groups or computer objects. Each ACE(Access Control Entry) in the ACL contains SID(Security Identifier), DACL(Discretionary Access Control List ) and SACL(System Access Control List). 

## SID
A SID is a unique identifier assigned to each user, group or computer objects which uniquely identifies them. When configuring permissions, we typically specify the SIDs of the users, groups or computer objects that should be granted or denied access.

## DACL(Discretionary Access Control List)
When a user or group attempts to access an object, the DACL is evaluated to determine whether the access is allowed or denied. It determines the permissions like read, write, modify, Execute, delete, full permission a user, group or computer object has over a resource.

## SACL(System Access Control List)
IT is used for auditing purpose. When a user or group accesses a resource, the specified security events are logged, providing an audit trail for monitoring and tracking purposes.

# Get ACL for specified object(student1 in this case) 
```
Get-DomainObjectAcl -SamAccountName student1 -ResolveGUIDs
```
# Get ACL for a group
```
Get-DomainObjectAcl -Identity <GroupIdentity> -ResolveGUIDs
```
# Get ACL for Computer
```
Get-NetComputer -HostName <ComputerName> | Get-DomainObjectAcl -ResolveGUIDs
```
# Get ACL for network share
```
Get-NetShare -ComputerName <ComputerName> | Get-DomainObjectAcl -ResolveGUIDs
```
# Get ACL for OU
```
Get-DomainObjectAcl -Identity "OU=<OUName>,DC=<DomainComponent>" -ResolveGUIDs
```
# Get ACL for a domain
```
Get-DomainObjectAcl -Identity "DC=<DomainComponent>" -ResolveGUIDs
```
# Get ACL for file/folder
```
Get-NetFileServer | Get-NetShare -Name <ShareName> | Get-FileAcl
```
# Get ACL for registry key
```
Get-NetComputer -HostName <ComputerName> | Get-RegAcl -KeyPath <RegistryKeyPath>
```
# Get ACL for a service
```
Get-NetComputer -HostName <ComputerName> | Get-ServiceAcl -Name <ServiceName>
```
# Get ACL for a path
```
Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"
```
# Get Interesting ACE
Here, keep in mind about groups for example you have compromised user student1 and you see RDPUsers group has ACE as generic all on Support1User that means student1 has generic all privilege on Support1User and can abuse it to compromise Support1User since student1 is member of RDPUsers group. But using bloodhound is a reliable and faster way to find interesting ACEs.
```
Find-InterestingDomainAcl -ResolveGUIDs
```

