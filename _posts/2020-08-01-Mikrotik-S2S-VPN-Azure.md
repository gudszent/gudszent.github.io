---
title: How to create Azure S2S VPN with mikrotik, if we have DynamicIP
author: Gudszent Otto
date: 2020-08-01 14:10:00 +0800
categories: [Azure, S2S VPN]
tags: [Azure, Mikrotik, S2S, VPN, PowerShell]
---

Required/tested on:

* Mikrotik `6.47`  
* Windows `Server 2019`  
* Azure 


## Install on Windows Server


```powershell
Set-ExecutionPolicy Unrestricted

Install-Module -Name Az -AllowClobber -Scope AllUsers

Install-Module AzureRM -AllowClobber
```


### Generate Key file for Azure user password

```powershell
$KeyFile = "D:\Backup\PowerShell\AzureAES.key"
$Key = New-Object Byte[] 16   # You can use 16, 24, or 32 for AES
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($Key)
$Key | out-file $KeyFile 
```

### Than password

```powershell
$PasswordFile = "D:\Backup\PowerShell\AzurePassword.txt"
$KeyFile = "D:\Backup\PowerShell\AzureAES.key"
$Key = Get-Content $KeyFile
$Password = "P@ssword1" | ConvertTo-SecureString -AsPlainText -Force
$Password | ConvertFrom-SecureString -key $Key | Out-File $PasswordFile 
```


## The Script, get my ip and update in Azure Gateway

Azure_VPN.ps1

```powershell
#Dynamic DNS Entry for your dynamic IP
$DynDNS = (Invoke-WebRequest -uri "http://ifconfig.me/ip").Content
#Azure subscription name
$SubscriptionName = "Visual Studio Enterprise"

#UPN of a user account with administrative access to the subscription
$User = "VPN@tenant.onmicrosoft.com"
#Password file
$PasswordFile = ".\AzurePassword.txt"
#Key file to decrypt the password
$KeyFile = ".\AzureAES.key"

Write-Host "Building Credentials"
#Grab the contents of the key
$key = Get-Content $KeyFile
$SecurePassword = Get-Content $PasswordFile | ConvertTo-SecureString -Key $key
#Build the credential object
$Creds = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User, $SecurePassword

#Get the Current Dynamic IP
[string]$DynIP = ([System.Net.DNS]::GetHostAddresses($DynDNS)).IPAddressToString
Write-Host "Current Dynamic IP:" $DynIP

#Log into the Azure Tenant
Login-AzureRmAccount -Credential $Creds
#Select the subscription
Select-AzureRmSubscription -SubscriptionName $SubscriptionName
#Grab the current LocalNetworkGateway
$LNG = Get-AzureRmLocalNetworkGateway -ResourceName HomeVPN_LNG -ResourceGroupName Home
#Output the IP to view
Write-Host "Current LocalNetworkGateway IP:" $LNG.GatewayIPAddress
Write-Host "Current Dynamic IP:" $DynIP

#Determine if we need to change it
If($DynIP -ne $LNG.GatewayIpAddress)
    {
    Write-Host "Dynamic IP is different to LocalNetworkGateway IP - Updating..."
    #Update the IP in the LNG Object
    $LNG.GatewayIpAddress =$DynIP
    #Update the LNG Object in Azure. AddressPrefix is required.
    Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $LNG -AddressPrefix @('10.6.1.0/24')
    }
else
    {
    Write-Host "No changes required"
    } 
```
## Create Windows ScheduleTask

program/script  

```  
C:\WINDOWS\system32\WindowsPowerShell\v1.0\powershell.exe  
```  
Add arguments  
```    
-NoLogo -NonInteractive -file "D:\Backup\PowerShell\Azure_VPN.ps1"
```

## Update with Automation Account

Create Automation account
- Create Credentials
- Create a powershell Runbook.

