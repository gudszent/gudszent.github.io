---
title: Teams Room User Create
author: Gudszent Otto
date: 2020-09-08 14:10:00 +0800
categories: [O365, Teams]
tags: [O365, Teams, Azure]
---
To create a Teams Room user we have two possible way:

# Microsoft script

Source
<https://docs.microsoft.com/en-us/microsoftteams/rooms/rooms-configure-accounts> 
   
At the buttom of the page  
  
SkypeRoomProvisioningScript.ps1 <https://go.microsoft.com/fwlink/?linkid=870105> 

>If you use custom domain name instead of domain.onmicrosoft.com, you have to edit it at 1113 line
```powershell
$sessSkype = New-CsOnlineSession -Credential $credSkype -OverrideAdminDomain "name.onmicrosoft.com"
```

# Manual powershell command

![ExchangeOnline-powershell](/assets/img/sample/2020-08-03-Teams/exchange-online-powershell.png)

```powershell
### Variables
$newRoom="targyalo@mydomain.com"
$name="Targyalo szoba"
$pwd="Password1"
$license="mydomainname:MEETING_ROOM"
$location="HU"
$orgName="mydomain.onmicrosoft.com"
$mailTip="This room is"

####prerequisite
Import-Module ExchangeOnlineManagement
Install-Module -Name MSOnline
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
Install-Module -Name AzureAD

### Download Skype Online PowerShell manual, than install it and restart the powershell console
https://www.microsoft.com/en-us/download/details.aspx?id=39366

### Install Exchange Online Module 
### Connecting to Microsoft Online Services 
Set-ExecutionPolicy RemoteSigned
$credential = Get-Credential
Connect-MsolService -Credential $credential
Import-Module SkypeOnlineConnector
$sfboSession = New-CsOnlineSession -Credential $credential -OverrideAdminDomain $orgName
Import-PSSession $sfboSession
$credential = Get-Credential
$exchangeSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://outlook.office365.com/powershell-liveid/" -Credential $credential -Authentication "Basic" -AllowRedirection
Import-PSSession $exchangeSession -DisableNameChecking

### View your licenses avaialble
Get-MsolAccountSku

### Creating a new Account
New-Mailbox -MicrosoftOnlineServicesID $newRoom -Name $name -Room -RoomMailboxPassword (ConvertTo-SecureString -String $pwd -AsPlainText -Force) -EnableRoomMailboxAccount $true

### Wait one minute before configuring the new account
Set-MsolUser -UserPrincipalName $newRoom -PasswordNeverExpires $true -UsageLocation $location

### Assigning a license to the room account
Set-MsolUserLicense -UserPrincipalName $newRoom -AddLicenses $license

### Setting a MailTip for the Room
Set-Mailbox -Identity $newRoom -MailTip $mailTip

### Configs the account to process requests
Set-CalendarProcessing -Identity $newRoom -AutomateProcessing AutoAccept -AddOrganizerToSubject $false -RemovePrivateProperty $false -DeleteComments $false -DeleteSubject $false -AddAdditionalResponse $true -AdditionalResponse "Your meeting is now scheduled and if it was enabled as a Teams or Skype Meeting will provide a seamless click-to-join experience from the conference room." 

### Enabling the account for SfB Online - Find the pool first then use the name in -RegistratPool
Get-CsOnlineUser |ft RegistrarPool

### Wait a few minutes before running this next command
Enable-CsMeetingRoom -Identity $newRoom -SipAddressType "EmailAddress" -RegistrarPool "sippoolCWLGB101.infra.lync.com"

### Enable the account for Enterprise Voice
Set-CsMeetingRoom -Identity $newRoom -EnterpriseVoiceEnabled $true

#### Option Configuration 
### Getting Room Mailboxes ###
Get-Mailbox -RecipientTypeDetails RoomMailbox

### Finding and setting allowed external meeting invites from outside the domain
Get-Mailbox $name | Get-CalendarProcessing | Select *external*
Get-Mailbox $name | Set-CalendarProcessing -ProcessExternalMeetingMessages $true

### Checking the Meeting Room Configuration
Get-CsMeetingRoom -Identity "$newroom"
Get-Mailbox -Identity "$newroom" | fl
```