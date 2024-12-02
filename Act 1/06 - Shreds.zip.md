> After unlocking the Silver Frosty Keypad puzzle, you were given a new item 'One Thousand Little Teeny Tiny Shredded Pieces of Paper' by speaking with Morcel Nougat:
> 
> ![Pasted image 20241128005650](attachments/Pasted%20image%2020241128005650.png)
> 
> The 'shredded pieces of paper' links to https://holidayhackchallenge.com/2024/shreds.zip
> 
> The hint 'On the Cutting Edge' mentions a python script 'heuristic_edge_detection.py', but we ignore that because we will solve this challenge using PowerShell.
 
> [!NOTE]
>  A lot of players who used the Python heuristics script ended up with a mirrored image output. The original image is actually not mirrored, but due to the way that the Python script reassembles the slices, there is a chance that it reconstructs a mirror of the original image. The script matches the second slice on the wrong side of the first slice, and the script will continue matching the rest of the slices, resulting in a reversed image.

> The Python image output:
> ![Pasted image 20241128010844](attachments/Pasted%20image%2020241128010844.png)
> 
> Surely, we can do better than that with PowerShell!
> 
> ![](attachments/Pasted%20image%2020241129004026.png) 
> 
> The following PowerShell script prints the metadata contained in the image files:
> 
> ~~~powershell
> function Get-ImageMetadata {
>     param (
>         [Parameter(Mandatory=$true)]
>         [string]$FilePath
>     )
> 
>     try {
>         # Get full path and verify file exists
>         $FullPath = Resolve-Path $FilePath -ErrorAction Stop
>         if (-not (Test-Path $FullPath)) {
>             throw "File not found: $FullPath"
>         }
> 
>         # Create Shell.Application COM object for basic file properties
>         $shell = New-Object -ComObject Shell.Application
>         $folder = $shell.Namespace((Get-Item $FullPath).DirectoryName)
>         $file = $folder.ParseName((Get-Item $FullPath).Name)
> 
>         Write-Host "`nFile System Properties:" -ForegroundColor Cyan
>         Write-Host "=====================" -ForegroundColor Cyan
> 
>         # Get all possible properties (0-288 is a safe range)
>         0..288 | ForEach-Object {
>             $propertyName = $folder.GetDetailsOf($folder.Items, $_)
>             $propertyValue = $folder.GetDetailsOf($file, $_)
>             
>             if ($propertyValue) {
>                 [PSCustomObject]@{
>                     Property = $propertyName
>                     Value = $propertyValue
>                 }
>             }
>         } | Format-Table -AutoSize
> 
>         # Add System.Drawing assembly for image-specific properties
>         Add-Type -AssemblyName System.Drawing
> 
>         Write-Host "`nImage-Specific Properties:" -ForegroundColor Cyan
>         Write-Host "======================" -ForegroundColor Cyan
> 
>         # Open image file with FileStream to ensure proper access
>         $fileStream = [System.IO.File]::OpenRead($FullPath)
>         try {
>             $image = [System.Drawing.Image]::FromStream($fileStream, $false, $false)
>             try {
>                 # Basic image properties
>                 $basicProps = [PSCustomObject]@{
>                     'Width' = $image.Width
>                     'Height' = $image.Height
>                     'Horizontal Resolution' = "$($image.HorizontalResolution) DPI"
>                     'Vertical Resolution' = "$($image.VerticalResolution) DPI"
>                     'Pixel Format' = $image.PixelFormat
>                     'Raw Format' = $image.RawFormat
>                     'Physical Dimension' = $image.PhysicalDimension
>                 }
>                 $basicProps | Format-List
> 
>                 # Get all image properties
>                 Write-Host "`nEXIF Properties:" -ForegroundColor Cyan
>                 Write-Host "===============" -ForegroundColor Cyan
> 
>                 $propItems = $image.PropertyItems
>                 foreach ($prop in $propItems) {
>                     $value = $null
>                     
>                     # Convert byte array to appropriate format based on type
>                     switch ($prop.Type) {
>                         1 { # Byte
>                             $value = $prop.Value[0]
>                         }
>                         2 { # ASCII
>                             $value = [System.Text.Encoding]::ASCII.GetString($prop.Value).Trim([char]0)
>                         }
>                         3 { # Short (16-bit)
>                             $value = [BitConverter]::ToUInt16($prop.Value, 0)
>                         }
>                         4 { # Long (32-bit)
>                             $value = [BitConverter]::ToUInt32($prop.Value, 0)
>                         }
>                         5 { # Rational
>                             if ($prop.Value.Length -ge 8) {
>                                 $numerator = [BitConverter]::ToUInt32($prop.Value, 0)
>                                 $denominator = [BitConverter]::ToUInt32($prop.Value, 4)
>                                 if ($denominator -ne 0) {
>                                     $value = "$numerator/$denominator"
>                                 }
>                             }
>                         }
>                         7 { # Undefined
>                             $value = [System.BitConverter]::ToString($prop.Value)
>                         }
>                         9 { # SLong (32-bit signed)
>                             $value = [BitConverter]::ToInt32($prop.Value, 0)
>                         }
>                         10 { # SRational
>                             if ($prop.Value.Length -ge 8) {
>                                 $numerator = [BitConverter]::ToInt32($prop.Value, 0)
>                                 $denominator = [BitConverter]::ToInt32($prop.Value, 4)
>                                 if ($denominator -ne 0) {
>                                     $value = "$numerator/$denominator"
>                                 }
>                             }
>                         }
>                         default {
>                             $value = [System.BitConverter]::ToString($prop.Value)
>                         }
>                     }
> 
>                     # Common EXIF IDs
>                     $exifTags = @{
>                         '0x010F' = 'Camera Manufacturer'
>                         '0x0110' = 'Camera Model'
>                         '0x0112' = 'Orientation'
>                         '0x829A' = 'Exposure Time'
>                         '0x829D' = 'F-Number'
>                         '0x8827' = 'ISO Speed'
>                         '0x9003' = 'Date Taken'
>                         '0x9004' = 'Date Digitized'
>                         '0x920A' = 'Focal Length'
>                         '0xA402' = 'Exposure Mode'
>                         '0xA403' = 'White Balance'
>                         '0xA406' = 'Scene Type'
>                     }
> 
>                     $hexId = "0x{0:X4}" -f $prop.Id
>                     $tagName = $exifTags[$hexId]
>                     if (-not $tagName) {
>                         $tagName = "Tag $hexId"
>                     }
> 
>                     [PSCustomObject]@{
>                         'Property' = $tagName
>                         'Value' = $value
>                         'Type' = $prop.Type
>                     } | Format-Table -AutoSize
>                 }
>             }
>             finally {
>                 if ($null -ne $image) {
>                     $image.Dispose()
>                 }
>             }
>         }
>         finally {
>             if ($null -ne $fileStream) {
>                 $fileStream.Dispose()
>             }
>         }
>     }
>     catch {
>         Write-Host "Error processing image: $_" -ForegroundColor Red
>         Write-Host "Stack Trace: $($_.Exception.StackTrace)" -ForegroundColor Red
>     }
>     finally {
>         if ($null -ne $shell) {
>             [System.Runtime.Interopservices.Marshal]::ReleaseComObject($shell) | Out-Null
>         }
>         [System.GC]::Collect()
>         [System.GC]::WaitForPendingFinalizers()
>     }
> }
> 
> # Example usage:
> Get-ImageMetadata -FilePath "$HOME\Desktop\HHC2024\shreds\slices\fbae876c-cc76-42b8-8563-df7c0a96b4f1.jpg"
> ~~~
> We see that each of the images contains a Base64 encoded string in the User Comment metadata field.
> 
> ~~~
> Comments              WyI9VXpNIiwgImEgbSJd
> ~~~
> 
> We can decode the Base64 encoded string using PowerShell as follows:
> ~~~powershell
$base64 = "WyI9VXpNIiwgImEgbSJd"
$decoded = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($base64))
Write-Host "Decoded string: $decoded"
> ~~~
> The Base64 comment in each of the slice image files decodes to a JSON structure:
> 
> ~~~
> Decoded string: ["=UzM", "a m"]
> ~~~
> 
> The first data object is a reversed Base64 string - the character set of these metadata elements is typical of Base64 and easily recognized as Base64, even when reversed.
> The second data object is a text string.
> 
> Decoding the reversed base64 returns the string '35':
> ~~~powershell
$reversed_base64 = "=UzM"
$unreversed_base64 = -join $reversed_base64[($reversed_base64.Length-1)..0]
Write-Host $([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($unreversed_base64)))
> ~~~
> ~~~
> 35
> ~~~
> 
> The strings are all numbers in the range 1 to 1000. There are 1000 slices, so the numbers very likely indicate the slice order of the original image.
> 
> We write a script in PowerShell that decodes the base64 comment data, and combines the 1000 slices in the correct order based on the decoded comment. Additionally, the script prints out the combined text objects:

