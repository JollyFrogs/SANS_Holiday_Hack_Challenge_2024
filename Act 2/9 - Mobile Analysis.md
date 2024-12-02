Click the snowball on your avatar 

  Select 'Story'
  
  Scroll down to 'ACT II' and click 'Go to Act II'

> ![](attachments/Pasted%20image%2020241201113646.png)

Walk a few steps West and talk to Eve Snowshoes

> Eve shares file download links for two files: SantaSwipe.apk and SantaSwipeSecure.aab
> 
> ![](attachments/Pasted%20image%2020241201113855.png)

Download https://www.holidayhackchallenge.com/2024/SantaSwipe.apk

Download https://www.holidayhackchallenge.com/2024/SantaSwipeSecure.aab

Download https://7-zip.org/download.html

Install the 7-Zip program on your computer

> [!WARNING] When downloading 7-zip make sure you use the correct link. The URI 7zip[dot]org uses an invalid certificate. The real website is 7-zip[dot]org not 7zip[dot]org. Additionally, there exists a critical vulnerability in 7-zip versions prior to version 24.07, in which the act of extracting a specially crafted, malicious compressed file can lead to compromise of your computer. 
> 
> Please stay safe and use safe, up-to-date versions of software on your computer.

> ### Silver Medal

On your computer, right-click 'SantaSwipe.apk' -> '7-Zip' -> 'Extract to "SantaSwipe\\"'

Navigate to https://www.decompiler.com/ and upload '.\SantaSwipe\classes3.dex'

Click 'MainActivity.java' to show the file's contents:

> ![](attachments/Pasted%20image%2020241201111542.png)
>
> ![](attachments/Pasted%20image%2020241201112009.png)
>
> The following line in the 'getNormalList()' function, excludes the user 'Ellie'
> 
> ~~~
> SELECT Item FROM NormalList WHERE Item NOT LIKE '%Ellie%'
> ~~~

Click on your avatar's snowball icon

  Select 'Objectives'

  Select 'Mobile Analysis'
  
  ```
  Ellie
  ```
  
  Click 'Submit'

> ![](attachments/Pasted%20image%2020241201120257.png)
> 
> This completes the silver challenge
> 
> ### Gold Medal
 
Right-click 'SantaSwipeSecure.apk' -> '7-Zip' -> 'Extract to "SantaSwipeSecure\\"'

Navigate to https://www.decompiler.com/ and upload '.\SantaSwipeSecure\base\dex\classes.dex'

  Click 'com'

  Click 'northpole/santaswipe'

  Click 'DatabaseHelper.java' to show its contents:

> ![](attachments/Pasted%20image%2020241201121722.png)
> 
> ![](attachments/Pasted%20image%2020241201123431.png)
> 
> We can see that SQL statements are stored in encrypted format, and decrypted when required.
> 
> The decryptData function is responsible for decrypting the data:
> 
> ![](attachments/Pasted%20image%2020241201123804.png)
> 
> The encryption uses type 'AES/GCM/NoPadding', using key 'this.secretKeySpec' with initialization vector 'this.iv'
> 
> We can trace the source of 'this.iv' to 'R.string.iv':
> 
> ![](attachments/Pasted%20image%2020241201124245.png)
> 
> We can trace the source of 'this.secretKeySpec' to 'R.string.ek':
> 
> ![](attachments/Pasted%20image%2020241201125120.png)
> 
> So what is 'R.string'? It is a reference to a string in a resource file such as an XML layout, or, in our case, a protobuf encoded resources file 'resources.pb'
> https://developer.android.com/guide/topics/resources/string-resource#String
>
> Opening the file 'SantaSwipeSecure\base\resources.pb' in a hex editor shows the layout of the file. Protobuf (.pb) files have a format that is described in accompanying .proto files. However, the format of the strings inside the protobuf file is easily identified as a length-prefixed string. The strings are prepended with the length of the string itself. For instance, 'ek' would is prepended with the byte 0x02 which identifies the length of the string.
>
> ![](attachments/Pasted%20image%2020241201140913.png)
> 
> The following PowerShell script searches for a specific length-prefixed string and shows the immediately following length-prefixed string:

