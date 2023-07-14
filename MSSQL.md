# MSSQL Servers

MS SQL servers are generally deployed in plenty in a Windows domain.SQL Servers provide very good options for lateral movement as domain users can be mapped to database roles. https://github.com/NetSPI/PowerUpSQL

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```
```
Import-Module C:\AD\Tools\PowerUpSQL-master\PowerUpSQL.psd1
```
## Get SQL instances
```
Get-SQLInstanceDomain -Verbose
```
## Check if we can connect on any of the instances
```
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```
## Gather information on the accessible instances
```
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

### MSSQL Servers Database Links
A database link allows a SQL Server to access external data sources like other SQL Servers and OLE DB data sources.In Microsoft SQL Server, a database link refers to a feature called Linked Servers. A Linked Server allows SQL Server to establish a connection to another database server, regardless of whether it is also a SQL Server or a different database platform. This feature enables SQL Server to access and query data from remote servers as if they were local. In case of database links between SQL servers, that is, linked SQL servers it is possible to execute stored procedures. Database links work even across forest trusts.

### Abusing Database LInks
Suppose there are three databases A B and C. If link between A and B was created using dbuser and link between B to C was created using sysadmin user and we have low level access(public user access) on database A then we would have public access on A dbuser access on B and sysadmin user on C. These database links can be between forests and can be very good source for lateral movement across forests.

- After finding the accessible instances we can look for links to remote servers from accessible server

```
Get-SQLServerLink -Instance dcorp-mssql -Verbose
```
OR run below on accessible instance(ex: dcorp-mssql)

```
select * from master..sysservers
```