```powershell
# create a working directory
$dir = "$HOME\Desktop\HHC2024\"; mkdir $dir -ea 0; cd $dir

# download shreds.zip
wget https://holidayhackchallenge.com/2024/shreds.zip -Outfile shreds.zip

# unzip shreds.zip
Expand-Archive -ea 0 shreds.zip

# magic time
Add-Type -AssemblyName System.Drawing

function Get-ExifUserComment {
    param ([string]$imagePath)
    try {
        $image = [System.Drawing.Image]::FromFile($imagePath)
        foreach ($prop in $image.PropertyItems) {
            if ($prop.Id -eq 37510) {
                return [System.Text.Encoding]::Unicode.GetString($prop.Value).Trim([char]0)
            }
        }
    } finally {
        if ($image) { $image.Dispose() }
    }
}

function Decode-MetadataComment {
    param ([string]$base64Comment)
    try {
        $decoded = [Text.Encoding]::Utf8.GetString([Convert]::FromBase64String($base64Comment))
        if ($decoded -match '^\["([^"]+)",\s*"([^"]+)"\]$') {
            $orderPartReversed = -join $matches[1][$matches[1].Length..0]
            $orderNumber = [Text.Encoding]::Utf8.GetString([Convert]::FromBase64String($orderPartReversed))
            return @{
                Order = [int]$orderNumber
                Text = $matches[2]
            }
        }
    } catch {
        Write-Host "Error decoding metadata: $_"
    }
}

function Convert-UnicodeEscapes {
    param ([string]$text)
    
    if ($text -match '\\u[0-9a-fA-F]{4}') {
        $decoded = [Regex]::Replace(
            $text,
            '\\u([0-9a-fA-F]{4})',
            { param($m) [char][int]('0x' + $m.Groups[1].Value) }
        )
        return $decoded
    }
    return $text
}

$imageData = @()
Get-ChildItem -Path ".\shreds\slices\" -Filter "*.jpg" | ForEach-Object {
    $comment = Get-ExifUserComment -imagePath $_.FullName
    if ($comment) {
        $metadata = Decode-MetadataComment -base64Comment $comment
        if ($metadata) {
            $imageData += @{
                Path = $_.FullName
                Order = $metadata.Order
                Text = $metadata.Text
            }
        }
    }
}

if ($imageData.Count -gt 0) {
    $sortedImages = $imageData | Sort-Object { [int]$_.Order }
    $images = @()
    $finalImage = $null
    $graphics = $null
    
    try {
        $totalWidth = 0
        $height = 0
        
        foreach ($imgData in $sortedImages) {
            $img = [System.Drawing.Image]::FromFile($imgData.Path)
            $images += $img
            $totalWidth += $img.Width
            $height = if ($height -eq 0) { $img.Height } else { $height }
        }
        
        $finalImage = New-Object System.Drawing.Bitmap([int]$totalWidth, [int]$height)
        $graphics = [System.Drawing.Graphics]::FromImage($finalImage)
        $graphics.InterpolationMode = [System.Drawing.Drawing2D.InterpolationMode]::HighQualityBicubic
        
        $currentX = 0
        foreach ($img in $images) {
            $graphics.DrawImage($img, $currentX, 0, $img.Width, $img.Height)
            $currentX += $img.Width
        }
        
        $outputPath = Join-Path $PWD.Path "combined.jpg"
        $jpegCodec = [System.Drawing.Imaging.ImageCodecInfo]::GetImageEncoders() | 
            Where-Object { $_.MimeType -eq 'image/jpeg' }
        $encoderParams = New-Object System.Drawing.Imaging.EncoderParameters(1)
        $encoderParams.Param[0] = New-Object System.Drawing.Imaging.EncoderParameter(
            [System.Drawing.Imaging.Encoder]::Quality, 100L)
        
        $finalImage.Save($outputPath, $jpegCodec, $encoderParams)
        Write-Host "Combined image saved to: $outputPath"
        
        $finalText = ($sortedImages | ForEach-Object { $_.Text }) -join ''
        $finalText = Convert-UnicodeEscapes $finalText
        Write-Host "`nDecoded text in order:"
        Write-Host $finalText
        
    } finally {
        if ($graphics) { $graphics.Dispose() }
        if ($finalImage) { $finalImage.Dispose() }
        foreach ($img in $images) { if ($img) { $img.Dispose() } }
    }
}
```

> The script outputs the combined image and a Christmas story:
> 
> ~~~
> Combined image saved to: C:\Users\username\Desktop\HHC2024\combined.jpg
> 
> Decoded text in order:
> Long ago, in the snowy realm of the North Pole (not too far away if you're a reindeer), there existed a magical land ruled by a mysterious figure known as the Great Claus. Two spirited elves, Twinkle and Jangle, roamed this frosty kingdom, defending it from the perils of holiday cheerlessness. Twinkle, sporting a bright red helmet-shaped hat that tilted just so, was quick-witted and even quicker with a snowball. Jangle, a bit taller, wore a green scarf that drooped like a sleepy reindeer’s ears. Together, they were the Mistletoe Knights, the protectors of the magical land and the keepers of Claus’ peace. One festive morning, the Great Claus summoned them for a critical quest. 'Twinkle, Jangle, the time has come,' he announced with a voice that rumbled like thunder across the ice plains. 'The fabled Never-Melting Snowflake, a relic that grants one wish, lies hidden beyond the Peppermint Expanse. Retrieve it, and all marshmallow supplies will be secured!' Armed with Jangle’s handmade map (created with crayon and a lot of optimism), the duo set off aboard their toboggan, the Frostwing. However, the map led them in endless loops around the Reindeer Academy, much to the amusement of trainee reindeer perfecting their aerial maneuvers. Blitzen eventually intercepted them, chuckling, 'Lost, fellas? The snowflake isn’t here. Try the Enchanted Peppermint Grove!' Twinkle facepalmed as Jangle pretended to adjust his map. With Blitzen’s directions, they zoomed off again, this time on the right course. The Peppermint Grove was alive with its usual enchantments—candy cane trees swayed and sang ancient ballads of epic sleigh battles and the triumphs of Claus’ candy cane squadrons. Twinkle and Jangle joined the peppermint choir, their voices harmonizing with the festive tune. Hours later, the duo stumbled upon a hidden cave guarded by giant gumdrop sentinels (luckily on their lunch break). Inside, the air shimmered with Claus’ magic. There it was—the Never-Melting Snowflake, glistening on a pedestal of ice. Twinkle’s eyes widened, 'We’ve found it, Jangle! The key to infinite marshmallows!' As Twinkle reached for the snowflake, a voice boomed from the cave walls, 'One wish, you have. Choose wisely or face the egg-nog of regret.' Without hesitation, Jangle exclaimed, 'An endless supply of marshmallows for our cocoa!' The snowflake glowed, and with a burst of magic, marshmallows poured down, covering the cave in a fluffy, sweet avalanche. Back at the workshop, the elves were hailed as heroes—the Marshmallow Knights of Claus. They spent the rest of the season crafting new cocoa recipes and sharing their bounty with all. And so, under the twinkling stars of the northern skies, Twinkle and Jangle continued their adventures, their mugs full of cocoa, their hearts full of joy, and their days full of magic. For in the North Pole, every quest was a chance for festive fun, and every snowflake was a promise of more marshmallows to come.
> ~~~
> 
> The combined image looks great!
> 
> ![Pasted image 20241128011522](attachments/Pasted%20image%2020241128011522.png)
>
> ~~~
> BAUD         : 115200
> PARITY       : EVEN
> DATA         : 7 BITS
> STOPPITS     : 1 BIT
> FLOW CONTROL : RTS
> ~~~

> [!NOTE]
> Solving this challenge does not reward a medal or achievement, but the information is a prerequisite to the 'Hardware Hacking 101 - Part 1' challenge.
