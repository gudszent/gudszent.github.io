---
title: SCCM Create OS wim image commands
author: Gudszent Otto
date: 2020-08-02 14:10:00 +0800
categories: [SCCM]
tags: [SCCM, PowerShell, Driver]
---

# SCCM driver package create
```powershell
Export-WindowsDriver
```

## Target computer mode

```
wmic computersystem get model
SCCM console
SELECT * FROM Win32_ComputerSystem WHERE Model LIKE '%HP EliteBook 8560w%'
```
# Create wim

## Get wim with only required type of Windows

```bash
Dism /Get-ImageInfo /ImageFile:D:\Temp\install.wim
###Get index of the Windows 10 Enterprise
Dism /Export-Image /SourceImageFile:D:\Temp\install.wim /SourceIndex:IndexNumber /DestinationImageFile:D:\Temp\exported.wim /Compress:max /CheckIntegrity
```

## Remove
```bash
Remove-WindowsImage -ImagePath "D:\Temp\install.wim" -Index 2 -CheckIntegrity
```
## Mount windows
```bash
Dism /Mount-Image /ImageFile:D:\Temp\install.wim /Index:1 /MountDir:"D:\Mount\Windows"
```
## Add language
```bash
Dism /Image:"D:\Mount\Windows" /Add-Package /PackagePath="D:\Temp\Microsoft-Windows-Client-Language-Pack_x64_hu-hu.cab"
```
## Commit, Finalize
```bash
Dism /Unmount-image /MountDir:D:\Mount\Windows /Commit
```
# SCCM Slow PXE Boot 
```bash
regedit
```

Location: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SMS\DP  
Name: RamDiskTFTPWindowSize  
Type: REG_DWORD

> The default value is 1 (1 data block fills the window)
We can also tweak the TFTPBlockSize which has been around for many versions of Configuration Manager.
Location: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SMS\DP
Name: RamDiskTFTPBlockSize
Type: REG_DWORD
Value: <customized block size>
The default value is 4096 (4k).