```powershell
function Find-BinaryStrings {
    param (
        [Parameter(Mandatory=$true)]
        [string]$FilePath,
        
        [Parameter(Mandatory=$false)]
        [string]$SearchString = ""
    )

    try {
        # Read the file as bytes
        $bytes = [System.IO.File]::ReadAllBytes($FilePath)
        
        # Array to store all found strings with their positions
        $foundStrings = @()
        
        # Find all valid length-prefixed strings
        for ($i = 0; $i -lt $bytes.Length - 2; $i++) {
            # Find sequence of printable characters
            $maxLen = [Math]::Min(64, $bytes.Length - $i - 1)
            $printableLen = 0
            
            for ($j = 0; $j -lt $maxLen; $j++) {
                $b = $bytes[$i + 1 + $j]
                if ($b -lt 32 -or $b -gt 126) {
                    break
                }
                $printableLen++
            }
            
            if ($printableLen -gt 0) {
                # Check if byte before sequence is the length
                if ($bytes[$i] -eq $printableLen) {
                    $stringBytes = $bytes[($i + 1)..($i + $printableLen)]
                    $string = [System.Text.Encoding]::ASCII.GetString($stringBytes)
                    $foundStrings += @{
                        Position = $i
                        String = $string
                    }
                }
                # Check if first byte of sequence is the length of remaining sequence
                elseif ($bytes[$i + 1] -eq ($printableLen - 1)) {
                    $stringBytes = $bytes[($i + 2)..($i + $printableLen)]
                    $string = [System.Text.Encoding]::ASCII.GetString($stringBytes)
                    $foundStrings += @{
                        Position = $i + 1
                        String = $string
                    }
                }
                
                # Skip ahead past this sequence
                $i += $printableLen
            }
        }
        
        # Find search string and return next string
        if ($SearchString) {
            for ($i = 0; $i -lt $foundStrings.Count - 1; $i++) {
                if ($foundStrings[$i].String -eq $SearchString) {
                    Write-Host "Output:" $foundStrings[$i + 1].String
                    return
                }
            }
        }

    } catch {
        Write-Error "Error reading file: $_"
    }
}

Find-BinaryStrings -FilePath "$HOME\Desktop\HHC2024\SantaSwipeSecure\base\resources.pb" -SearchString "ek"

Find-BinaryStrings -FilePath "$HOME\Desktop\HHC2024\SantaSwipeSecure\base\resources.pb" -SearchString "iv"

```

> ![](attachments/Pasted%20image%2020241201151834.png)
> 
> The 'ek' and 'iv' strings are base64-encoded
> The base64-decoded value of 'ek' is 32 bytes, which matches the length of an AES key.
> The base64-decoded value of 'iv' is 12 bytes
> 
> We now have all we need to decrypt SQL statements used in the 'SantaSwipeSecure' app
> 
> One encrypted string that looks particularly interesting is the following code, which is used during the creation of the database:
> ~~~
> QLiteDatabase.execSQL(decryptData("IVrt+9Zct4oUePZeQqFwyhBix8cSCIxtsa+lJZkMNpNFBgoHeJlwp73l2oyEh1Y6AfqnfH7gcU9Yfov6u70cUA2/OwcxVt7Ubdn0UD2kImNsclEQ9M8PpnevBX3mXlW2QnH8+Q+SC7JaMUc9CIvxB2HYQG2JujQf6skpVaPAKGxfLqDj+2UyTAVLoeUlQjc18swZVtTQO7Zwe6sTCYlrw7GpFXCAuI6Ex29gfeVIeB7pK7M4kZGy3OIaFxfTdevCoTMwkoPvJuRupA6ybp36vmLLMXaAWsrDHRUbKfE6UKvGoC9d5vqmKeIO9elASuagxjBJ"));
> ~~~
>
>The following PowerShell script will decrypt the string:

