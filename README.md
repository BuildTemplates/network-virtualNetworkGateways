# virtualNetworkGateways
Reference deployment
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageAccountName": {
      "type": "string",
      "defaultValue": ""
    },
    "ContainerName": {
      "type": "string",
      "defaultValue": ""
    },
    "SasToken": {
      "type": "string",
      "defaultValue": ""
    }
  },
  "variables": {
    "virtualNetworkGatewayUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),'/Microsoft.Network','/virtualNetworkGateways')]",
  },
  "resources": [
    {
      "name": "BuildVirtualNetworkGatewaySku",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('virtualNetworkGatewayUri'), '/virtualNetworkGatewaySku.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "HighPerformance"
          },
          "tier": {
            "value": "HighPerformance"
          }
        }
      }
    },
    {
      "name": "BuildBgpSettings",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('virtualNetworkGatewayUri'), '/bgpSettings.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
        }
      }
    },
    {
      "name": "BuildVirtualNetworkGatewayIpConfigurations",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('virtualNetworkGatewayUri'), '/ipConfigurations.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "ipconfig1"
          },
          "privateIPAllocationMethod": {
            "value": "Dynamic"
          },
          "publicIPAddressId": {
            "value": "/subscriptions/3c22907e-eb72-46ba-8e71-2ed329c8394d/resourceGroups/Testing/providers/Microsoft.Network/publicIPAddresses/myPip"
          },
          "subnetId": {
            "value": "/subscriptions/3c22907e-eb72-46ba-8e71-2ed329c8394d/resourceGroups/Testing/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/default"
          }
        }
      }
    },
    {
      "name": "BuildVirtualNetworkGateway",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworkGatewayIpConfigurations')]",
        "[concat('Microsoft.Resources/deployments/','BuildBgpSettings')]",
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworkGatewaySku')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('virtualNetworkGatewayUri'), '/virtualNetworkGateways.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "name": {
            "value": "VnetGW"
          },
          "gatewayType": {
            "value": "ExpressRoute"
          },
          "bgpSettings": {
            "value": "[reference('BuildBgpSettings').outputs.bgpSettings.value]"
          },
          "ipConfigurations": {
            "value": "[createArray(reference('BuildVirtualNetworkGatewayIpConfigurations').outputs.ipConfiguration.value)]"
          },
          "sku": {
            "value": "[reference('BuildVirtualNetworkGatewaySku').outputs.virtualNetworkGatewaySku.value]"
          }
        }
      }
    }
  ],
  "outputs": {
    "Gateway": {
      "type": "object",
      "value": "[reference('BuildVirtualNetworkGateway').outputs.virtualNetworkGateway.value]"
    }
  }
}
```
