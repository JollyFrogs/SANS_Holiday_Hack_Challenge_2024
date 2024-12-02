
Walk North-East and click the 'Snowball Showdown' poster:

> ![](attachments/Pasted%20image%2020241201161914.png)
> 
> This opens a new browser tab similar to: https://hhc24-snowballshowdown.holidayhackchallenge.com/?&challenge=termSnowballShowdown&username=your_name&id=your_id&area=frontyardact2&location=25,33&tokens=&dna=ATATATTAATATATATATATTAGCATATATATATCGTAATATATATATATATGCATATATATATATATATTAATATCGATATATATATATATGCTAATATATATATATTACGATATCGGC
> 
> ### Bronze Medal
>  
> To get the Bronze medal, play the game and win without using cheats. 
> Since getting the Silver medal unlocks the Bronze medal, too - we won't bother with it.
> 
> ### Silver Medal
> 
> To get the Silver medal, play the game and win by using cheats. in single player mode

> [!TIP] 
> Cheats include flying / large bullets / large explosions / fast walk / fast bullets / fast shoot / wall hack / aimbot / triggerbot / etc..

> We will use Chrome overrides to change the game's code to our advantage:
 
In Chrome, press F12 or right-click anywhere in the game and select 'inspect'

Click the 'Sources' tab

Click '?&challenge=termSnowballShowdown&username=...' in the left panel to reveals the code:

> ![](attachments/Pasted%20image%2020241201163741.png)
> 
> We can see the game is started using a new URL in the form:
> https://hhc24-snowballshowdown.holidayhackchallenge.com/game.html?username=your_username&roomId=room_id&roomType=room_type&id=your_id&dna=ATATATTAATATATATATATTAGCATATATATATCGTAATATATATATATATGCATATATATATATATATTAATATCGATATATATATATATGCTAATATATATATATTACGATATCGGC&singlePlayer=false
> 
> There is also a commented-out option 'gameType' which, when set to 'free-for-all' will enable PVP mode - we'll complete the challenge in non-PVP mode.

Click 'CREATE PRIVATE ROOM' inside the game to start a new game

> The game now loads the game logic javascript file 'phaser-snowball-game.js':
> 
> ![](attachments/Pasted%20image%2020241201170117.png)
>

> [!NOTE] 
> Refer to [03 - Elf Minder 9000](../_Prologue/03%20-%20Elf%20Minder%209000.md) on how to override javascript content. We will use the same method below, without screenshots:

Right-click 'phaser-snowball-game.js'

Click 'Override content'

Open a Windows PowerShell Terminal, and run the following script to load the 'windiff' function into your PowerShell session:

```powershell
function windiff {
    param (
        [Parameter(Position=0)]
        [string]$File1,
        
        [Parameter(Position=1)]
        [string]$File2,
        
        [Parameter()]
        [switch]$apply,
        
        [Parameter()]
        [switch]$backup = $true
    )

    # Function to create diff
    function Create-Diff {
        param($src, $dst)
        $output = New-Object System.Text.StringBuilder
        $output.AppendLine("--- $src") | Out-Null
        $output.AppendLine("+++ $dst") | Out-Null

        $srcLines = Get-Content $src
        $dstLines = Get-Content $dst

        for ($i = 0; $i -lt [Math]::Max($srcLines.Count, $dstLines.Count); $i++) {
            if ($i -ge $srcLines.Count) {
                $output.AppendLine("@@ -$i,0 +$($i+1),1 @@") | Out-Null
                $output.AppendLine("+$($dstLines[$i])") | Out-Null
                $output.AppendLine("") | Out-Null
            }
            elseif ($i -ge $dstLines.Count) {
                $output.AppendLine("@@ -$($i+1),1 +$i,0 @@") | Out-Null
                $output.AppendLine("-$($srcLines[$i])") | Out-Null
                $output.AppendLine("") | Out-Null
            }
            elseif ($srcLines[$i] -ne $dstLines[$i]) {
                $output.AppendLine("@@ -$($i+1),1 +$($i+1),1 @@") | Out-Null
                $output.AppendLine("-$($srcLines[$i])") | Out-Null
                $output.AppendLine("+$($dstLines[$i])") | Out-Null
                $output.AppendLine("") | Out-Null
            }
        }
        return $output.ToString()
    }

    # Function to apply diff
    function Apply-Diff {
        param($original, $diff)
        if ($backup) {
            Copy-Item $original "$original.bak"
        }

        $diffContent = Get-Content $diff
        $fileLines = Get-Content $original

        $i = 2  # Skip headers
        while ($i -lt $diffContent.Count) {
            if ($diffContent[$i] -match '@@ -(\d+),(\d+) \+(\d+),(\d+) @@') {
                $startLine = [int]$Matches[1] - 1
                $linesCount = [int]$Matches[2]
                
                $i++
                $changes = @()
                while ($i -lt $diffContent.Count -and -not ($diffContent[$i] -match '@@ ')) {
                    if ($diffContent[$i] -match '^[+-]') {
                        $changes += $diffContent[$i]
                    }
                    $i++
                }
                
                $newLines = @()
                foreach ($change in $changes) {
                    if ($change.StartsWith('+')) {
                        $newLines += $change.Substring(1)
                    }
                }
                
                $fileLines = $fileLines[0..($startLine-1)] + 
                            $newLines + 
                            $fileLines[($startLine + $linesCount)..($fileLines.Count-1)]
            }
            else { $i++ }
        }
        $fileLines | Set-Content $original
    }

    # Main logic
    if ($apply) {
        if (-not $File1) { Write-Error "Original file path required"; return }
        if (-not $File2) { Write-Error "Diff file path required"; return }
        Apply-Diff $File1 $File2
    }
    else {
        if (-not $File1) { Write-Error "Source file path required"; return }
        if (-not $File2) { Write-Error "Target file path required"; return }
        return Create-Diff $File1 $File2
    }
}
#
# Usage examples:
#
# windiff original.js modified.js > changes.diff
# windiff -apply original.js changes.diff
# windiff -apply -backup:$false original.js changes.diff
```

