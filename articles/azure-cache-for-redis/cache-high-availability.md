---
title: Alta disponibilidade para o cache do Azure para Redis
description: Saiba mais sobre o cache do Azure para opções e recursos de alta disponibilidade do Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 10/28/2020
ms.author: yegu
ms.openlocfilehash: e44aed1415f85bf4ea597eac6720207301946b97
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93076904"
---
# <a name="high-availability-for-azure-cache-for-redis"></a>Alta disponibilidade para o cache do Azure para Redis

O cache do Azure para Redis tem alta disponibilidade interna. O objetivo de sua arquitetura de alta disponibilidade é garantir que sua instância Redis gerenciada esteja funcionando mesmo quando suas VMs (máquinas virtuais) subjacentes forem afetadas por interrupções planejadas ou não planejadas. Ele fornece taxas de percentual muito maiores do que o que é atingível ao hospedar Redis em uma única VM.

O cache do Azure para Redis implementa alta disponibilidade usando várias VMs, chamadas de *nós* , para um cache. Ele configura esses nós de modo que a replicação de dados e o failover ocorram no modos coordenado. Ele também orquestra operações de manutenção, como a aplicação de patches de software Redis. Várias opções de alta disponibilidade estão disponíveis nas camadas Standard, Premium e Enterprise:

