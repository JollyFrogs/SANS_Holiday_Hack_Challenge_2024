> Walk East, until you see the 'Elf Connect' terminal:
> 
> ![Pasted image 20241127184420](attachments/Pasted%20image%2020241127184420.png)
> 
> ### Silver Medal

Click on the 'Elf Connect' terminal

Right-click one of the green squares in the game and click 'Inspect' 

> This opens the Chrome Developer Console. 
> 
> The Chrome Developer console is docked in the right hand side by default. You should move it to the bottom by clicking the 3 vertical dots in the top right and choosing 'Dock to bottom' (refer red arrow below)
> 
> ![Pasted image 20241127195048](attachments/Pasted%20image%2020241127195048.png)

Click the 'Sources' tab

Then in the left-hand panel, expand 'top'

Expand 'hhc24-elfconnect.holidayhackchallenge.com/'

Expand 'hhc24-elfconnect.holidayhackchallenge.com'

Select '?&challenge=termElfConnect&username=...'

> ![Pasted image 20241127184641](attachments/Pasted%20image%2020241127184641.png)
> 
> The source-code has everything we need to get the correct answers.
> 
> The following PowerShell script will put together the correct answers by cross-referencing 'wordSets' with 'correctSets'.

Run this PowerShell script on your own Windows computer to get the correct answers:

```powershell
# Define word sets as hashtables
$wordSets = @{
    1 = @("Tinsel", "Sleigh", "Belafonte", "Bag", "Comet", "Garland", "Jingle Bells", "Mittens", "Vixen", "Gifts", "Star", "Crosby", "White Christmas", "Prancer", "Lights", "Blitzen")
    2 = @("Nmap", "burp", "Frida", "OWASP Zap", "Metasploit", "netcat", "Cycript", "Nikto", "Cobalt Strike", "wfuzz", "Wireshark", "AppMon", "apktool", "HAVOC", "Nessus", "Empire") 
    3 = @("AES", "WEP", "Symmetric", "WPA2", "Caesar", "RSA", "Asymmetric", "TKIP", "One-time Pad", "LEAP", "Blowfish", "hash", "hybrid", "Ottendorf", "3DES", "Scytale")
    4 = @("IGMP", "TLS", "Ethernet", "SSL", "HTTP", "IPX", "PPP", "IPSec", "FTP", "SSH", "IP", "IEEE 802.11", "ARP", "SMTP", "ICMP", "DNS")
}

# Define correct sets as array of arrays
$correctSets = @(
    @(0, 5, 10, 14),  # Set 1
    @(1, 3, 7, 9),    # Set 2 
    @(2, 6, 11, 12),  # Set 3
    @(4, 8, 13, 15)   # Set 4
)

function Display-AllSets {
    # Loop through each word set
    for($setNum = 1; $setNum -le 4; $setNum++) {
        Write-Host "`nWord Set $($setNum):"
        
        # Get all sets that use words from this wordSet
        $setGroups = @()
        for($i = 0; $i -lt 4; $i++) {
            $indices = $correctSets[$i]
            $words = $indices | ForEach-Object { $wordSets[$setNum][$_] }
            $setGroups += ,$words
        }
        
        # Display each group
        for($i = 0; $i -lt $setGroups.Count; $i++) {
            $group = $setGroups[$i]
            Write-Host "Group $($i + 1): $($group -join ", ")"
        }
    }
}

# Call the function
Display-AllSets
```

> ![](attachments/Pasted%20image%2020241130185346.png)
> 

Complete the game with the provided answers to complete the Silver challenge.

> Tip: Silver can also be completed by typing the following into the console:
> 
> ~~~javascript
> submitAction(1);
> ~~~
> 
> ### Gold Medal
> In the source-code of '(index)' or '?&challenge=termElfConnect&username=...', we spot the following code:
> 
> ![Pasted image 20241127185954](attachments/Pasted%20image%2020241127185954.png)
> 
> The code block checks if the score is larger than 50000, and if true, sets the highScoreText, explodes some fireworks, and runs function 'submitAction(2)', then displays the congratulatory message. 
> 
> To get Gold on this challenge, we can bypass the highscore check and manually submit 'submitAction(2);'.
> 
> To submit that function, the Chrome Developer Console tools needs to refer to the frame that contains our game . Click the 'top' dropdown and select 'hhc24-elfconnect.holidayhackchallenge.com/'
> 
> ![Pasted image 20241127185647](attachments/Pasted%20image%2020241127185647.png)

In Chrome's 'Console' tab, enter the following command:

```javascript
submitAction(2);
```

> ![Pasted image 20241127185807](attachments/Pasted%20image%2020241127185807.png)
> 
> This completes the Gold challenge.
> 
> Medal progress:
> 
> ![Pasted image 20241127202358](attachments/Pasted%20image%2020241127202358.png)