```powershell
#Dynamic DNS Entry for your dynamic IP
$hostName = "domain.hu"
$ip = "192.168.1.0/24"
$DynDNS = [system.net.dns]::GetHostByName($hostName).AddressList.IPAddressToString
#Azure subscription name
$SubscriptionName = "Visual Studio Enterprise"

#UPN of a user account with administrative access to the subscription
$myCredential = Get-AutomationPSCredential -Name 'vpnuser'
$User = $myCredential.UserName
$securePassword = $myCredential.Password
$password = $myCredential.GetNetworkCredential().Password


#Build the credential object
$Creds = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User, $SecurePassword

#Get the Current Dynamic IP
[string]$DynIP = ([System.Net.DNS]::GetHostAddresses($DynDNS)).IPAddressToString
Write-Host "Current Dynamic IP:" $DynIP
Write-Output "Current Dynamic IP:" $DynIP
#Log into the Azure Tenant
Login-AzureRmAccount -Credential $Creds | Out-Null
#Select the subscription
Select-AzureRmSubscription -SubscriptionName $SubscriptionName | Out-Null
#Grab the current LocalNetworkGateway
$LNG = Get-AzureRmLocalNetworkGateway -ResourceName Home-VPN-LNG -ResourceGroupName Home-GW
#Output the IP to view
Write-Host "Current LocalNetworkGateway IP:" $LNG.GatewayIPAddress
Write-Output "Current LocalNetworkGateway IP:" $LNG.GatewayIPAddress
Write-Host "Current Dynamic IP:" $DynIP
Write-Output "Current Dynamic IP:" $DynIP

#Determine if we need to change it
If($DynIP -ne $LNG.GatewayIpAddress)
    {
    Write-Host "Dynamic IP is different to LocalNetworkGateway IP - Updating..."
    Write-Output "Dynamic IP is different to LocalNetworkGateway IP - Updating..."
    #Update the IP in the LNG Object
    $LNG.GatewayIpAddress =$DynIP
    #Update the LNG Object in Azure. AddressPrefix is required.
    Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $LNG -AddressPrefix @($ip)
    }
else
    {
    Write-Host "No changes required"
    Write-Output "No changes required"
    }
```



## Mikrotik Setup

With Winbox, we have to create firewall rule and ipsec policy. That will connect the Azure Virtual Gateway and control the network communication. 

### Firewall

```bash
/ip firewall nat add chain=srcnat action=accept src-address=10.10.10.0/24 dst-address=10.6.1.0/24

# Set TCPMSS to 1350 (varies depends on your local network configuration)

/ip firewall mangle add chain=forward action=change-mss new-mss=1350 passthrough=yes tcp-flags=syn protocol=tcp tcp-mss=!0-1350 log=no log-prefix=""
```
> **Note**: Important, this have to be before the NAT.  


```
/ip firewall mangle add chain=forward action=change-mss new-mss=1350 passthrough=yes tcp-flags=syn protocol=tcp tcp-mss=!0-1350 log=no log-prefix=""
/ip ipsec mode-config
add name=AzureS2S_Mode responder=no
/ip ipsec policy group
add name=AzureS2S_Group
/ip ipsec profile
add dh-group=modp1024 enc-algorithm=aes-256 lifetime=2h name=AzureS2S_profile
/ip ipsec peer
add address=AZUREGATEWAYIP/32 exchange-mode=ike2 name=AzureS2S_Peer profile=AzureS2S_profile
/ip ipsec proposal
set [ find default=yes ] disabled=yes
add enc-algorithms=aes-256-cbc,aes-128-cbc lifetime=7h30m name=AzureS2S_Porposal pfs-group=none
add disabled=yes enc-algorithms=aes-256-cbc lifetime=2h name=azure-ipsec-proposal
/ip ipsec identity
add peer=AzureS2S_Peer secret=password
/ip ipsec policy
set 0 disabled=yes
add dst-address=10.10.0.0/16 peer=AzureS2S_Peer proposal=AzureS2S_Porposal sa-dst-address=Azure_S2S_publicIP sa-src-address=0.0.0.0 src-address=10.6.1.0/24 tunnel=yes

```
> **Note**: Important, AZUREGATEWAYIP change to your Azure Virtual Gateway  