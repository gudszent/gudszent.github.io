---
title: Azure Policy
author: Gudszent Otto
date: 2021-06-17 14:10:00 +0800
categories: [Azure, Policy]
tags: [Json]
---

How to Create RBAC with json for Management Group LvL

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "ManagamentGroup": {
            "type": "string"
        },
        "enableVmBackup": {
            "type": "string",
            "defaultValue": "Yes",
            "allowedValues": [
                "Yes",
                "No"
            ]
        },
        "denyIpForwarding": {
            "type": "string",
            "defaultValue": "Yes",
            "allowedValues": [
                "Yes",
                "No"
            ]
        },
        "denySubnetWithoutNsg": {
            "type": "string",
            "allowedValues": [
                "Yes",
                "No"
            ],
            "defaultValue": "Yes"
        },
        "GDPRAllowedEurope": {
            "type": "string",
            "defaultValue": "Yes"
        },
        "isAllowedSize": {
            "type": "string",
            "allowedValues": [
                "Yes",
                "No"
            ],
            "defaultValue": "Yes"
        },
        "AllowedSize": {
            "type": "array",
            "defaultValue": [ "Standard_DS1_v2" ]
        },
        "enableTaglz": {
            "type": "string",
            "allowedValues": [
                "Yes",
                "No"
            ],
            "defaultValue": "Yes"
        }

    },
    "variables": {
        "scope": "[concat('/providers/Microsoft.Management/managementGroups/', parameters('ManagamentGroup'))]",
        "policyDefinitions": {
            "deployVmBackup": "/providers/Microsoft.Authorization/policyDefinitions/013e242c-8828-4970-87b3-ab247555486d",
            "denyIpForwarding": "/providers/Microsoft.Authorization/policyDefinitions/88c0b9da-ce96-4b03-9635-f29a937e2900",
            "GDPRAllowedEurope": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
            "AllowedSize": "/providers/Microsoft.Authorization/policyDefinitions/cccc23c7-8427-4f53-ad12-b6a63eb452b3",
            "tagfromsubscription": "/providers/Microsoft.Authorization/policyDefinitions/b27a0cbd-a167-4dfa-ae64-4337be671140"
        },
        "policyAssignmentNames": {
            "deployVmBackup": "Deploy-VM-Backup",
            "denySubnetWithoutNsg": "Deny-Subnet-Without-Nsg",
            "denyIpForwarding": "Deny-IP-forwarding",
            "GDPRAllowedEurope": "Eu-location-allowed",
            "AllowedSize": "Allowed-VM-Size",
            "tagfromsubscription": "Tag-From-subscription"
        },
        "rbacOwner": "8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
        "roleAssignmentNames": {
            "deployVmBackup": "[guid(concat(parameters('ManagamentGroup'),variables('policyAssignmentNames').deployVmBackup))]"
        },
        "listOfAllowedLocations_array": {
            "listOfAllowedLocations": {
                "value": [ "NorthEurope", "WestEurope", "Europe" ]
            }
        },
        "listOfAllowedSKUs_array": {
            "listOfAllowedSKUs": {
                "value": "[parameters('AllowedSize')]"
            }
        },
        "tagname_array": {
            "tagName": {
                "value": "Subscription"
            }
        }
    },

    "resources": [
        {
            "condition": "[equals(parameters('enableVmBackup'), 'Yes')]",
            "type": "Microsoft.Authorization/policyAssignments",
            "apiVersion": "2018-05-01",
            "name": "[variables('policyAssignmentNames').deployVmBackup]",
            "location": "[deployment().location]",
            "scope": "[variables('scope')]",
            "identity": {
                "type": "SystemAssigned"
            },
            "properties": {
                "description": "Deploy-VM-Backup",
                "displayName": "Deploy-VM-Backup",
                "policyDefinitionId": "[variables('policyDefinitions').deployVmBackup]",
                "parameters": {}
            }
        },
        {
            "condition": "[equals(parameters('enableTaglz'), 'Yes')]",
            "type": "Microsoft.Authorization/policyAssignments",
            "apiVersion": "2018-05-01",
            "name": "[variables('policyAssignmentNames').tagfromsubscription]",
            "location": "[deployment().location]",
            "scope": "[variables('scope')]",
            "identity": {
                "type": "SystemAssigned"
            },
            "properties": {
                "description": "[variables('policyAssignmentNames').tagfromsubscription]",
                "displayName": "[variables('policyAssignmentNames').tagfromsubscription]",
                "policyDefinitionId": "[variables('policyDefinitions').tagfromsubscription]",
                "parameters": "[variables('tagname_array')]"
            }
        },
        {
            "condition": "[equals(parameters('enableVmBackup'), 'Yes')]",
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2020-04-01-preview",
            "name": "[variables('roleAssignmentNames').deployVmBackup]",
            "scope": "[variables('scope')]",
            "dependsOn": [
                "[variables('policyAssignmentNames').deployVmBackup]"
            ],
            "properties": {
                "principalType": "ServicePrincipal",
                "roleDefinitionId": "[concat('/providers/Microsoft.Authorization/roleDefinitions/', variables('rbacOwner'))]",
                "principalId": "[if(equals(parameters('enableVmBackup'), 'Yes'), toLower(reference(concat('/providers/Microsoft.Authorization/policyAssignments/', variables('policyAssignmentNames').deployVmBackup), '2018-05-01', 'Full' ).identity.principalId), 'na')]"
            }
        },
        {
            "condition": "[equals(parameters('denyIpForwarding'), 'Yes')]",
            "type": "Microsoft.Authorization/policyAssignments",
            "apiVersion": "2018-05-01",
            "name": "[variables('policyAssignmentNames').denyIpForwarding]",
            "location": "[deployment().location]",
            "scope": "[variables('scope')]",
            "properties": {
                "description": "Deny-IP-Forwarding",
                "displayName": "Deny-IP-Forwarding",
                "policyDefinitionId": "[variables('policyDefinitions').denyIpForwarding]"
            }
        },
        {
            "condition": "[equals(parameters('GDPRAllowedEurope'), 'Yes')]",
            "type": "Microsoft.Authorization/policyAssignments",
            "apiVersion": "2018-05-01",
            "name": "[variables('policyAssignmentNames').GDPRAllowedEurope]",
            "location": "[deployment().location]",
            "scope": "[variables('scope')]",
            "properties": {
                "description": "[variables('policyAssignmentNames').GDPRAllowedEurope]",
                "displayName": "[variables('policyAssignmentNames').GDPRAllowedEurope]",
                "policyDefinitionId": "[variables('policyDefinitions').GDPRAllowedEurope]",
                "parameters": "[variables('listOfAllowedLocations_array')]"
            }
        },
        {
            "condition": "[equals(parameters('isAllowedSize'), 'Yes')]",
            "type": "Microsoft.Authorization/policyAssignments",
            "apiVersion": "2018-05-01",
            "name": "[variables('policyAssignmentNames').AllowedSize]",
            "location": "[deployment().location]",
            "scope": "[variables('scope')]",
            "properties": {
                "policyDefinitionId": "[variables('policyDefinitions').AllowedSize]",
                "parameters": "[variables('listOfAllowedSKUs_array')]"
            }
        }
    ],
    "outputs": {}
}

```
Resource naming  with Policy

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "not": {
            "anyOf": [
              {
                "field": "name",
                "match": "VM-???-prod-NE-##"
              },
              {
                "field": "name",
                "match": "VM-??-prod-NE-##"
              }
            ]
          }
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```
Usefull links:
<https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming>