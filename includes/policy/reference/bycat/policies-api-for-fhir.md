---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/20/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 95aa94f8e1459459a941c96e0d06d13d76a38879
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/20/2020
ms.locfileid: "96007457"
---
|Nome<br /><sub>(Portal do Azure)</sub> |Descrição |Efeito(s) |Versão<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[A API do Azure para FHIR deve usar uma CMK (chave gerenciada pelo cliente) para criptografar dados em repouso](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F051cba44-2429-45b9-9649-46cec11c7119) |Use uma chave gerenciada pelo cliente para controlar a criptografia em repouso dos dados armazenados na API do Azure para FHIR quando esse for um requisito regulatório ou de conformidade. As chaves gerenciadas pelo cliente também fornecem criptografia dupla adicionando uma segunda camada de criptografia além do padrão um feito com chaves gerenciadas por serviço. |auditoria, desabilitado |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_EnableByok_Audit.json) |
|[A API do Azure para FHIR deve usar um link privado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1ee56206-5dd1-42ab-b02d-8aae8b1634ce) |A API do Azure para FHIR deve ter, pelo menos, uma conexão de ponto de extremidade privado aprovada. Os clientes em uma rede virtual podem acessar com segurança recursos que têm conexões de ponto de extremidade privado por meio de links privados. Para obter mais informações, visite: [https://aka.ms/fhir-privatelink](https://aka.ms/fhir-privatelink). |Audit, desabilitado |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_PrivateLink_Audit.json) |
|[CORS não deve permitir que todos os domínios acessem sua API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d) |O CORS (compartilhamento de recurso de origem cruzada) não deve permitir que todos os domínios acessem sua API para FHIR. Para proteger sua API para FHIR, remova o acesso de todos os domínios e defina explicitamente os domínios com permissão para se conectar. |auditoria, desabilitado |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
