---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/18/2020
ms.author: glenga
ms.openlocfilehash: eb54439f89cc2443eeed2d3b63dfbe7fedb4bf17
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/05/2020
ms.locfileid: "80673489"
---
## <a name="update-the-tests"></a>Atualizar os testes

Como o arquétipo também cria um conjunto de testes, você precisa atualizar esses testes para manipular o novo parâmetro `msg` na assinatura do método `run`.  

Procure a localização do código de teste em _src/test/java_, abra o arquivo de projeto *Function.java* e substitua a linha de código em `//Invoke` pelo código a seguir.

:::code language="java" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/src/test/java/com/function/FunctionTest.java" range="48-50":::
