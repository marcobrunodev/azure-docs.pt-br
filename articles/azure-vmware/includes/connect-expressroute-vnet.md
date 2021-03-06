---
title: Conectar o ExpressRoute ao gateway de rede virtual
description: Etapas para conectar o ExpressRoute ao gateway de rede virtual.
ms.topic: include
ms.date: 09/28/2020
ms.openlocfilehash: 214ef9c01193b238c8e456ef2809f7a2edbdb6c7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91598183"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-configure-networking.md -->

1. Navegue até a nuvem privada criada no tutorial anterior, selecione **Conectividade** em **Gerenciar** e selecione a guia **ExpressRoute**.

1. Copie a chave de autorização. Se não houver uma chave de autorização, você precisará criar uma: selecione **+ Solicitar uma chave de autorização**.

   :::image type="content" source="../media/expressroute-global-reach/start-request-auth-key.png" alt-text="Copie a chave de autorização. Se não houver uma chave de autorização, você precisará criar uma: selecione + Solicitar uma chave de autorização" border="true" lightbox="../media/expressroute-global-reach/start-request-auth-key.png":::.

1. Navegue até o Gateway de Rede Virtual criado na etapa anterior e, em **Configurações**, selecione **Conexões**. Na página **Conexões**, selecione **+ Adicionar**.

1. Na página **Adicionar conexão**, forneça valores para os campos e selecione **OK**. 

   | Campo | Valor |
   | --- | --- |
   | **Nome**  | Insira um nome para a conexão.  |
   | **Tipo de conexão**  | Selecione **ExpressRoute**.  |
   | **Resgatar autorização**  | Esta caixa deve estar selecionada.  |
   | **Gateway de rede virtual** | O gateway de Rede Virtual criado anteriormente.  |
   | **Chave de autorização**  | Copie e cole a chave de autorização da guia ExpressRoute do seu Grupo de recursos. |
   | **URI de circuito de par**  | Copie e cole a ID do ExpressRoute da guia ExpressRoute do seu Grupo de recursos.  |

   :::image type="content" source="../media/expressroute-global-reach/open-cloud-shell.png" alt-text="Copie a chave de autorização. Se não houver uma chave de autorização, você precisará criar uma: selecione + Solicitar uma chave de autorização" border="true" lightbox="../media/expressroute-global-reach/open-cloud-shell.png":::

A conexão entre o circuito do ExpressRoute e sua Rede Virtual é criada.