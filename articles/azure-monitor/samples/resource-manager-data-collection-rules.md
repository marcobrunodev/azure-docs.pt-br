---
title: Exemplos de modelo do Resource Manager para regras de coleta de dados
description: Modelos do Azure Resource Manager de exemplo para criar associações entre regras de coleta de dados e máquinas virtuais no Azure Monitor.
ms.subservice: logs
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 11/17/2020
ms.openlocfilehash: 1c059edb1422a572011f167f7f1c02d5e87e5da2
ms.sourcegitcommit: 5ae2f32951474ae9e46c0d46f104eda95f7c5a06
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95324815"
---
# <a name="resource-manager-template-samples-for-data-collection-rules-in-azure-monitor"></a>Exemplos de modelo do Resource Manager para regras de coleta de dados no Azure Monitor
Este artigo inclui [modelos do Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) de amostra para implantar e configurar o [agente do Log Analytics](../platform/log-analytics-agent.md) e a [extensão de diagnóstico](../platform/diagnostics-extension-overview.md) para máquinas virtuais no Azure Monitor. Cada amostra inclui um arquivo de modelo e um arquivo de parâmetros com valores de amostra para fornecer ao modelo.

[!INCLUDE [azure-monitor-samples](../../../includes/azure-monitor-resource-manager-samples.md)]


## <a name="create-association-between-azure-vm-and-rule"></a>Criar associação entre a VM do Azure e a regra

O exemplo a seguir instala o agente do Azure Monitor em uma máquina virtual do Microsoft Azure. Uma associação é criada entre uma máquina virtual do Azure e uma regra de coleta de dados.

### <a name="template-file"></a>Arquivo de modelo

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string",
            "metadata": {
                "description": "Name of the virtual machine."
            }
        },
        "associationName": {
            "type": "string",
            "metadata": {
                "description": "Name of the association."
            }
        },
        "dataCollectionRuleId": {
            "type": "string",
            "metadata": {
                "description": "Resource ID of the data collection rule."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Compute/virtualMachines/providers/dataCollectionRuleAssociations",
            "name": "[concat(parameters('vmName'),'/microsoft.insights/', parameters('associationName'))]",
            "apiVersion": "2019-11-01-preview",
            "properties": {
                "description": "Association of data collection rule. Deleting this association will break the data collection for this virtual machine.",
                "dataCollectionRuleId": "[parameters('dataCollectionRuleId')]"
            }
        }
    ]
}
```

### <a name="parameter-file"></a>Arquivo de parâmetro.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "vmName": {
        "value": "my-windows-vm"
      },
      "location": {
        "value": "eastus"
      }
  }
}
```

## <a name="create-association-between-azure-arc-and-rule"></a>Criar associação entre o Azure Arc e uma regra

O exemplo a seguir instala o agente do Azure Monitor em uma máquina virtual do Microsoft Azure. Uma associação é criada entre um computador de servidor habilitado para Azure Arc e uma regra de coleta de dados.

### <a name="template-file"></a>Arquivo de modelo

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string",
            "metadata": {
                "description": "Name of the virtual machine."
            }
        },
        "associationName": {
            "type": "string",
            "metadata": {
                "description": "Name of the association."
            }
        },
        "dataCollectionRuleId": {
            "type": "string",
            "metadata": {
                "description": "Resource ID of the data collection rule."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.HybridCompute/machines/providers/dataCollectionRuleAssociations",
            "name": "[concat(parameters('machineName'),'/microsoft.insights/', parameters('associationName'))]",
            "apiVersion": "2019-11-01-preview",
            "properties": {
                "description": "Association of data collection rule. Deleting this association will break the data collection for this Arc server.",
                "dataCollectionRuleId": "[parameters('dataCollectionRuleId')]"
            }
        }
    ]
}
```

### <a name="parameter-file"></a>Arquivo de parâmetro.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "vmName": {
        "value": "my-windows-vm"
      },
      "location": {
        "value": "eastus"
      }
  }
}
```


## <a name="next-steps"></a>Próximas etapas

* [Obtenha outros modelos de amostra do Azure Monitor](resource-manager-samples.md).
* [Saiba mais sobre o agente do Log Analytics](../platform/log-analytics-agent.md).
* [Saiba mais sobre a extensão de diagnóstico](../platform/diagnostics-extension-overview.md).
