---
title: MS Teams
author: Gudszent Otto
date: 2021-06-17 22:10:00 +0800
categories: [Teams]
tags: [Powershell]
---

Allow link share for MS Teams's SP site

```json
Install-Module -Name Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser

Connect-SPOService -Url "https://valami-admin.sharepoint.com/"

Set-SPOSite -Identity "https://valami.sharepoint.com/sites/TesztCsoportOttnak" -SharingCapability ExternalUserAndGuestSharing
```