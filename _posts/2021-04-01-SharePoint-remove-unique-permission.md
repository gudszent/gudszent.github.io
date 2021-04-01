---
title: SharePoint remove unique permission in folder 
author: Gudszent Otto
date: 2021-04-01 14:10:00 +0800
categories: [PowerShell, SharePoint]
tags: [PowerShell, SharePoint]
---

```powershell
#Set Variables
$SiteURL = "" #Document Library URL
$FolderURL = "" #Document Library Site Relative URL
 
#Connect to PnP Online
Connect-PnPOnline -Url $SiteURL -UseWebLogin 
 
#Function to reset permissions of all Sub-Folders
Function Reset-SubFolderPermissions($FolderURL)
{
    #Get all sub-folders of the Folder - Exclude system folders
    $SubFolders = Get-PnPFolderItem -FolderSiteRelativeUrl $FolderURL -ItemType Folder -Recursive | Where {$_.Name -ne "Forms" -and $_.Name -ne "Document"} 
 
    #Loop through each sub-folder
    ForEach($SubFolder in $SubFolders)
    {
        $SubFolderURL = $FolderUrl+"/"+$SubFolder.Name
        Write-host -ForegroundColor Green "Processing Folder '$($SubFolder.Name)' at $SubFolderURL"
 
        #Get the Folder Object - with HasUniqueAssignments and ParentList properties
        $Folder = Get-PnPFolder -Url $SubFolderURL -Includes ListItemAllFields.HasUniqueRoleAssignments, ListItemAllFields.ParentList, ListItemAllFields.ID
 
        #Get the List Item of the Folder
        $FolderItem = $Folder.ListItemAllFields
 
        #Check if the Folder has unique permissions
        If($FolderItem.HasUniqueRoleAssignments)
        {
            #Reset permission inheritance
            Set-PnPListItemPermission -List $FolderItem.ParentList -Identity $FolderItem.ID -InheritPermissions
            Write-host "`tUnique Permissions are removed from the Folder!"
        }
 
        #Call the function recursively
        Reset-SubFolderPermissions $SubFolderURL
    }
}
   
#Call the function
Reset-SubFolderPermissions $FolderURL
```
Original:
https://www.sharepointdiary.com/2019/02/sharepoint-online-remove-unique-permissions-from-all-folders-using-powershell.html