---
title: Projetar e implementar um banco de dados Oracle no Azure | Microsoft Docs
description: Projete e implemente um banco de dados Oracle no seu ambiente do Azure.
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: 5e9ddecd694a9051e746d07cbc1bee4d98bf5829
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484423"
---
# <a name="design-and-implement-an-oracle-database-in-azure"></a>Projete e implemente um banco de dados Oracle no Azure

## <a name="assumptions"></a>Suposições

- Você está planejando a migração de um banco de dados Oracle do local para o Azure.
- Você tem o [pacote de diagnóstico](https://docs.oracle.com/cd/E11857_01/license.111/e11987/database_management.htm) para o Oracle Database que pretende migrar
- Você tem uma compreensão das várias métricas dos relatórios do Oracle AWR.
- Você tem uma compreensão das linha de base do desempenho de aplicativo e da utilização da plataforma.

## <a name="goals"></a>Metas

- Compreender como otimizar sua implantação do Oracle no Azure.
- Explorar as opções de ajuste de desempenho para um banco de dados Oracle em um ambiente do Azure.

## <a name="the-differences-between-an-on-premises-and-azure-implementation"></a>As diferenças entre uma implementação local e no Azure 

Veja a seguir algumas coisas importantes a ter em mente ao migrar aplicativos locais para o Azure. 

Uma diferença importante é que, em uma implementação do Azure, recursos como VMs, discos e redes virtuais são compartilhados entre outros clientes. Além disso, os recursos podem ser limitados de acordo com os requisitos. Em vez de se concentrar em evitar falhas (MTBF), o Azure é mais focado em sobreviver a elas (MTTR).

A tabela a seguir lista algumas das diferenças entre uma implementação local e uma implementação do Azure de um banco de dados Oracle.


|  | Implementação local | Implementação no Azure |
| --- | --- | --- |
| **Rede** |LAN/WAN  |SDN (Rede definida por software)|
| **Grupo de segurança** |Ferramentas de restrições de IP/porta |[Grupo de segurança de rede (NSG)](https://azure.microsoft.com/blog/network-security-groups) |
| **Resiliência** |MTBF (tempo médio entre falhas) |MTTR (tempo médio para recuperação)|
| **Manutenção planejada** |Aplicação de patch/upgrades|[Conjuntos de disponibilidade](/previous-versions/azure/virtual-machines/windows/infrastructure-example) (aplicação de patch/upgrades gerenciados pelo Azure) |
| **Recurso** |Dedicado  |Compartilhado com outros clientes|
| **Regiões** |Datacenters |[Pares de regiões](../../regions.md#region-pairs)|
| **Storage** |SAN/discos físicos |[Armazenamento gerenciado pelo Azure](https://azure.microsoft.com/pricing/details/managed-disks/?v=17.23h)|
| **Dimensionar** |Escala vertical |Escala horizontal|


### <a name="requirements"></a>Requisitos

- Determinar o tamanho do banco de dados e sua taxa de crescimento.
- Determinar os requisitos de IOPS, que podem ser avaliados com base nos relatórios do Oracle AWR ou outras ferramentas de monitoramento de rede.

## <a name="configuration-options"></a>Opções de configuração

Há quatro áreas possíveis que você pode ajudar para melhorar o desempenho em um ambiente do Azure:

- Tamanho da máquina virtual
- Taxa de transferência de rede
- Tipos e configurações de disco
- Configurações de cache de disco

### <a name="generate-an-awr-report"></a>Gerar um relatório AWR

Se você tiver um banco de dados Oracle existente e estiver planejando a migração para o Azure, terá várias opções. Se você tiver o [pacote de diagnóstico](https://www.oracle.com/technetwork/oem/pdf/511880.pdf) para suas instâncias do Oracle, poderá executar o relatório AWR da Oracle para obter as métricas (IOPS, Mbps, GiBs e assim por diante). Em seguida, escolha a VM com base nas métricas que você coletou. Ou então, contate sua equipe de infraestrutura para obter informações semelhantes.

Considere a execução do relatório AWR durante cargas de trabalho regulares e de pico para poder comparar a diferença. Com base nesses relatórios, você pode dimensionar as VMs com base na carga de trabalho média ou na carga de trabalho máxima.

Veja a seguir um exemplo de como gerar um relatório AWR (gerar seus relatórios AWR usando o Oracle Enterprise Manager, se a instalação atual tiver um):

```bash
$ sqlplus / as sysdba
SQL> EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT;
SQL> @?/rdbms/admin/awrrpt.sql
```

### <a name="key-metrics"></a>Métricas-chave

Veja a seguir as métricas que você pode obter do relatório AWR:

- Número total de núcleos
- Velocidade de clock da CPU
- Memória total em GB
- Utilização da CPU
- Pico da taxa de transferência de dados
- Taxa de alterações de E/S (leitura/gravação)
- Taxa de log da fase refazer (MBPs)
- Taxa de transferência de rede
- Taxa de latência de rede (baixa/alta)
- Tamanho do banco de dados em GB
- Bytes recebidos por meio de SQL \*Net de/para o cliente

### <a name="virtual-machine-size"></a>Tamanho da máquina virtual

#### <a name="1-estimate-vm-size-based-on-cpu-memory-and-io-usage-from-the-awr-report"></a>1. estimar o tamanho da VM com base na CPU, memória e uso de e/s do relatório AWR

Examine os cinco eventos de primeiro plano maior tempo, que indicam onde estão os gargalos do sistema.

Por exemplo, no diagrama a seguir, a sincronização de arquivos de log está na parte superior. Isso indica o número de esperas necessárias antes que o LGWR grave o buffer de log no arquivo de log da fase refazer. Esses resultados indicam que há a necessidade de armazenamento ou discos com um desempenho melhor. Além disso, o diagrama também mostra o número de CPU (núcleos) e da quantidade de memória.

![Captura de tela que mostra a sincronização do arquivo de log na parte superior da tabela.](./media/oracle-design/cpu_memory_info.png)

O diagrama a seguir mostra o total de E/S de leitura e gravação. 59 GB de leitura e 247,3 GB de gravação ocorreram durante o período do relatório.

![Captura de tela que mostra a e/s total de leitura e gravação.](./media/oracle-design/io_info.png)

#### <a name="2-choose-a-vm"></a>2. escolher uma VM

Com base nas informações coletadas do relatório AWR, a próxima etapa é escolher uma VM com um tamanho parecido que atenda às suas necessidades. Encontre uma lista de VMs disponíveis no artigo [Memória otimizada](../../sizes-memory.md).

#### <a name="3-fine-tune-the-vm-sizing-with-a-similar-vm-series-based-on-the-acu"></a>3. Ajuste o dimensionamento da VM com uma série de VMs semelhante com base no ACU

Depois de escolher a VM, preste atenção às ACUs das VMs. Você pode escolher uma VM diferente com base no valor da ACU que atender melhor às suas necessidades. Para saber mais, confira [Unidade de computação do Azure](../../acu.md).

![Captura de tela da página de unidades de ACU](./media/oracle-design/acu_units.png)

### <a name="network-throughput"></a>Taxa de transferência de rede

O diagrama a seguir mostra a relação entre a taxa de transferência e o IOPS:

![Captura de tela da taxa de transferência](./media/oracle-design/throughput.png)

A taxa de transferência de rede total é estimada com base nas seguintes informações:
- Tráfego SQL \*Rede
- MBps x número de servidores (fluxo de saída, como o Oracle Data Guard)
- Outros fatores, como replicação de aplicativo

![Captura de tela da taxa de transferência SQL*Rede](./media/oracle-design/sqlnet_info.png)

Com base nos requisitos de largura de banda de sua rede, há vários tipos de gateway para escolher. Entre eles basic, VpnGw e Azure ExpressRoute. Para saber mais, confira a [página de preços do gateway de VPN](https://azure.microsoft.com/pricing/details/vpn-gateway/?v=17.23h).

**Recomendações**

- A latência de rede é maior em comparação com uma implantação local. Reduzir as viagens de ida e volta da rede pode melhorar significativamente o desempenho.
- Para reduzir as viagens de ida e volta, consolide os aplicativos que têm transações alta ou aplicativos “comunicativos” na mesma máquina virtual.
- Use máquinas virtuais com [rede acelerada](../../../virtual-network/create-vm-accelerated-networking-cli.md) para melhorar o desempenho da rede.
- Para determinadas distribuições do Linux, considere habilitar o [suporte a corte/desmapeamento](/previous-versions/azure/virtual-machines/linux/configure-lvm#trimunmap-support).
- Instale o [Oracle Enterprise Manager](https://www.oracle.com/technetwork/oem/enterprise-manager/overview/index.html) em uma máquina virtual separada.
- As páginas enormes não são habilitadas no Linux por padrão. Considere habilitar páginas enormes e definir `use_large_pages = ONLY` no Oracle DB. Isso pode ajudar a aumentar o desempenho. Encontre mais informações [aqui](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/refrn/USE_LARGE_PAGES.html#GUID-1B0F4D27-8222-439E-A01D-E50758C88390).

### <a name="disk-types-and-configurations"></a>Tipos e configurações de disco

- *Discos padrão do SO*: esses tipos de disco oferecem dados persistentes e cache. Eles são otimizados para acesso de SO na inicialização, e não foram projetados para cargas de trabalho transacionais ou de data warehouse (analíticas).

- *Discos não gerenciados*: com esses tipos de disco, você gerencia as contas de armazenamento que armazenam os arquivos de VHD (disco rígido virtual) que correspondem aos seus discos de VM. Os arquivos VHD são armazenados como blobs de páginas nas contas de armazenamento do Azure.

- *Discos gerenciados*: o Azure gerencia contas de armazenamento que podem ser usadas para seus discos de VM. Especifique o tipo de disco (premium ou standard) e o tamanho do disco de que você precisa. O Azure cria e gerencia o disco para você.

- *Discos de armazenamento premium*: esses tipos de disco são ideais para cargas de trabalho de produção. O armazenamento premium dá suporte a discos de VM que podem ser anexados a VMs com série e tamanho específicos, por exemplo, das séries DS, DSv2, GS e F. O disco premium vem com tamanhos diferentes, e você pode optar entre discos de 32 GB a 4.096 GB. Cada tamanho de disco tem suas próprias especificações de desempenho. Dependendo dos requisitos do aplicativo, você pode anexar um ou mais discos à VM.

Ao criar um novo disco gerenciado no portal, você pode escolher o **Tipo de conta** para o tipo de disco que você deseja usar. Lembre-se de que nem todos os discos disponíveis aparecem no menu suspenso. Depois de escolher um tamanho de VM específico, o menu mostrará apenas os SKUs de armazenamento premium disponíveis com base no tamanho da VM.

![Captura de tela da página de disco gerenciado](./media/oracle-design/premium_disk01.png)

Depois de configurar o armazenamento em uma VM, convém realizar um teste de carga nos discos antes de criar um banco de dados. Saber a taxa de E/S em termos de latência e taxa de transferência pode ajudar a determinar se as VMs dão suporte às metas de taxa de transferência e latência esperadas.

Existem diversas ferramentas para realizar o teste de carga do aplicativo, como Oracle Orion, Sysbench e Fio.

Execute o teste de carga novamente depois de implantar um banco de dados Oracle. Inicie suas cargas de trabalho de pico e regulares, e os resultados mostram a linha de base de seu ambiente.

Talvez seja mais importante medir o tamanho do armazenamento com base na taxa de IOPS em vez do tamanho do armazenamento em si. Por exemplo, se o IOPS exigido for 5.000, mas você só precisar de 200 GB, você ainda poderá obter o disco premium de classe P30, embora ele venha com mais de 200 GB de armazenamento.

A taxa de IOPS pode ser obtida no relatório do AWR. Ela é determinada pela taxa de gravações, leituras físicas e log da fase refazer.

![Captura de tela da página do relatório AWR](./media/oracle-design/awr_report.png)

Por exemplo, o tamanho da fase refazer é 12.200.000 bytes por segundo, que é igual a 11,63 MBPs.
O IOPS é 12.200.000 / 2.358 = 5.174.

Quando você tiver uma visão clara dos requisitos de E/S, poderá escolher a combinação de unidades mais adequada para atender a esses requisitos.

**Recomendações**

- Para o espaço de tabela de dados, distribua a carga de trabalho de E/S entre diversos discos usando o armazenamento gerenciado ou o Oracle ASM.
- À medida que o tamanho do bloco de E/S aumentar para operações de leitura e gravação intensas, adicione mais discos de dados.
- Aumente o tamanho do bloco para processos sequenciais grandes.
- Use a compactação de dados para reduzir E/S (para dados e índices).
- Separe os logs da fase refazer, sistema e temps e desfaça o TS em discos de dados separados.
- Não coloque arquivos de aplicativo em discos do SO padrão (/dev/sda). Esses discos são otimizados inicializações rápidas de VM, e talvez não forneçam um bom desempenho para seu aplicativo.
- Ao usar VMs da série M no armazenamento Premium, habilite [acelerador de gravação](../../how-to-enable-write-accelerator.md) no disco de logs de restauração.

### <a name="disk-cache-settings"></a>Configurações de cache de disco

Há três opções para o cache de host:

- *ReadOnly*: todas as solicitações são armazenadas em cache para leituras futuras. Todas as gravações são persistidas diretamente no armazenamento de Blobs do Azure.

- *ReadWrite*: é um algoritmo "Read-Ahead". As leituras e gravações são armazenadas em cache para futuras leituras. Gravações não write-through são persistidas no cache local primeiro. Ele também fornece a menor latência de disco para cargas de trabalho leves. Usar o cache ReadWrite com um aplicativo que não manipule a persistência dos dados necessários pode levar à perda de dados no caso de falha da VM.

- *Nenhum* (desabilitado): com essa opção, você pode ignorar o cache. Todos os dados são transferidos para o disco e persistidos para o Armazenamento do Azure. Esse método fornece a mais alta taxa de E/S para cargas de trabalho intensivas de E/S. Você também precisa levar o "custo de transação" em consideração.

**Recomendações**

Para maximizar a taxa de transferência, recomendamos que você inicie com **nenhum** para o cache de host. Para o Armazenamento Premium, lembre-se de que você deve desabilitar as "barreiras" ao montar o sistema de arquivos com as opções **ReadOnly** ou **Nenhum**. Atualize o arquivo /etc/fstab com o UUID para os discos.

![Captura de tela da página de disco gerenciado que mostra as opções ReadOnly e None.](./media/oracle-design/premium_disk02.png)

- Para discos do sistema operacional, use o cache de **Leitura/Gravação** padrão.
- Para SYSTEM, TEMP e UNDO, use **Nenhum** para o cache.
- Para DATA, use **Nenhum** para armazenar em cache. Porém, se o banco de dados for somente leitura ou leitura intensiva, use o cache de **Somente leitura**.

Após a gravação da configuração de disco de dados, não será possível alterar a configuração de cache de host, a menos que você desmonte a unidade no nível do SO e remonte-a após a alteração.

## <a name="security"></a>Segurança

Depois de instalar e configurar seu ambiente do Azure, a próxima etapa será proteger sua rede. Veja algumas recomendações:

- *Política de NSG*: o NSG pode ser definido por uma sub-rede ou NIC. É mais simples controlar o acesso no nível de sub-rede, tanto para segurança quanto para roteamento de força para coisas como firewalls de aplicativo.

- *Jumpbox*: para um acesso mais seguro, os administradores não devem conectar diretamente ao serviço de aplicativo ou ao banco de dados. Um jumpbox é usado como um intermediário entre o computador do administrador e os recursos do Azure.
![Captura de tela da página de topologia do Jumpbox](./media/oracle-design/jumpbox.png)

    O computador do administrador deve oferecer somente acesso restrito por IP ao jumpbox. O jumpbox deve ter acesso ao aplicativo e ao banco de dados.

- *Rede privada* (sub-redes): recomendamos que o serviço de aplicativo e o banco de dados estejam em sub-redes separadas, assim é possível definir um controle melhor por política de NSG.


## <a name="additional-reading"></a>Leituras adicionais

- [Configurar o Oracle ASM](configure-oracle-asm.md)
- [Configurar o Oracle Data Guard](configure-oracle-dataguard.md)
- [Configurar o Oracle Golden Gate](configure-oracle-golden-gate.md)
- [Backup e recuperação do Oracle](./oracle-overview.md)

## <a name="next-steps"></a>Próximas etapas

- [Tutorial: criar VMs altamente disponíveis](../../linux/create-cli-complete.md)
- [Explorar exemplos da CLI do Azure de implantação de VM](../../linux/cli-samples.md)