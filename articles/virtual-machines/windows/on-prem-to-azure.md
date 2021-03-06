---
title: Migração do AWS e outras plataformas para os Managed Disks no Azure
description: Criação de VMs no Azure usando VHDs carregados de outras nuvens, como AWS ou outras plataformas de virtualização, e tirar proveito dos Azure Managed Disks.
author: roygara
manager: twooley
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: conceptual
ms.date: 10/07/2017
ms.author: rogarana
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 32cbfbcc8feeff66101ab5e2c95f476a4a4215e9
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91973896"
---
# <a name="migrate-from-amazon-web-services-aws-and-other-platforms-to-managed-disks-in-azure"></a>Migrar do AWS (Amazon Web Services) e outras plataformas para o Managed Disks no Azure

Você pode carregar arquivos VHD do AWS ou soluções de virtualização locais no Azure para criar VMs que aproveitam os Managed Disks. Com o Managed Disks do Azure, não é preciso gerenciar as contas de armazenamento para VMs IaaS do Azure. Você precisa apenas especificar o tipo (Premium ou Standard) e o tamanho do disco necessário e o Azure cria e gerencia o disco para você. 

Você pode carregar VHDs especializados e generalizados. 
- **VHD generalizado** – teve todas as informações da sua conta pessoal removidas usando o Sysprep. 
- **VHD especializado** – mantém as contas de usuário, aplicativos e outros dados de estado de sua VM original. 

