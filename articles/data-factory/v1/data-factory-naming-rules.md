---
title: Regras para nomear entidades de Azure Data Factory-versão 1
description: Descreve regras de nomenclatura para entidades Data Factory v1.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.assetid: bc5e801d-0b3b-48ec-9501-bb4146ea17f1
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 0ef997f54deafc56f1b3af603071c0e650be40d4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495388"
---
# <a name="rules-for-naming-azure-data-factory-entities"></a>Regras para nomear entidades de Azure Data Factory
> [!NOTE]
> Este artigo aplica-se à versão 1 do Data Factory. Se você estiver usando a versão atual do serviço Data Factory, consulte [regras de nomenclatura no Data Factory](../naming-rules.md).

A tabela a seguir fornece regras de nomenclatura para artefatos Data Factory.

| Nome | Exclusividade do nome | Verificações de validação |
|:--- |:--- |:--- |
| Data Factory |Exclusivo em todo o Microsoft Azure. Os nomes não diferenciam maiúsculas de minúsculas, isto é, `MyDF` e `mydf` referem-se ao mesmo data factory. |<ul><li>Cada data factory está vinculado a exatamente uma assinatura do Azure.</li><li>Os nomes do objeto devem começar com uma letra ou número e podem conter apenas letras, números e o caractere traço (-).</li><li>Cada caractere traço (-) deve ser imediatamente precedido e seguido por uma letra ou um número. Não há permissão para traços consecutivos em nomes de contêiner.</li><li>O nome pode ter de 3 a 63 caracteres.</li></ul> |
| Serviços/tabelas/pipelines vinculados |Exclusivo em um mesmo data factory. Os nomes não diferenciam maiúsculas de minúsculas. |<ul><li>Número máximo de caracteres em um nome de tabela: 260.</li><li>Os nomes de objetos devem começar com uma letra, um número ou um sublinhado (_).</li><li>Os seguintes caracteres não são permitidos: “.”, “+”, “?”, “/”, “<”, ”>”,” * ”,”%”,”&”,”:”,”\\”</li></ul> |
| Grupo de recursos |Exclusivo em todo o Microsoft Azure. Os nomes não diferenciam maiúsculas de minúsculas. |<ul><li>Número máximo de caracteres: 1000.</li><li>O nome pode conter letras, dígitos e os seguintes caracteres: “-”, “_”, “,” e “.”</li></ul> |

