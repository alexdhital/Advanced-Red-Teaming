# Configure collector(SharpHound)

- First use Invisi shell then again bypass .net AMSI for powershell using the provided AMSI bypass in lab manual then
```
. C:\AD\Tools\BloodHound-master\Collectors\SharpHound.ps1
```
```
Invoke-BloodHound -CollectionMethod All
```
OR

```
SharpHound.exe --CollectionMethods All
```
# Less detection
```
Invoke-BloodHound -Stealth
```
```
SharpHound.exe --Stealth
```
# Avoid MDI Detections
```
Invoke-BloodHound -ExcludeDCs
```
# NOTE:
Use compitable version of bloodhound and sharphound

# To-do
- Upload the data to bloodhound and check shortest path to domain admins, this should be done after doing priv esc on compromised machine and finding local admin access because latest version of bloodhound 4.2.0 has a bug where it doesnot show a user's local admin rights on another machine.