Walk East until you see the 'Elf Minder 9000' terminal:
 
> ![Pasted image 20241127190514](attachments/Pasted%20image%2020241127190514.png)
> 
> ### Silver Medal

Click the 'Elf Minder 9000' terminal

Right-click anywhere in the Elf Minder 9000 (e.g. the 'Sandy Start' button) and select 'Inspect'

Click the 'Sources' tab and then click on 'Overrides' tab in the left panel (refer red arrow below)

> ![Pasted image 20241127202905](attachments/Pasted%20image%2020241127202905.png)

Click 'Select folder for overrides' and choose any folder e.g. C:\Users\Username\Desktop\HHC2024\

> [!IMPORTANT]
> Click 'Allow' to allow DevTools to use the folder

> ![Pasted image 20241127203354](attachments/Pasted%20image%2020241127203354.png)

Still in the 'Sources' tab, in the left-hand panel click the 'Page' tab:

Expand 'top'

Expand 'index.html'

Expand 'hhc24-elfminder.holidayhackchallenge.com'

Right-click 'game2.js' and select 'Override content'

> ![Pasted image 20241127204126](attachments/Pasted%20image%2020241127204126.png)
> 
> Chrome will open the 'Overrides' tab and show that 'game2.js' is now being overridden, by tagging the filename with a purple dot
> 
> ![Pasted image 20241127204825](attachments/Pasted%20image%2020241127204825.png)
> 
> Now you can edit the local file 'game2.js', which will override the javascript code of the game's original files.

> [!WARNING]
> Do not edit 'game2.js' in the browser - instead edit 'game2.js' in the override directory

Open a 'Windows Terminal' (PowerShell prompt) on your Windows computer, and paste the following contents.

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

In the same Terminal / PowerShell, paste the following contents to modify 'game2.js':

```powershell
# Change to your Override directory
cd $HOME\Desktop\HHC2024\hhc24-elfminder.holidayhackchallenge.com\

# Create the diff file
@"
--- .\game2.js
+++ .\game2.js
@@ -942,1 +942,1 @@
-            if (!relevantEntities.length && !springsAtPoint.length) {
+            if (1) {

@@ -944,1 +944,1 @@
-                if (existingSprings.length === 2) {
+                if (existingSprings.length === 999) {

@@ -954,1 +954,1 @@
-            if (!existingEntities.length) {
+            if (1) {

@@ -957,1 +957,1 @@
-                if (inTwoSegments) return;
+                                

@@ -964,1 +964,1 @@
-                game.entities.push([...cellMidpoint, EntityTypes.PORTAL]);
+                game.entities.push([...cursor, EntityTypes.PORTAL]);
"@ | Out-File -FilePath "game2.js.diff" -Encoding UTF8

# Apply the diff. If this step fails, ensure that you have loaded the windiff function shared earlier
windiff -apply game2.js .\game2.js.diff
```

> Afted applying the diff, the browser should ask whether to apply the changes (Click 'OK' when it does), but if it does not, simply refresh the game website to apply the changes.
> 
> With the changes made, you can now place unlimited Springs, and you can place Tunnels within 1 distance of objects - which will be crucial for the Gold challenge.
> 
> Below are screenshots of solutions for each level:

> [!NOTE]
> If you have issues placing a tunnel or spring at a specific location, open the developer console, and try placing it with the developer console open. If that does not work, close the developer console and try placing it with the console closed. If that doesn't work, resize the game window until it works.

> Sandy Start
> 
> ![Pasted image 20241127225008](attachments/Pasted%20image%2020241127225008.png)
> 
> Waves and Crates
> 
> ![Pasted image 20241127225202](attachments/Pasted%20image%2020241127225202.png)
> 
> Tidal Treasures
> 
> ![Pasted image 20241127225249](attachments/Pasted%20image%2020241127225249.png)
> 
> Dune Dash
> 
> ![Pasted image 20241127230156](attachments/Pasted%20image%2020241127230156.png)
> 
> Coral Cove
> 
> ![Pasted image 20241127230424](attachments/Pasted%20image%2020241127230424.png)
> 
> Shell Seekers
> 
> ![Pasted image 20241127230837](attachments/Pasted%20image%2020241127230837.png)
> 
> Palm Grove Shuffle
> 
> ![Pasted image 20241127231017](attachments/Pasted%20image%2020241127231017.png)
> 
> Tropical Tangle
> 
> ![Pasted image 20241127231132](attachments/Pasted%20image%2020241127231132.png)
> 
> Crate Caper
> 
> ![Pasted image 20241127231232](attachments/Pasted%20image%2020241127231232.png)
> 
> Shoreline Shuffle
> 
> ![Pasted image 20241127231318](attachments/Pasted%20image%2020241127231318.png)
> 
> Beach Bounty
> 
> ![Pasted image 20241127231430](attachments/Pasted%20image%2020241127231430.png)
> 
> Driftwood Dunes
> 
> ![Pasted image 20241127231518](attachments/Pasted%20image%2020241127231518.png)
> 
> Completing the above levels will award the Silver medal
> 
> ### Gold Medal
> 
> A Real Pickle
> 
> ![Pasted image 20241127231702](attachments/Pasted%20image%2020241127231702.png)
> 
> Completing the level 'A Real Pickle' awards the Gold medal
> 
> Medal progress:
> 
> ![Pasted image 20241127231927](attachments/Pasted%20image%2020241127231927.png)
