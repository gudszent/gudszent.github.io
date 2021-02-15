---
title: Clear Teams Cache
author: Gudszent Otto
date: 2020-09-07 14:10:00 +0800
categories: [O365, Teams]
tags: [O365, Teams]
---
Just run as Administrator a Powershell script

```powershell
Write-Host "Stopping Teams Process" -ForegroundColor Yellow
try{
Get-Process -ProcessName Teams | Stop-Process -Force
Start-Sleep -Seconds 3
Write-Host "Teams Process Sucessfully Stopped" -ForegroundColor Green
}catch{
echo $_
}
Write-Host "Clearing Teams Disk Cache" -ForegroundColor Yellow
try{
Get-ChildItem "C:\Users\*\AppData\Roaming\Microsoft\Teams\*" -directory | Where name -in ('application cache','blob storage','databases','GPUcache','IndexedDB','Local Storage','tmp') | ForEach{Remove-Item $_.FullName -Recurse -Force -WhatIf}
Write-Host "Teams Disk Cache Cleaned" -ForegroundColor Green
}catch{
echo $_
}
Write-Host "Cleanup Complete... Launching Teams" -ForegroundColor Green
Start-Process -FilePath $env:LOCALAPPDATA\Microsoft\Teams\current\Teams.exe
```



