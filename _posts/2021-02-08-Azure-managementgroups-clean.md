---
title: Azure management-groups clean  
author: Gudszent Otto
date: 2021-02-07 14:10:00 +0800
categories: [Azure,CAF]
tags: [Azure, CAF, Powershell]
---

## simple script to remove Managament groups, before run check, no subscription under groups only in root tenant

```powershell
For ($n=0; $n -le 3; $n++) {$groups =Get-AzManagementGroup; Foreach ($i in $groups.Name){remove-AzManagementGroup -GroupId $i; $i}}
```