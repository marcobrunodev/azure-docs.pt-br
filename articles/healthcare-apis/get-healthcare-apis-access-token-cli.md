---
title: Obter token de acesso usando o CLI do Azure-API do Azure para FHIR
description: Este artigo explica como obter um token de acesso para a API do Azure para FHIR usando o CLI do Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/26/2019
ms.author: matjazl
ms.openlocfilehash: 4d1c4cfcb15d97a2c54a04344f0bd098f65c1392
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660365"
---
# <a name="get-access-token-for-azure-api-for-fhir-using-azure-cli"></a>Obter token de acesso para a API do Azure para FHIR usando CLI do Azure

Neste artigo, você aprenderá a obter um token de acesso para a API do Azure para FHIR usando o CLI do Azure. Ao [provisionar a API do Azure para FHIR](fhir-paas-portal-quickstart.md), você configura um conjunto de usuários ou entidades de serviço que têm acesso ao serviço. Se sua ID de objeto de usuário estiver na lista de IDs de objeto permitidas, você poderá acessar o serviço usando um token obtido usando o CLI do Azure.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="obtain-a-token"></a>Obter um token

A API do Azure para FHIR usa `resource`  um `Audience` URI ou com igual ao URI do servidor FHIR `https://<FHIR ACCOUNT NAME>.azurehealthcareapis.com` . Você pode obter um token e armazená-lo em uma variável (chamada `$token` ) com o seguinte comando:

```azurecli-interactive
token=$(az account get-access-token --resource=https://<FHIR ACCOUNT NAME>.azurehealthcareapis.com --query accessToken --output tsv)
```

## <a name="use-with-azure-api-for-fhir"></a>Usar com a API do Azure para FHIR

```azurecli-interactive
curl -X GET --header "Authorization: Bearer $token" https://<FHIR ACCOUNT NAME>.azurehealthcareapis.com/Patient
```

## <a name="next-steps"></a>Próximas etapas

Neste artigo, você aprendeu a obter um token de acesso para a API do Azure para FHIR usando o CLI do Azure. Para saber como acessar a API do FHIR usando o Postman, prossiga para o tutorial do Postman.

>[!div class="nextstepaction"]
>[Acessar a API do FHIR usando o Postman](access-fhir-postman-tutorial.md)
