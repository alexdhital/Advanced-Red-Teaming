## Enumerate Current forest, domain, functional level, dc name
```
$ADClass = [System.DirectoryServices.ActiveDirectory.Domain]
$ADClass::GetCurrentDomain()
```
## Tools

Use microsoft AD module and dll in case if powershell has constrained language mode enabled.

 - Powerview => https://github.com/ZeroDayLab/PowerSploit/blob/master/Recon/PowerView.ps1
 - SharpView
 - ActiveDirectory powershell module => https://github.com/samratashok/ADModule
 - Microsoft signed DLL => https://github.com/samratashok/ADModule
 

## Find shares on hosts in current domain (Needs defense bypass modification)
https://github.com/darkoperator/Veil-PowerView/blob/master/PowerView/functions/Invoke-ShareFinder.ps1

```
Invoke-ShareFinder -Verbose
```
## Find sensative files on computers in the domain (Needs defense bypass modification)
https://github.com/gryhathack/PowerSploit_Sensitive_Info_Hunter/blob/25cf4f3d755fef1bf08e766a11e29b03a7d2d4b9/Invoke-FileFinder.ps1#L2
```
Invoke-FileFinder -Verbose
```
## Find all file servers of the domain (Needs defense bypass modification)
https://github.com/zloeber/PSAD/blob/fcf2936b79b5e49c99f09cea96fbafd26e6ecbf2/src/inprogress/Get-NetFileServer.ps1#L2

```
Get-NetFileServer
```
