---
title: 'Tutorial: Criar um balanceador de carga entre regiões usando o portal do Azure'
titleSuffix: Azure Load Balancer
description: Comece a usar este tutorial implantando um Azure Load Balancer entre regiões com o portal do Azure.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 11/24/2020
ms.openlocfilehash: fe335b00df23ff132ad61b27efd733d195ee3bfb
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030913"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-the-azure-portal"></a>Tutorial: Criar um Azure Load Balancer entre regiões usando o portal do Azure

Um balanceador de carga entre regiões garante que um serviço esteja disponível globalmente em várias regiões do Azure. Se uma região falhar, o tráfego será roteado para o balanceador de carga regional íntegro mais próximo.  

Neste tutorial, você aprenderá como:

> [!div class="checklist"]
> * Criar um balanceador de carga entre regiões.
> * Criar um pool de back-end que contém dois balanceadores de carga regionais.
> * Crie uma regra de balanceador de carga.
> * Testar o balanceador de carga.

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

- Uma assinatura do Azure.
- Dois Azure Load Balancers de SKU **Standard** com pools de back-end implantados em duas regiões diferentes do Azure.
    - Para obter informações sobre como criar um Standard Load Balancer regional e máquinas virtuais para pools de back-end, confira [Guia de Início Rápido: Criar um balanceador de carga público para balancear cargas de VMs usando o portal do Azure](quickstart-load-balancer-standard-public-portal.md).
        - Acrescente o nome dos balanceadores de carga, das máquinas virtuais e de outros recursos em cada região com um **-R1** e **-R2**. 

## <a name="sign-in-to-azure-portal"></a>Entre no portal do Azure

> [!IMPORTANT]
> Atualmente, o balanceador de carga entre regiões está em versão prévia e oculto na versão prévia do portal.  Entre em **https://preview.portal.azure.com/?feature.globallb=true** para ver e implantar o recurso.

## <a name="create-cross-region-load-balancer"></a>Criar um balanceador de carga entre regiões

Nesta seção, você criará um balanceador de carga entre regiões e um endereço IP público.

1. No canto superior esquerdo da tela, selecione **Criar um recurso > Rede > Balanceador de Carga** ou pesquise **Balanceador de Carga** na caixa de pesquisa.

2. Na guia **Informações Básicas** da página **Criar balanceador de carga**, insira ou selecione as seguintes informações: 

    | Configuração                 | Valor                                              |
    | ---                     | ---                                                |
    | Subscription               | Selecione sua assinatura.    |    
    | Resource group         | Selecione **Criar** e insira **CreateCRLBTutorial-rg** na caixa de texto.|
    | Name                   | Insira **myLoadBalancer-CR**                                   |
    | Região         | Selecione **Oeste dos EUA**.                                        |
    | Type          | Selecione **Público**.                                        |
    | SKU           | Selecione **Standard** |
    | Camada           | Selecione **Global** |
    | Endereço IP público | Selecione **Criar novo**.|
    | Nome do endereço IP público | Digite **myPublicIP-CR** na caixa de texto.|
    | Preferência de roteamento| Selecione **Rede da Microsoft** |

    > [!NOTE]
    > O balanceador de carga entre regiões só pode ser implantado nas seguintes regiões de residência: **Leste dos EUA 2, Oeste dos EUA, Oeste da Europa, Sudeste da Ásia, EUA Central, Norte da Europa e Leste da Ásia**. Para obter mais informações, confira **https://aka.ms/homeregionforglb**.


3. Aceite os padrões para as demais configurações e selecione **Examinar + criar**.

4. Na guia **Examinar + criar**, selecione **Criar**.   

    :::image type="content" source="./media/tutorial-cross-region-portal/create-cross-region.png" alt-text="Criar um balanceador de carga entre regiões" border="true":::

## <a name="create-backend-pool"></a>Criar pool de back-end

Nesta seção, você adicionará dois balanceadores de carga Standard regionais ao pool de back-end do balanceador de carga entre regiões.

> [!IMPORTANT]
> Para concluir essas etapas, verifique se dois balanceadores de carga regionais com pools de back-end foram implantados na sua assinatura.  Para obter mais informações, confira **[Guia de Início Rápido: Criar um balanceador de carga público para balancear cargas de VMs usando o portal do Azure](quickstart-load-balancer-standard-public-portal.md)** .

Crie o pool de endereços de back-end **myBackendPool-CR** para incluir os balanceadores de carga Standard regionais.

1. Escolha **Todos os serviços** no menu à esquerda, **Todos os recursos** e **myLoadBalancer-CR** na lista de recursos.

2. Em **Configurações**, selecione **Pools de back-end** e, em seguida, **Adicionar**.

3. Na página **Adicionar um pool de back-end**, como o nome, digite **myBackendPool-CR**.

4. Selecione **Adicionar**.

4. Selecione **myBackendPool-CR**.

5. Em **Balanceadores de carga**, escolha a caixa suspensa em **Balanceador de carga**.

5. Escolha **myLoadBalancer-R1** ou o nome do balanceador de carga na região 1.

6. Selecione a caixa suspensa em **Configuração de IP de front-end**. Escolha **LoadBalancerFrontEnd**.

7. Repita as etapas 4 a 6 para adicionar **myLoadBalancer-R2**.

