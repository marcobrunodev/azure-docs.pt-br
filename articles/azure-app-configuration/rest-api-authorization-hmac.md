---
title: API REST de configuração de Azure App-autorização de HMAC
description: Usar HMAC para autorização em relação à configuração de Azure App usando a API REST
author: lisaguthrie
ms.author: lcozzens
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 8d1f8a17f51711cc5c10567797e1224f96eb7630
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93423867"
---
# <a name="hmac-authorization---rest-api-reference"></a>Autorização de HMAC – referência da API REST

Quando a autenticação HMAC é usada, as operações caem em uma de duas categorias, leitura ou gravação. As chaves de acesso de leitura/gravação concedem permissão para chamar todas as operações. Chaves de acesso somente leitura concedem permissão para chamar somente operações de leitura. Se uma chave de acesso é somente leitura ou leitura/gravação é determinada por sua `readOnly` propriedade. Qualquer tentativa de fazer uma solicitação de gravação com uma chave de acesso somente leitura resultará na não autorização da solicitação.

## <a name="obtaining-access-keys"></a>Obtendo chaves de acesso

A especificação que descreve as chaves de acesso e a API usada para obtê-las é detalhada na especificação do provedor de recursos de configuração do Azure App [aqui](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/appconfiguration/resource-manager/Microsoft.AppConfiguration/stable/2019-10-01/appconfiguration.json). As chaves de acesso são obtidas por meio da operação "ConfigurationStores_ListKeys".

## <a name="errors"></a>Errors

```http
HTTP/1.1 403 Forbidden
```

**Motivo:** A chave de acesso usada para autenticar a solicitação não fornece as permissões necessárias para executar a operação solicitada.
**Solução:** Obtenha uma chave de acesso que forneça permissão para executar a operação solicitada e use-a para autenticar a solicitação.