> In the same Terminal / PowerShell, paste the following contents to modify the overridden source file 'phaser-snowball-game.js':

```powershell
# Change to your Override directory
cd $HOME\Desktop\HHC2024\hhc24-snowballshowdown.holidayhackchallenge.com\js\

# Create the diff file
@"
--- .\phaser-snowball-game.js
+++ .\phaser-snowball-game.js
@@ -14,1 +14,1 @@
-        this.playerMoveSpeed = 150;
+        this.playerMoveSpeed = 450;

@@ -17,1 +17,1 @@
-        this.percentageShotPower = 0;
+        this.percentageShotPower = 100;

@@ -20,1 +20,1 @@
-        this.wombleyElvesThrowDelayMin = 1500;
+        this.wombleyElvesThrowDelayMin = 150000;

@@ -21,1 +21,1 @@
-        this.wombleyElvesThrowDelayMax = 2500;
+        this.wombleyElvesThrowDelayMax = 250000;

@@ -25,1 +25,1 @@
-        this.throwSpeed = 1000;
+        this.throwSpeed = 4000;

@@ -26,1 +26,1 @@
-        this.throwRateOfFire = 1000;
+        this.throwRateOfFire = 10;

@@ -112,1 +112,1 @@
-        let numOfWombleyElves = 10;
+        let numOfWombleyElves = 2;

@@ -679,1 +679,1 @@
-        this.snowBallBlastRadius = 24;
+        this.snowBallBlastRadius = 500;

@@ -680,1 +680,1 @@
-        this.onlyMoveHorizontally = true;
+        this.onlyMoveHorizontally = false;
"@ | Out-File -FilePath "phaser-snowball-game.js.diff" -Encoding UTF8

# Apply the diff. If this step fails, ensure that you have loaded the windiff function shared earlier
windiff -apply phaser-snowball-game.js .\phaser-snowball-game.js.diff
```

> Afted applying the diff, the browser should ask whether to apply the changes (Click 'OK' when it does), but if it does not, simply refresh the game website to apply the changes.
> 
> With the changes made, ensure that the browser shows the modified javascript code. If it does not, restart the game.
>
> Because we don't want to wait for other players to join our game, we will play single-player.

Change the link in the browser tab and set singlePlayer=true:

> ![](attachments/Pasted%20image%2020241201172443.png)

Press 'Enter' after updating the URL in the Chrome tab to reload the tab in single-player mode

Click 'Ready' and win the game with the cheats enabled

> This completes the Silver challenge.
> 
> ### Gold Medal
> 
> To get the Gold medal, win the game by using the 'secret weapon' that Dusty mentions.

Click the Poster again to start a new SnowBall Showdown game

Click 'CREATE PRIVATE ROOM'

Change 'singlePlayer=true' like we did before

Click 'Ready' in the game

Open the Chrome Console and enter the following command:

```javascript
window.mainScene.moasb()
```

> This summons the 'mother-of-all-snow-bombs' and completes the Gold challenge.
>
> ![](attachments/Pasted%20image%2020241201180259.png)
> 
> Medal progress:
> 
> ![](attachments/Pasted%20image%2020241201180856.png)