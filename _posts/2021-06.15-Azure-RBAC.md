---
title: Azure RBAC
author: Gudszent Otto
date: 2021-06-14 14:10:00 +0800
categories: [Azure, RBAC]
tags: [Json]
---

How to Create RBAC with json for Management Group LvL

```json
{
	"$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
	"contentVersion": "1.0.0.0",
	"parameters": {
		"roleName": {
			"type": "string",
			"defaultValue": "Custom Role - DevOps",
			"metadata": {
				"description": "Friendly name of the role definition"
			}
		},
		"roleDescription": {
			"type": "string",
			"defaultValue": "For DevOps Users",
			"metadata": {
				"description": "Detailed description of the role definition"
			}
		},
		"managementid": {
			"type": "string",
			"defaultValue": "test2",
			"metadata": {
				"description": "ManagementID"
			}
		},
		"RDGuid": {
			"type": "string",
			"defaultValue": "[newGuid()]",
			"metadata": {
				"description": "Guid For RBAC"
			}
		}
	},
	"variables": {
        "management": "[concat('/providers/Microsoft.Management/managementGroups/', parameters('managementid'))]"
    },
	"resources": [
		{
			"type": "Microsoft.Authorization/roleDefinitions",
			"apiVersion": "2018-01-01-preview",
			"name": "[parameters('RDGuid')]",
			"properties": {
				"roleName": "[parameters('roleName')]",
				"description": "[parameters('roleDescription')]",
				"type": "customRole",
				"isCustom": true,
				"permissions": [
					{
						"actions": [
							"*"
						],
						"notActions": [
							"Microsoft.Authorization/*/write",
							"Microsoft.Network/publicIPAddresses/write",
							"Microsoft.Network/virtualNetworks/write",
							"Microsoft.KeyVault/locations/deletedVaults/purge/action"
						],
						"dataActions": [],
						"notDataActions": []
					}
				],
				"assignableScopes": [
					"[variables('management')]"
				]
			}
		}
	]
}

```
Useful links:  
https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles  
https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles  
https://docs.microsoft.com/en-us/azure/role-based-access-control/role-definitions