> [!IMPORTANT]
> Antes de carregar qualquer VHD no Azure, você deve seguir [preparar um VHD do Windows ou VHDX para carregar no Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
>
>


| Cenário                                                                                                                         | Documentação                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Você tem instâncias EC2 do AWS existentes que gostaria de migrar para VMs do Azure usando o Managed Disks                              | [Mover uma VM do AWS (Amazon Web Services) para o Azure](aws-to-azure.md)                           |
| Você tem uma VM de outra plataforma de virtualização que deseja usar como uma imagem para criar várias VMs do Azure. | [Carregar um VHD generalizado e usá-lo para criar uma nova VM no Azure](upload-generalized-managed.md) |
| Você tem uma VM exclusivamente personalizada que deseja recriar no Azure.                                                      | [Carregar um VHD especializado para o Azure e criar uma nova VM](create-vm-specialized.md)         |


## <a name="overview-of-managed-disks"></a>Visão geral do Managed Disks

O Azure Managed Disks simplifica o gerenciamento de VM, eliminando a necessidade de gerenciar contas de armazenamento. O Managed Disks também fornece melhor confiabilidade de VMs em um conjunto de disponibilidade. Isso garante um isolamento suficiente dos discos de VMs diferentes em um conjunto de disponibilidade a fim de evitar um ponto único de falha. Ele coloca automaticamente os discos de VMs diferentes em um conjunto de disponibilidade em unidades de escala (carimbos) de armazenamentos diferentes, o que limita o impacto de falhas em uma única unidade de escala de armazenamento causadas por falhas de hardware e de software.
Com base em suas necessidades, você pode escolher entre quatro tipos de opções de armazenamento. Para saber mais sobre os tipos de disco disponíveis, consulte nosso artigo [selecionar um tipo de disco](../disks-types.md).

## <a name="plan-for-the-migration-to-managed-disks"></a>Como planejar a migração para os Managed Disks

Esta seção ajuda você a tomar a melhor decisão sobre VM e tipos de disco.

Se você estiver planejando sobre migração de discos não gerenciados para discos gerenciados, você deverá estar ciente de que os usuários com a função [Colaborador de Máquina Virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) não poderão alterar o tamanho da VM (como podiam antes da conversão). Isso ocorre porque as VMs com discos gerenciados requerem que o usuário tenha a permissão Microsoft.Compute/disks/write nos discos do sistema operacional.

### <a name="location"></a>Localização

Escolha um local onde o Azure Managed Disks estão disponíveis. Se você estiver migrando para Managed Disks Premium, verifique também se o armazenamento premium está disponível na região que você pretende migrar. Confira [Serviços do Azure por região](https://azure.microsoft.com/regions/#services) para obter informações atualizadas sobre as localizações disponíveis.

### <a name="vm-sizes"></a>Tamanhos de VM

Se você estiver migrando para Managed Disks Premium, será necessário atualizar o tamanho da VM para um tamanho compatível com o armazenamento premium disponível na região onde a VM está localizada. Examine os tamanhos de VM que são compatíveis com o armazenamento premium. As especificações de tamanho de VM do Azure são listadas em [Tamanhos para máquinas virtuais](../sizes.md).
Examine as características de desempenho das máquinas virtuais que funcionam com o Armazenamento Premium e escolha o tamanho de VM mais apropriado que melhor atende à sua carga de trabalho. Certifique-se de que há largura de banda suficiente disponível na sua VM para direcionar o tráfego de disco.

### <a name="disk-sizes"></a>Tamanhos do disco

**Managed Disks Premium**

Há sete tipos de discos gerenciados premium que podem ser usados com sua VM e cada um tem IOPs e limites de taxa de transferência específicos. Leve em consideração esses limites ao escolher o tipo de disco Premium para sua VM com base nas necessidades de seu aplicativo em termos de capacidade, desempenho, escalabilidade e cargas de pico.

| Tipo de discos premium  | P4    | P6    | P10   | P15   | P20   | P30   | P40   | P50   | 
|---------------------|-------|-------|-------|-------|-------|-------|-------|-------|
| Tamanho do disco           | 32 GB| 64 GB| 128 GB| 256 GB|512 GB | 1024 GB (1 TB)    | 2048 GB (2 TB)    | 4095 GB (4 TB)    | 
| IOPS por disco       | 120   | 240   | 500   | 1100  |2300              | 5.000              | 7500              | 7500              | 
| Taxa de transferência por disco | 25 MB por segundo  | 50 MB por segundo  | 100 MB por segundo | 125 MB por segundo |150 MB por segundo | 200 MB por segundo | 250 MB por segundo | 250 MB por segundo |

**Managed Disks padrão**

Há sete tipos de discos gerenciados standard que podem ser usados com sua VM. Cada um deles tem uma capacidade diferente, mas com os mesmos limites de taxa de transferência e IOPS. Escolha o tipo de discos gerenciados Standard com base nas necessidades de capacidade do seu aplicativo.

| Tipo de disco Standard  | S4               | S6               | S10              | S15              | S20              | S30              | S40              | S50              | 
|---------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------| 
| Tamanho do disco           | 30 GB            | 64 GB            | 128 GB           | 256 GB           |512 GB           | 1024 GB (1 TB)   | 2048 GB (2TB)    | 4095 GB (4 TB)   | 
| IOPS por disco       | 500              | 500              | 500              | 500              |500              | 500              | 500             | 500              | 
| Taxa de transferência por disco | 60 MB por segundo | 60 MB por segundo | 60 MB por segundo | 60 MB por segundo |60 MB por segundo | 60 MB por segundo | 60 MB por segundo | 60 MB por segundo | 

### <a name="disk-caching-policy"></a>Política de cache de disco 

**Managed Disks Premium**

Por padrão, a política de cache de disco é *somente leitura* para todos os discos de dados Premium e *leitura/gravação* para o disco do sistema operacional Premium anexado à VM. Esta definição de configuração é recomendável para atingir o desempenho ideal de leituras de entrada e saída dos seus aplicativos. Para discos de dados de gravação intensa ou somente gravação (como arquivos de log do SQL Server), desabilite o cache de disco para que possa obter o melhor desempenho do aplicativo.

### <a name="pricing"></a>Preços

Confira os [preços dos Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/). Os preços dos Managed Disks Premium são iguais aos dos discos não gerenciados premium. No entanto, os preços do Standard Managed Disks são diferentes dos preços do Standard Unmanaged Disks.


## <a name="next-steps"></a>Próximas etapas

- Antes de carregar qualquer VHD no Azure, você deve seguir [preparar um VHD do Windows ou VHDX para carregar no Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)