8. Selecione **Adicionar**.

    :::image type="content" source="./media/tutorial-cross-region-portal/add-to-backendpool.png" alt-text="Adicionar balanceadores de carga regionais ao pool de back-end" border="true":::

## <a name="create-a-health-probe"></a>Criar uma investigação de integridade

Nesta seção, você criará uma investigação de integridade para criar a regra de balanceamento de carga:

* Chamado **myHealthProbe**.
* Protocolo **TCP**.
* Intervalo de **5** segundos.
* Limite não íntegro de **duas** falhas.

1. Escolha **Todos os serviços** no menu à esquerda, **Todos os recursos** e **myLoadBalancer-CR** na lista de recursos.

2. Em **Configurações**, selecione **Investigações de integridade**.

3. Use estes valores para configurar a investigação de integridade:

    | Configuração | Valor |
    | ------- | ----- |
    | Nome | Insira **myHealthProbe**. |
    | Protocolo | selecione **TCP**. |
    | Porta | Insira **80**. |
    | Intervalo | Insira **5**. |
    | Limite não íntegro | Insira **2**. |

4. Selecione **OK**.

    > [!NOTE]
    > O balanceador de carga entre regiões tem uma investigação de integridade interna. Essa investigação é um espaço reservado para que a criação da regra de balanceamento de carga funcione.  Para obter mais informações, confira **[Limitações do balanceador de carga entre regiões](cross-region-overview.md#limitations)** .

## <a name="create-a-load-balancer-rule"></a>Criar uma regra de balanceador de carga

Nesta seção, você criará uma regra de balanceador de carga:

* Chamada **myHTTPRule**.
* No front-end chamado **LoadBalancerFrontEnd**.
* Escutando a **Porta 80**.
* Direciona o tráfego com balanceamento de carga para o back-end chamado **myBackendPool-CR** na **Porta 80**.

    > [!NOTE]
    > A porta de front-end precisa corresponder à porta de back-end e à porta de front-end dos balanceadores de carga regionais no pool de back-end.

1. Escolha **Todos os serviços** no menu à esquerda, **Todos os recursos** e **myLoadBalancer-CR** na lista de recursos.

2. Em **Configurações**, selecione **Regras de balanceamento de carga** e **Adicionar**.

3. Use estes valores para configurar a regra de balanceamento de carga:
    
    | Configuração | Valor |
    | ------- | ----- |
    | Nome | Insira **myHTTPRule**. |
    | Versão IP | Selecione **IPv4** |
    | Endereço IP de front-end | Selecione **LoadBalancerFrontEnd** |
    | Protocolo | selecione **TCP**. |
    | Porta | Insira **80**.|
    | Porta de back-end | Insira **80**. |
    | Pool de back-end | Selecione **myBackendPool**.|
    | Investigação de integridade | Selecione **myHealthProbe**. |
    | Tempo limite de ociosidade (minutos) | Mova o controle deslizante para **15**. |
    | Redefinição de TCP | Selecione **Habilitado**. |

4. Deixe o restante dos padrões e selecione **OK**.

    :::image type="content" source="./media/tutorial-cross-region-portal/create-lb-rule.png" alt-text="Criar uma regra para o balanceador de carga" border="true":::

## <a name="test-the-load-balancer"></a>Testar o balanceador de carga

Nesta seção, você testará o balanceador de carga entre regiões. Você se conectará ao endereço IP público em um navegador da Web.  Você interromperá as máquinas virtuais em um dos pools de back-end do balanceador de carga regional e observará o failover.

1. Encontre o endereço IP público para o balanceador de carga na tela **Visão Geral**. Escolha **Todos os serviços** no menu à esquerda, **Todos os recursos** e **myPublicIP-CR**.

2. Copie o endereço IP público e cole-o na barra de endereços do seu navegador. A página padrão do servidor Web do IIS é exibida no navegador.

    :::image type="content" source="./media/tutorial-cross-region-portal/test-cr-lb-1.png" alt-text="Testar o balanceador de carga" border="true":::

3. Interrompa as máquinas virtuais no pool de back-end de um dos balanceadores de carga regionais.

4. Atualize o navegador da Web e observe o failover da conexão no outro balanceador de carga regional.

    :::image type="content" source="./media/tutorial-cross-region-portal/test-cr-lb-2.png" alt-text="Testar o balanceador de carga após o failover" border="true":::

## <a name="clean-up-resources"></a>Limpar os recursos

Quando não forem mais necessários, exclua o grupo de recursos, o balanceador de carga e todos os recursos relacionados. 

Para fazer isso, selecione o grupo de recursos **CreateCRLBTutorial-rg** que contém os recursos e escolha **Excluir**.

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você:

* Criou um balanceador de carga entre regiões.
* Adicionou balanceadores de carga regionais ao pool de back-end do balanceador de carga entre regiões.
* Criou uma regra de balanceamento de carga.
* Testou o balanceador de carga.

Para obter mais informações sobre o balanceador de carga entre regiões, confira [Balanceador de carga entre regiões (versão prévia)](cross-region-overview.md).


Prossiga para o próximo artigo para saber como:
> [!div class="nextstepaction"]
> [VMs com balanceamento de carga entre zonas de disponibilidade](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
