# Finding local admin access
From our compromised machine always check where do we have local admin access we can use powerview's below command to find all machines where our current user has local admin access.
```
Find-LocalAdminAccess -Verbose
```
- Find-WMILocalAdminAccess.ps1 and Find-PSRemotingLocalAdminAccess.ps1 powershell scripts can be used for the same purpose where above does not work.

# Find computers where a domain admin or specified user/group has sessions
This is same as finding local admin access on a computer since server 2019 local administrator privileges are required to list sessions and if we are able to list sessions it means we are local admin on that computer
```
Find-DomainUserLocation -Verbose
```
```
Find-DomainUserLocation -UserGroupIdentity "RDPUsers"
```
# Find computers where a domain admin session is available and current user has admin access
IT means a computer where a domain admin is actively interacting whether accessing some shares or service and has session established and our current user has local admin access to that computer
```
Find-DomainUserLocation -CheckAccess
```
# Find computers (File Servers and Distributed File servers) where a domain admin session is available.
```
Find-DomainUserLocation -Stealth
```