> The Frosty Keypad is a prerequisite for The Hardware Hacking 101 - Part 1 challenge, so we'll complete Frosty Keypad first.
> 
> Walk North-East toward the 'Frosty Keypad'. Hidden behind the pile of boxes next to the 'Frosty Keypad' is a UV Flashlight. You can change the camera angle to reveal these hidden objects:
> 
> ![Pasted image 20241127234827](attachments/Pasted%20image%2020241127234827.png)
> 
> When you walk behind the pile of boxes, the UV Flashlight is revealed:
> 
> ![Pasted image 20241127235146](attachments/Pasted%20image%2020241127235146.png)
> 

Walk over the UV Flashlight to pick it up.

> Nearby, there is another item obfuscated behind a box - a Frosty Book:
> 
> ![Pasted image 20241127235831](attachments/Pasted%20image%2020241127235831.png)
> 
> The Frosty Book can be seen by moving East past the crate:
> 
> ![Pasted image 20241127235953](attachments/Pasted%20image%2020241127235953.png)
> 

Walk over the Frosty Book to pick it up.

> [!IMPORTANT]
> Check that you picked up both items by clicking your 'Snowball' -> 'Items':

> ![Pasted image 20241128000441](attachments/Pasted%20image%2020241128000441.png)

Click the 'Frosty Keypad' to start:

> ![Pasted image 20241128000617](attachments/Pasted%20image%2020241128000617.png)
> 
> ### Silver Medal
> 
> Immediately, we see a note on the Frosty Keypad with the following information:
> ~~~
>  2:6:1    4:19:3
>  6:1:1    3:10:4
>      14:8:3
> ~~~
> 
> ![Pasted image 20241128000654](attachments/Pasted%20image%2020241128000654.png)
> 
> Looking at the source-code of this challenge, we can see a reference to 'Ottendorf':
> 
> ![Pasted image 20241128000844](attachments/Pasted%20image%2020241128000844.png)

> [!TIP] Ottendorf is a book cipher. Morcel Nougat mentioned elves carrying a book when using the keypad, so it is likely that the note is using the Ottendorf book cipher, and that the book we found nearby is the code book.

Click your Avatar's snowball -> 'Items' -> Click 'Read the Book...'

> The book opens the following link in a new tab: https://frost-y-book.com/
> 
> The Ottendorf cipher can be read as x:y:z where x is the book's page number, y is the nth word on that page, and z is the nth letter of that word. For example 2:4:1 would be the first letter of the fourth word on the second page of the book.
> 
> Decoding the note using the Ottendorf cipher results in the word 'SANTA':
> 
> | Ottendorf code | Page, word and letter from Frosty Book | Decoded letter |
> | :-- | -- | -- |
> | 2:6:1 | Page 2, word 6 'SNUG', letter 1 | S |
> | 4:19:3 | Page 4, word 19 'WHAT', letter 3 | A |
> | 6:1:1 | Page 6, word 1 'NOW', letter 1 | N |
> | 3:10:4 | Page 3, word 10 'CLATTER', letter 4 | T |
> | 14:8:3 | Page 14, word 8 'TEAM', letter 3 | A |
> 
> 'SANTA' is 5 letters, and since the keypad accepts a maximum of 5 digits, it is possible that 'SANTA' is a phoneword. The phoneword system groups letters to digits on a phone keypad:
> 
> ![Pasted image 20241128001334](attachments/Pasted%20image%2020241128001334.png)
> 
> The word SANTA would be written as a phoneword using the number '72682'.

Enter code '72682' on the keypad to complete the Silver challenge:

> ![Pasted image 20241128001456](attachments/Pasted%20image%2020241128001456.png)
> 
> ### Gold Medal
> 
> When we open the 'Frosty Keypad' again, we now see a UV Flashlight. By shining the UV Flashlight over each key, we can see finger smudges on the digits 2,6,7 and 8.
> Based on the first code and the maximum keypad code input length, the second code likely also consists of 5 digits.
> Based on the smudges found with the UV Flashlight, each of the keys 2,6,7 and 8 is used at least once.
> 
> The following PowerShell script generates a list of all possible combinations and shows the number of possible permutations:

```
$digits = 2,6,7,8

$combinations = foreach ($d1 in $digits) {
    foreach ($d2 in $digits) {
        foreach ($d3 in $digits) {
            foreach ($d4 in $digits) {
                foreach ($d5 in $digits) {
                    $combo = "$d1$d2$d3$d4$d5"
                    # Check if all required digits are present
                    if (($combo.Contains("2")) -and 
                        ($combo.Contains("6")) -and 
                        ($combo.Contains("7")) -and 
                        ($combo.Contains("8"))) {
                        $combo
                    }
                }
            }
        }
    }
}
Write-Host "Total combinations:" $combinations.Count
```

> 
> ~~~
> Total combinations: 240
> ~~~
> 
> We can copy one request from the browser, and copy it into a script that will brute-forces all 240 combinations. 
 
Open the Chrome Developer tools by clicking 'Inspect' anywhere on the keypad screen, then click the 'Network' tab.
 
Enter any 5 digit code e.g. 26788 in the keypad and click the 'Enter' key on the keypad to get the browser to send a request:
 
> ![Pasted image 20241128003636](attachments/Pasted%20image%2020241128003636.png)
> 
> You should see one new request named 'submit?id=....' with a status code of '400'

