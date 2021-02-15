---
title: Lost trust relationship between domain and workstation  
author: Gudszent Otto
date: 2021-02-07 14:10:00 +0800
categories: [Windows]
tags: [Domain]
---


```bash
Test-ComputerSecureChannel -Repair
$cred = Get-Credential
Test-ComputerSecureChannel -Credential $cred -Repair
```