```powershell
function Decrypt-AESMessage {
    param(
        [Parameter(Mandatory=$true)]
        [string]$EncryptedBase64
    )
    
    function Convert-FromBase64 {
        param([string]$base64)
        return [Convert]::FromBase64String($base64)
    }

    function Decrypt-AES-CTR {
        param(
            [byte[]]$key,
            [byte[]]$iv,
            [byte[]]$data
        )
        
        $aes = [System.Security.Cryptography.Aes]::Create()
        $aes.Mode = [System.Security.Cryptography.CipherMode]::ECB
        $aes.Padding = [System.Security.Cryptography.PaddingMode]::None
        $aes.Key = $key
        
        $counter = $iv.Clone()
        $encryptor = $aes.CreateEncryptor()
        $plaintext = New-Object byte[] $data.Length
        
        for ($i = 0; $i -lt $data.Length; $i += 16) {
            $keystream = $encryptor.TransformFinalBlock($counter, 0, 16)
            $blockSize = [Math]::Min(16, $data.Length - $i)
            
            for ($j = 0; $j -lt $blockSize; $j++) {
                $plaintext[$i + $j] = $data[$i + $j] -bxor $keystream[$j]
            }
            
            # Increment counter
            for ($j = 15; $j -ge 0; $j--) {
                $counter[$j]++
                if ($counter[$j] -ne 0) { break }
            }
        }
        
        $encryptor.Dispose()
        $aes.Dispose()
        
        return $plaintext
    }

    try {
        # Known values from the original script
        $key = Convert-FromBase64 "rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw="
        $iv = [byte[]]@(0x43,0x68,0x65,0x63,0x6B,0x4D,0x61,0x74,0x65,0x72,0x69,0x78,0x00,0x00,0x00,0x02)
        
        # Decode input ciphertext
        $ciphertext = Convert-FromBase64 $EncryptedBase64
        
        # Remove last 16 bytes (auth tag)
        $ciphertext = $ciphertext[0..($ciphertext.Length-17)]
        
        # Decrypt
        $plaintext = Decrypt-AES-CTR -key $key -iv $iv -data $ciphertext
        $result = [System.Text.Encoding]::UTF8.GetString($plaintext)
        
        return $result
    }
    catch {
        Write-Error "Decryption failed: $_"
        return $null
    }
}

$encrypted = "IVrt+9Zct4oUePZeQqFwyhBix8cSCIxtsa+lJZkMNpNFBgoHeJlwp73l2oyEh1Y6AfqnfH7gcU9Yfov6u70cUA2/OwcxVt7Ubdn0UD2kImNsclEQ9M8PpnevBX3mXlW2QnH8+Q+SC7JaMUc9CIvxB2HYQG2JujQf6skpVaPAKGxfLqDj+2UyTAVLoeUlQjc18swZVtTQO7Zwe6sTCYlrw7GpFXCAuI6Ex29gfeVIeB7pK7M4kZGy3OIaFxfTdevCoTMwkoPvJuRupA6ybp36vmLLMXaAWsrDHRUbKfE6UKvGoC9d5vqmKeIO9elASuagxjBJ"

$decrypted = Decrypt-AESMessage -EncryptedBase64 $encrypted
Write-Host $decrypted
```

> ~~~
> CREATE TRIGGER DeleteIfInsertedSpecificValue
>    AFTER INSERT ON NormalList
>    FOR EACH ROW
>    BEGIN
>        DELETE FROM NormalList WHERE Item = 'KGfb0vd4u/4EWMN0bp035hRjjpMiL4NQurjgHIQHNaRaDnIYbKQ9JusGaa1aAkGEVV8=';
>    END;
> ~~~
>
> We can decrypt the encrypted string in the DELETE statement:

```powershell
$encrypted = "KGfb0vd4u/4EWMN0bp035hRjjpMiL4NQurjgHIQHNaRaDnIYbKQ9JusGaa1aAkGEVV8="

$decrypted = Decrypt-AESMessage -EncryptedBase64 $encrypted
Write-Host $decrypted
```

> [!NOTE] 
> If you get an error: 'Decrypt-AESMessage : The term 'Decrypt-AESMessage' is not recognized as the name of a cmdlet, function, script file, or operable program.' then you should copy the previous command that adds the function to your PowerShell session.

> ~~~
> Joshua, Birmingham, United Kingdom
> ~~~

Click on your avatar's snowball icon

  Select 'Objectives'

  Select 'Mobile Analysis'
  
  ```
  Joshua
  ```
  
  Click 'Submit'

> This completes the Gold challenge.
> 
> Medal progress:
>
> ![](attachments/Pasted%20image%2020241201181431.png)
> 
> ### Bonus - Decrypting with CyberChef
> 
> The following CyberChef recipe uses CTR decryption and an IV padded with 0x00000002
> ~~~
> From_Base64('A-Za-z0-9+/=',false,false)
> Find_/_Replace({'option':'Regex','string':'(.){16}$'},'',true,false,false,false)
> AES_Decrypt({'option':'Base64','string':'rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw='},{'option':'Hex','string':'43 68 65 63 6b 4d 61 74 65 72 69 78 00 00 00 02'},'CTR','Raw','Raw',{'option':'Hex','string':'00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00'},{'option':'Hex','string':''})
> ~~~
>
> The following CyberChef recipe uses GCM decryption, takes the GCM authentication Tag from the end of the encrypted string, and feeds it into the GCM Tag field via a register:
> ~~~
> From_Base64('A-Za-z0-9+/=',true,false)
> To_Hex('None',0)
> Register('.*(.{32})',true,false,false)
> Find_/_Replace({'option':'Regex','string':'.{32}$'},'',true,false,true,false)
> AES_Decrypt({'option':'Base64','string':'rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw='},{'option':'Base64','string':'Q2hlY2tNYXRlcml4'},'GCM','Hex','Raw',{'option':'Hex','string':'$R0'},{'option':'Hex','string':''})
> ~~~