Right-click the request and select 'Copy' -> 'Copy as PowerShell':

> ![Pasted image 20241128003707](attachments/Pasted%20image%2020241128003707.png)
> 
> Here is an example of what Chrome copies to clipboard:
> 
> ~~~
> $session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
> $session.UserAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
> $session.Cookies.Add((New-Object System.Net.Cookie("CreativeCookieName", "somevalue", "/", "hhc24-frostykeypad.holidayhackchallenge.com")))
> Invoke-WebRequest -UseBasicParsing -Uri "https://hhc24-frostykeypad.holidayhackchallenge.com/submit?id=myid" `
> -Method "POST" `
> -WebSession $session `
> -Headers @{
> "authority"="hhc24-frostykeypad.holidayhackchallenge.com"
>   "method"="POST"
>   "path"="/submit?id=myid"
>   "scheme"="https"
>   "accept"="*/*"
>   "accept-encoding"="gzip, deflate, br, zstd"
>   "accept-language"="en-GB,en-US;q=0.9,en;q=0.8"
>   "cache-control"="no-cache"
>   "origin"="https://hhc24-frostykeypad.holidayhackchallenge.com"
>   "pragma"="no-cache"
>   "priority"="u=1, i"
>   "referer"="https://hhc24-frostykeypad.holidayhackchallenge.com/?&challenge=termFrostyKeypad&username=myusername&id=myid&area=frontyardact1&location=59,19&tokens=&dna=ATATATTAATATATATATATTAGCATATATATATCGTAATATATATATATATGCATATATATATATATATTAATATCGATATATATATATATGCTAATATATATATATTACGATATCGGC"
>   "sec-ch-ua"="`"Google Chrome`";v=`"131`", `"Chromium`";v=`"131`", `"Not_A Brand`";v=`"24`""
>   "sec-ch-ua-mobile"="?0"
>   "sec-ch-ua-platform"="`"Windows`""
>   "sec-fetch-dest"="empty"
>   "sec-fetch-mode"="cors"
>   "sec-fetch-site"="same-origin"
> } `
> -ContentType "application/json" `
> -Body "{`"answer`":`"26788`"}"
> ~~~
> 
> Using relevant parts of the script above, we can create a script that brute-forces the correct code:

> [!TIP]
> The brute-force script works with even with an invalid cookie and id ¯\\\_(ツ)\_/¯

```
$digits = 2,6,7,8

# Create web session with cookie
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.UserAgent = "JollyFrogs PowerShell BruteForcer!"
$session.Cookies.Add((New-Object System.Net.Cookie("CreativeCookieName", "¯\_(ツ)_/¯", "/", "hhc24-frostykeypad.holidayhackchallenge.com")))

$combinations = foreach ($d1 in $digits) {
    foreach ($d2 in $digits) {
        foreach ($d3 in $digits) {
            foreach ($d4 in $digits) {
                foreach ($d5 in $digits) {
                    $combo = "$d1$d2$d3$d4$d5"
                    # Check if all required digits are present
                    if (($combo.Contains("2")) -and 
                        ($combo.Contains("6")) -and 
                        ($combo.Contains("7")) -and 
                        ($combo.Contains("8"))) {
                        $combo
                    }
                }
            }
        }
    }
}

Write-Host "Total combinations to try:" $combinations.Count

foreach ($code in $combinations) {
    Write-Host "Trying combination: $code"
    try {
        $response = Invoke-WebRequest -UseBasicParsing `
            -Uri "https://hhc24-frostykeypad.holidayhackchallenge.com/submit?id=1337" `
            -Method "POST" `
            -WebSession $session `
            -Headers @{
                "authority"="hhc24-frostykeypad.holidayhackchallenge.com"
                "method"="POST"
                "path"="/submit?id=1337"
                "scheme"="https"
            } `
            -ContentType "application/json" `
            -Body "{`"answer`":`"$code`"}"

        if ($response.StatusCode -eq 200) {
        
            Write-Host "SUCCESS! The correct code is: $code" -ForegroundColor Green
            if ($code -ne "72682") { break }
        }
    }
    catch {
        $statusCode = $_.Exception.Response.StatusCode.value__
        if ($statusCode -eq 400) {
            Write-Host "Failed with code: $code" -ForegroundColor Red
        }
        else {
            Write-Host "Error occurred with code $code : $statusCode" -ForegroundColor Yellow
        }
    }
    
    # Add a small delay to not overwhelm the server
    Start-Sleep -Milliseconds 1000
}
```

> A delay of 1000ms prevents server status code 429 errors (Too many requests).
> 
> Running the PowerShell script, we find the valid code in just 4 attempts:
> 
> ~~~
> Trying combination: 22678
> Failed with code: 22678
> Trying combination: 22687
> Failed with code: 22687
> Trying combination: 22768
> Failed with code: 22768
> Trying combination: 22786
> SUCCESS! The correct code is: 22786
> ~~~

Enter code '22786' on the keypad to complete the Gold challenge:

> ![Pasted image 20241128005202](attachments/Pasted%20image%2020241128005202.png)

> [!IMPORTANT]
> After completing the challenge, speak with Morcel Nougat to receive an item called
> 'One Thousand Little Teeny Tiny Shredded Pieces of Paper'

> Medal progress:
> 
> ![Pasted image 20241128005409](attachments/Pasted%20image%2020241128005409.png)