| Opção | Descrição | Disponibilidade | Standard | Premium | Enterprise |
| ------------------- | ------- | ------- | :------: | :---: | :---: |
| [Replicação padrão](#standard-replication)| Configuração replicada de nó duplo em um único datacenter ou em uma zona de disponibilidade (AZ) com failover automático | 99,9% |✔|✔|-|
| [Cluster corporativo](#enterprise-cluster) | Instâncias de cache vinculadas em duas regiões, com failover automático | 99,9% |-|-|✔|
| [Redundância de zona](#zone-redundancy) | Configuração replicada de vários nós em AZs, com failover automático | 99,95% (replicação padrão), 99,99% (cluster corporativo) |-|✔|✔|
| [Replicação geográfica](#geo-replication) | Instâncias de cache vinculadas em duas regiões, com failover controlado pelo usuário | 99,9% (para uma única região) |-|✔|-|

## <a name="standard-replication"></a>Replicação padrão

Um cache do Azure para Redis na camada Standard ou Premium é executado em um par de servidores Redis por padrão. Os dois servidores são hospedados em VMs dedicadas. O Redis de código-fonte aberto permite que apenas um servidor manipule solicitações de gravação de dados. Esse servidor é o nó *primário* , enquanto a outra *réplica* . Depois de provisionar os nós de servidor, o cache do Azure para Redis atribui funções primárias e de réplica a eles. O nó primário geralmente é responsável por atender a gravação, bem como solicitações de leitura de clientes Redis. Em uma operação de gravação, ele confirma uma nova chave e uma atualização de chave para sua memória interna e responde imediatamente ao cliente. Ele encaminha a operação para a réplica de forma assíncrona.

:::image type="content" source="media/cache-high-availability/replication.png" alt-text="Configuração de replicação de dados":::
   
>[!NOTE]
>Normalmente, um cliente Redis se comunica com o nó primário em um cache Redis para todas as solicitações de leitura e gravação. Determinados clientes Redis podem ser configurados para leitura a partir do nó de réplica.
>
>

Se o nó primário em um cache Redis estiver indisponível, a réplica se promoverá para se tornar o novo primário automaticamente. Esse processo é chamado de *failover* . A réplica aguardará por tempo suficiente antes de assumir o caso de o nó primário se recuperar rapidamente. Quando ocorre um failover, o cache do Azure para Redis provisiona uma nova VM e a une ao cache como o nó de réplica. A réplica executa uma sincronização de dados completa com o primário para que ele tenha outra cópia dos dados de cache.

Um nó primário pode sair do serviço como parte de uma atividade de manutenção planejada, como Redis software ou atualização do sistema operacional. Ele também pode parar de funcionar devido a eventos não planejados, como falhas em hardware, software ou rede subjacentes. [Failover e aplicação de patch para o cache do Azure para Redis](cache-failover.md) fornece uma explicação detalhada sobre os tipos de failovers do Redis. Um cache do Azure para Redis passará por vários failovers durante seu tempo de vida. A arquitetura de alta disponibilidade foi projetada para tornar essas alterações dentro de um cache tão transparente para seus clientes quanto possível.

>[!NOTE]
>O seguinte está disponível como uma visualização.
>
>

Além disso, o cache do Azure para Redis permite nós de réplica adicionais na camada Premium. Um [cache de várias réplicas](cache-how-to-multi-replicas.md) pode ser configurado com até três nós de réplica. Ter mais réplicas geralmente melhora a resiliência devido aos nós adicionais que fazem backup do primário. Mesmo com mais réplicas, um cache do Azure para instância Redis ainda pode ser seriamente impactado por uma interrupção de Datacenter ou AZ-Wide. Você pode aumentar a disponibilidade do cache usando várias réplicas em conjunto com a [redundância de zona](#zone-redundancy).

## <a name="enterprise-cluster"></a>Cluster corporativo

>[!NOTE]
>Isso está disponível como uma visualização.
>
>

Um cache em uma camada corporativa é executado em um cluster do Redis Enterprise. Ele requer um número ímpar de nós de servidor em todos os momentos para formar um quorum. Por padrão, ele é composto por três nós, cada um hospedado em uma VM dedicada. Um cache corporativo tem dois *nós de dados* de mesmo tamanho e um *nó de quorum* menor. Um cache flash corporativo tem três nós de dados de tamanho igual. O cluster corporativo divide os dados do Redis em partições internamente. Cada partição tem um *primário* e pelo menos uma *réplica* . Cada nó de dados contém uma ou mais partições. O cluster corporativo garante que as réplicas e primárias de qualquer partição nunca sejam colocalizadas no mesmo nó de dados. As partições replicam dados de forma assíncrona de primárias para suas réplicas correspondentes.

Quando um nó de dados fica indisponível ou ocorre uma divisão de rede, um failover semelhante ao descrito na [replicação padrão](#standard-replication) ocorre. O cluster corporativo usa um modelo baseado em quorum para determinar quais nós sobreviventes participarão de um novo quorum. Ele também promove partições de réplica dentro desses nós para os primários, conforme necessário.

## <a name="zone-redundancy"></a>Redundância de zona

>[!NOTE]
>Isso está disponível como uma visualização.
>
>

O cache do Azure para Redis dá suporte a configurações com redundância de zona nas camadas Premium e Enterprise. Um [cache com redundância de zona](cache-how-to-zone-redundancy.md) pode colocar seus nós entre diferentes [zonas de disponibilidade do Azure](../availability-zones/az-overview.md) na mesma região. Ele elimina o datacenter ou AZ a interrupção como um ponto único de falha e aumenta a disponibilidade geral do cache.

O diagrama a seguir ilustra a configuração com redundância de zona:

:::image type="content" source="media/cache-high-availability/zone-redundancy.png" alt-text="Configuração de replicação de dados":::
   
O cache do Azure para Redis distribui os nós em um cache com redundância de zona em um modo Round Robin sobre o AZs que você selecionou. Ele também determina qual nó servirá como o primário inicialmente.

Um cache com redundância de zona fornece failover automático. Quando o nó primário atual não estiver disponível, uma das réplicas assumirá. Seu aplicativo poderá apresentar um tempo de resposta de cache maior se o novo nó primário estiver localizado em um AZ diferente. AZs estão geograficamente separados. Alternar de um AZ para outro altera a distância física entre onde o aplicativo e o cache estão hospedados. Essa alteração afeta as latências de rede de ida e volta de seu aplicativo para o cache. Espera-se que a latência extra se enquadrasse em um intervalo aceitável para a maioria dos aplicativos. Recomendamos que você teste seu aplicativo para garantir que ele possa funcionar bem com um cache com redundância de zona.

## <a name="geo-replication"></a>Replicação geográfica

A replicação geográfica é projetada principalmente para recuperação de desastres. Ele oferece a capacidade de configurar um cache do Azure para a instância do Redis, em uma região do Azure diferente, para fazer backup do cache primário. [Configurar a replicação geográfica para o cache do Azure para Redis](cache-how-to-geo-replication.md) fornece uma explicação detalhada de como a replicação geográfica funciona.

## <a name="next-steps"></a>Próximas etapas

Saiba mais sobre como configurar o cache do Azure para opções de alta disponibilidade do Redis.

* [Cache do Azure para camadas de serviço do Redis Premium](cache-overview.md#service-tiers)
* [Adicionar réplicas ao cache do Azure para Redis](cache-how-to-multi-replicas.md)
* [Habilitar a redundância de zona para o cache do Azure para Redis](cache-how-to-zone-redundancy.md)
* [Configurar a replicação geográfica para o cache do Azure para Redis](cache-how-to-geo-replication.md)