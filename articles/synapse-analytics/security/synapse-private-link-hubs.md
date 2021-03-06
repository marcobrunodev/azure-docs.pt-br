---
title: Conectar-se a um Synapse Studio usando links privados
description: Este artigo ensinará como se conectar ao Azure Synapse Studio usando links privados
author: NanditaV
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: security
ms.date: 12/01/2020
ms.author: NanditaV
ms.reviewer: jrasnick
ms.openlocfilehash: 2613a4fd931ad49a4f40a4221ea20e8c25f185fe
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501387"
---
# <a name="connect-to-azure-synapse-studio-using-azure-private-link-hubs"></a>Conectar-se ao Azure Synapse Studio usando hubs de link privado do Azure 

Este artigo explica como os links privados dos hubs de link privado do Azure Synapse Analytics são usados para se conectar com segurança ao Synapse Studio. 

## <a name="azure-private-link-hubs"></a>Hubs de link privado do Azure 
Você pode se conectar com segurança ao Azure Synapse Studio de sua rede virtual do Azure usando links privados. Os hubs de link privado do Azure Synapse Analytics são recursos do Azure que atuam como conectores entre sua rede segura e a experiência da Web do Synapse Studio. 

Há duas etapas para se conectar ao Synapse Studio usando links privados. Primeiro, você deve criar um recurso de hubs de link privado. Em segundo lugar, você deve criar um ponto de extremidade privado de sua rede virtual do Azure para esse Hub de link privado. Você pode usar pontos de extremidade privados para se comunicar com segurança com o Synapse Studio. Você deve integrar os pontos de extremidade privados com sua solução DNS, seja sua solução local ou DNS privado do Azure. 

## <a name="azure-private-links-hubs-and-azure-synapse-studio"></a>Hubs de links privados do Azure e o Azure Synapse Studio
Você pode usar um único recurso do hub de link privado do Azure Synapse para se conectar de forma privada a todos os seus espaços de trabalho do Azure Synapse Analytics usando o Azure Synapse Studio. Os espaços de trabalho não precisam estar na mesma região que o Hub de link privado do Azure Synapse. O recurso de Hub de link privado do Azure Synapse também pode ser usado para conexões com espaços de trabalho Synapse em assinaturas diferentes ou locatários do Azure AD.

Você pode criar seu hub de link privado procurando por *hubs de link Synapse privado* no portal do Azure e selecionando **análise de Synapse do Azure (hubs de link privado)** de serviços. Siga as etapas no guia para saber como [se conectar a recursos de espaço de trabalho de uma rede restrita](./how-to-connect-to-workspace-from-restricted-network.md) para obter detalhes.

>[!NOTE]
>Os hubs de link privado destinam-se a carregar com segurança o conteúdo estático Synapse Studio sobre links privados. Você deve criar **pontos de extremidade privados e separados** para os recursos aos quais deseja se conectar no espaço de trabalho, como pools do SQL provisionados/dedicados ou pools do Spark. 

## <a name="azure-private-links-hubs-and-azure-virtual-network"></a>Hubs de links privados do Azure e rede virtual do Azure
Você deve conectar sua rede virtual do Azure ao recurso Synapse Private link Hub para proteger a conexão de ponta a ponta ao Synapse Studio. Para isso, você deve criar um ponto de extremidade privado de sua rede virtual para o Hub de link privado que você criou. Você pode usar o portal do Azure para o seu hub de link privado e ir para a seção ponto de extremidade particular. Selecione "+ ponto de extremidade privado" para criar um novo ponto de extremidade privado que se conecta ao seu hub de link privado.

:::image type="content" source="./media/synapse-private-link-hubs/synapse-private-links-private-endpoint.png" alt-text="Criar um ponto de extremidade privado para o Hub de link privado":::

Certifique-se de escolher o tipo de recurso "Microsoft. Synapse/privateLinkHubs" na guia "recurso". :::image type="content" source="./media/synapse-private-link-hubs/synapse-private-links-resource-type.png" alt-text="criar um ponto de extremidade privado para o Hub de link privado":::

Na guia "configuração", selecione "privatelink.azuresynapse.net" para zonas de DNS privado ao integrar com sua rede virtual e zona DNS privada.

:::image type="content" source="./media/synapse-private-link-hubs/synapse-private-links-dns-zones.png" alt-text="Criar um ponto de extremidade privado para o Hub de link privado":::

## <a name="next-steps"></a>Próximas etapas

[Conectar-se a recursos de espaço de trabalho de uma rede restrita](./how-to-connect-to-workspace-from-restricted-network.md)

Saiba mais sobre a [rede virtual do espaço de trabalho gerenciado](./synapse-workspace-managed-vnet.md)

Saiba mais sobre [Pontos de extremidade privados gerenciados](./synapse-workspace-managed-private-endpoints.md)

[Criar Pontos de extremidade privados gerenciados para suas fontes de dados](./how-to-create-managed-private-endpoints.md)

[Conectar-se ao espaço de trabalho Synapse usando pontos de extremidade privados](./how-to-connect-to-workspace-with-private-links.md)

