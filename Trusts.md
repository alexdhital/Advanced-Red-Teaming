# Trusts
In active directory trust is relationship between two domains or forest which allows users of one domain or forest to access resource of another domain or forest.

# Trusts direction

## One way trusts
Also known as one directional trust where users in trusted domain can access resource in trusting domain but the reverse is not true. for example there is forest product.com with two domains users.product.com and resources.product.com. If resources.product.com trusts users.product.com the resources.product.com in trusting domain and users.product.com is the trusted domain so users.product.com can access resources in resources.product.com since trusted can access trusting but trusting cannot access trusted here users in resources.product.com cannot access resources or computers in users.product.com.

## Two way trusts
Also known as bi-directional trust where users of both domain can access each other's resources. for example in forest product.com both users in users.product.com and resources.product.com can access each other's resources.

# Trust Transitivity
There are two types of trust transitivity they are transitive and non transitive.

## Transitive trust
It means if domain A trusts domain B and if domain B trusts domain C then domain A implicitly trusts domain C without establishing a direct trust relationship. for example in a forest alex.local there are three domains users.alex.local, management.alex.local and finance.alex.local. Here, if users.alex.local has bi-directional trust with management.alex.local and management.alex.local has bi-directional trust with finance.alex.local then users.alex.local automatically has bi-directional trust with finance.alex.local.
- This exists automatically in domains inside a forest.

## Non Transitive trust
It means if domain A trusts domain B and if domain B trusts domain C then domain A will not trust domain C and a separate trust would need to be created directly between them.
- This exists automaticlly between two domains in different forests.

# Parent-Child Trust
It is created automatically between parent and child domain and is always two way transitive. for example in a forest alex.local there are two domains finance.alex.local and users.finance.alex.local since users.finance.alex.local is child domain of finance.alex.local there is two way transitive trust between them meaning both users and resources in users.finance.alex.local and finance.alex.local can access each other's resource.

# Tree root Trust
It is also automatically created between trees in a forest.for example the forest root is alex.local there are three domains management.alex.local, finance.alex.local and helpdesk.alex.local. Here, helpdesk.alex.local has another child domain users.helpdesk.alex.local. Here if users.helpdesk.alex.local is compromised then attacker can compromise hepdesk.alex.local since it has two way transitive trust again from helpdesk.alex.local an attacker can compromise alex.local forest itself due to two way transitive trust.

# EXternal Trust
This needs to be explicitly defined. THis trust is manually established by enterprise administrator between domains in two two different forest. for example alex.local has finance.alex.local and management.alex.local and another forest dhital.local has users.dhital.local and admins.dhital.local here enterprise administrator can setup one way or two way trust between finance.alex.local and users.dhital.local. But even if users.dhital.local is compromised and attacker cannot compromise finance.alex.local and even if an attacker can access resource of finance.alex.local he/she needs to escalate his/her privilege on finance.alex.local to further move to alex.local or other parent/child domains inside alex.local forest.

# Get list of all domain trusts for current forest
```
Get-DomainTrust
``` 
# Get list of all domain trusts for current forest(AD module)
```
Get-ADTrust
```
# Map forest trust(Powerview)
```
Get-ForestTrust
```
# Map trust of another forest
```
Get-ForestTrust -Forest eurocorp.local
```
# Map trust of current forest(AD module)
```
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```
# Map trust of all domains of another forest
```
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```
# Enumerate all external trust of all domains in current forest
```
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```
# Note (Important)
If we have bi directional trust with another forest we can enumerate users, acls, gpo, logon, groups, computers everything in that forest.