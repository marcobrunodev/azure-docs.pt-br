---
title: Tutorial – Criar e gerenciar um trabalho do Stream Analytics usando o portal do Azure
description: Este tutorial fornece uma demonstração ponta a ponta sobre como usar o Azure Stream Analytics para analisar chamadas fraudulentas em um fluxo de chamadas telefônicas.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc
ms.date: 10/30/2020
ms.openlocfilehash: 70acc696f1cb366d25299f616744e52491a54471
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024170"
---
# <a name="tutorial-analyze-phone-call-data-with-stream-analytics-and-visualize-results-in-power-bi-dashboard"></a>Tutorial: Analisar os dados de uma chamada telefônica com o Stream Analytics e visualizar os resultados em um dashboard do Power BI

Este tutorial ensina como analisar dados de chamadas telefônicas usando o Azure Stream Analytics. Os dados de chamadas telefônicas, gerados por um aplicativo cliente, contêm algumas chamadas fraudulentas que serão filtradas pelo trabalho do Stream Analytics.

Neste tutorial, você aprenderá como:

> [!div class="checklist"]
> * Gerar dados de chamadas telefônicas de exemplo e enviá-los aos Hubs de Eventos do Azure
> * Criar um trabalho de Stream Analytics
> * Configurar entrada e saída do trabalho
> * Definir uma consulta para filtrar chamadas fraudulentas
> * Testar e iniciar o trabalho
> * Visualizar os resultados no Power BI

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar, execute as seguintes ações:

* Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/).
* Entre no [portal do Azure](https://portal.azure.com/).
* Faça o download do aplicativo gerador de evento de chamada telefônica [TelcoGenerator.zip](https://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip) do Centro de Download da Microsoft ou obtenha o código-fonte no [GitHub](https://aka.ms/azure-stream-analytics-telcogenerator).
* Você precisará de uma conta do Power BI.

## <a name="create-an-azure-event-hub"></a>Criar um Hub de Eventos do Azure

Para que o Stream Analytics possa analisar o fluxo de dados de chamadas fraudulentas, os dados precisam ser enviados ao Azure. Neste tutorial, você enviará dados ao Azure usando o [Hubs de Eventos do Azure](../event-hubs/event-hubs-about.md).

Use as seguintes etapas para criar um Hub de Eventos e enviar dados de chamada para ele:

1. Entre no [portal do Azure](https://portal.azure.com/).
2. Selecione **Criar um recurso** > **Internet das Coisas** > **Hubs de Eventos**.

   ![Criar um Hub de Eventos do Azure no portal](media/stream-analytics-manage-job/find-event-hub-resource.png)
3. Preencha o painel **Criar Namespace** com os seguintes valores:

   |**Configuração**  |**Valor sugerido** |**Descrição**  |
   |---------|---------|---------|
   |Nome     | asaTutorialEventHub        |  Um nome exclusivo para identificar o namespace de hub de eventos.       |
   |Subscription     |   \<Your subscription\>      |   Selecione uma assinatura do Azure em que deseja criar o hub de eventos.      |
   |Resource group     |   MyASADemoRG      |  Selecione **Criar Novo** e insira um novo nome de grupo de recursos para a conta.       |
   |Local     |   Oeste dos EUA 2      |    Local onde o namespace do hub de eventos pode ser implantado.     |

4. Use opções padrão nas configurações restantes e selecione **Examinar + criar**. Depois selecione **Criar** para iniciar a implantação.

   ![Criar um namespace do hub de eventos no portal do Azure](media/stream-analytics-manage-job/create-event-hub-namespace.png)

5. Quando o namespace terminar a implantação, acesse **Todos os recursos** e localize *asaTutorialEventHub* na lista de recursos do Azure. Clique em *asaTutorialEventHub* para abri-lo.

6. Depois selecione **+ Hub de Eventos** e insira um **Nome** para o Hub de Eventos. Defina a **Contagem de Partições** como *2*.  Use as opções padrão nas configurações restantes e clique em **Criar**. Aguarde até que a implantação tenha êxito.

   ![Configuração do Hub de Eventos no portal do Azure](media/stream-analytics-manage-job/create-event-hub-portal.png)

### <a name="grant-access-to-the-event-hub-and-get-a-connection-string"></a>Conceder acesso para o hub de eventos e obter uma cadeia de caracteres de conexão

Antes que um processo possa enviar dados aos Hubs de Eventos do Azure, o hub de eventos deve ter uma política que permita o devido acesso. A política de acesso produz uma cadeia de conexão que inclui informações de autorização.

1. Acesse o hub de eventos *MyEventHub* criado na etapa anterior. Escolha **Políticas de acesso compartilhado** em **Configurações** e, em seguida, **+ Adicionar**.

2. Nomeie a política **MyPolicy** e verifique se a opção **Gerenciar** está marcada. Em seguida, selecione **Criar**.

   ![Criar política de acesso compartilhado do hub de eventos](media/stream-analytics-manage-job/create-event-hub-access-policy.png)

3. Após criar a política, clique no nome dela para abri-la. Localize a opção **Cadeia de conexão – chave primária**. Clique no botão **Copiar** ao lado da cadeia de conexão.

   ![Salvar a cadeia de conexão da política de acesso compartilhado](media/stream-analytics-manage-job/save-connection-string.png)

4. Cole a cadeia de conexão em um editor de texto. Você precisará dessa cadeia de conexão na próxima seção.

   A cadeia de conexão tem esta aparência:

   `Endpoint=sb://<Your event hub namespace>.servicebus.windows.net/;SharedAccessKeyName=<Your shared access policy name>;SharedAccessKey=<generated key>;EntityPath=<Your event hub name>`

   A cadeia de conexão contém vários pares chave-valor separados por ponto e vírgula: **Endpoint**, **SharedAccessKeyName**, **SharedAccessKey** e **EntityPath**.

## <a name="start-the-event-generator-application"></a>Iniciar o aplicativo gerador de evento

Antes de iniciar o aplicativo TelcoGenerator, configure-o para enviar dados para os Hubs de Eventos do Azure criados anteriormente.

1. Extraia o conteúdo do arquivo [TelcoGenerator.zip](https://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip).
2. Abra o arquivo `TelcoGenerator\TelcoGenerator\telcodatagen.exe.config` em um editor de texto de sua escolha (há mais de um arquivo de configuração, portanto, lembre-se de abrir o arquivo correto).

3. Atualize o elemento `<appSettings>` no arquivo de configuração com os seguintes detalhes:

   * Defina o valor da chave *EventHubName* como o valor de EntityPath na cadeia de conexão.
   * Defina o valor da chave *Microsoft.ServiceBus.ConnectionString* para a cadeia de conexão sem o valor de EntityPath (não se esqueça de remover o ponto e vírgula que o precede).

4. Salve o arquivo.
5. Em seguida, abra uma janela de comando e altere para a pasta onde o aplicativo TelcoGenerator foi descompactado. Em seguida, digite o seguinte comando:

   ```cmd
   telcodatagen.exe 1000 0.2 2
   ```

   Esse comando usa os seguintes parâmetros:
   * Número de registros de dados de chamadas por hora.
   * Porcentagem de probabilidade de fraude, que é a frequência com que o aplicativo deve simular uma chamada fraudulenta. O valor 0,2 significa que cerca de 20% dos registros de chamada parecerão ser fraudulentos.
   * Duração em horas, que é o número de horas em que o aplicativo deve ser executado. Também é possível interromper o aplicativo a qualquer momento encerrando o processo (**Ctrl+C**) na linha de comando.

   Depois de alguns segundos, o aplicativo é iniciado exibindo registros de chamada telefônica na tela, enquanto envia para o hub de eventos. Os dados de chamadas telefônicas contêm os seguintes campos:

   |**Registro**  |**Definição**  |
   |---------|---------|
   |CallrecTime    |  Carimbo de data/hora para a hora de início da chamada.       |
   |SwitchNum     |  Chave do telefone usada para se conectar à chamada. Neste exemplo, as opções são cadeias de caracteres que representam o país/região de origem (Estados Unidos, China, Reino Unido, Alemanha ou Austrália).       |
   |CallingNum     |  Número de telefone do autor da chamada.       |
   |CallingIMSI     |  A Identidade do Assinante Móvel Internacional (IMSI). É um identificador exclusivo do autor da chamada.       |
   |CalledNum     |   O número de telefone do destinatário da chamada.      |
   |CalledIMSI     |  Identidade do Assinante Móvel Internacional (IMSI). É um identificador exclusivo do destinatário da chamada.       |

## <a name="create-a-stream-analytics-job"></a>Criar um trabalho de Stream Analytics

Agora que você tem um fluxo de eventos de chamada, pode criar um trabalho do Stream Analytics que lê dados do hub de eventos.

1. Para criar um trabalho do Stream Analytics, navegue até o [portal do Azure](https://portal.azure.com/).

2. Clique em **Criar um recurso** e pesquise um **trabalho do Stream Analytics**. Selecione o bloco **Trabalho do Stream Analytics** e selecione **Criar**.

3. Preencha o formulário **Novo trabalho do Stream Analytics** com os seguintes valores:

   |**Configuração**  |**Valor sugerido**  |**Descrição**  |
   |---------|---------|---------|
   |Nome do trabalho     |  ASATutorial       |   Um nome exclusivo para identificar o namespace de hub de eventos.      |
   |Subscription    |  \<Your subscription\>   |   Selecione uma assinatura do Azure em que deseja criar o trabalho.       |
   |Resource group   |   MyASADemoRG      |   Selecione **Usar existente** e insira um novo nome de grupo de recursos para sua conta.      |
   |Local   |    Oeste dos EUA 2     |      Local onde o trabalho pode ser implantado. É recomendável colocar o trabalho e o hub de eventos na mesma região para melhor desempenho e para que não seja necessário pagar para transferir dados entre regiões.      |
   |Ambiente de hospedagem    | Nuvem        |     Os trabalhos do Stream Analytics podem ser implantados na nuvem ou na borda. O Cloud permite que você implante no Azure Cloud e o Edge permite que você implante em um dispositivo IoT Edge.    |
   |Unidades de transmissão     |    1       |      As unidades de streaming representam os recursos de computação necessários para executar um trabalho. Por padrão, esse valor é definido como 1. Para saber mais sobre como dimensionar unidades de streaming, confira o artigo [Entendendo e ajustando as unidades de streaming](stream-analytics-streaming-unit-consumption.md).      |

4. Use as opções padrão nas configurações restantes, selecione **Criar** e aguarde até que a implantação seja concluída com êxito.

   ![Criar um trabalho do Azure Stream Analytics](media/stream-analytics-manage-job/create-stream-analytics-job.png)

## <a name="configure-job-input"></a>Configurar entrada de trabalho

A próxima etapa é definir uma fonte de entrada para o trabalho ler os dados usando o hub de eventos criado na seção anterior.

1. No portal do Azure, abra a página **Todos os recursos** e localize o trabalho *ASATutorial* do Stream Analytics.

2. Na seção **Topologia do Trabalho** do trabalho do Stream Analytics, clique em **Entradas**.

3. Marque **+ Adicionar entrada de fluxo** e **Hub de eventos**. Preencha o formulário de entrada com os seguintes valores:

   |**Configuração**  |**Valor sugerido**  |**Descrição**  |
   |---------|---------|---------|
   |Alias de entrada     |  CallStream       |  Forneça um nome amigável para identificar a entrada. O alias de entrada pode conter somente caracteres alfanuméricos, hifens e sublinhados e deve ter entre 3 e 63 caracteres.       |
   |Subscription    |   \<Your subscription\>      |   Selecione a assinatura do Azure em que você criou o hub de eventos. O hub de eventos pode estar na mesma assinatura ou em uma diferente da do trabalho do Stream Analytics.       |
   |Namespace do Hub de Eventos    |  asaTutorialEventHub       |  Selecione o namespace do hub de eventos criado na seção anterior. Todos os namespaces de hub de eventos disponíveis na sua assinatura atual aparecem na lista suspensa.       |
   |Nome do Hub de Eventos    |   MyEventHub      |  Selecione o hub de eventos criado na seção anterior. Todos os hubs de eventos disponíveis na sua assinatura atual aparecem na lista suspensa.       |
   |Nome da política do Hub de Eventos   |  MyPolicy       |  Selecione a política de acesso compartilhado de hub de eventos criada na seção anterior. Todas as políticas de hubs de eventos disponíveis na sua assinatura atual aparecem na lista suspensa.       |

4. Use as opções padrão nas configurações restantes e escolha **Salvar**.

   ![Configurar a entrada do Azure Stream Analytics](media/stream-analytics-manage-job/configure-stream-analytics-input.png)

## <a name="configure-job-output"></a>Configurar saída de trabalho

A última etapa é definir um coletor de saída no qual o trabalho poderá gravar os dados transformados. Neste tutorial, você gera e visualiza dados com o Power BI.

1. No portal do Azure, abra **Todos os recursos** e selecione o trabalho *ASATutorial* do Stream Analytics.

2. Na seção **Topologia do Trabalho** do trabalho do Stream Analytics, selecione a opção **Saídas**.

3. Escolha **+ Adicionar** > **Power BI**. Depois clique em **Autorizar** e siga os prompts para autenticar o Power BI.

O :::image type="content" source="media/stream-analytics-manage-job/authorize-power-bi.png" alt-text="botão Autorizar do Power BI":::

4. Preencha o formulário de saída com os seguintes detalhes e clique em **Salvar**:

   |**Configuração**  |**Valor sugerido**  |
   |---------|---------|
   |Alias de saída  |  MyPBIoutput  |
   |Agrupar o workspace| Meu workspace |
   |Nome do conjunto de dados  |   ASAdataset  |
   |Nome da tabela |  ASATable  |
   | Modo de autenticação | Token de usuário |

   ![Configurar a saída do Stream Analytics](media/stream-analytics-manage-job/configure-stream-analytics-output.png)

   Este tutorial usará o modo de autenticação *Token de usuário*. Para usar a identidade gerenciada, confira [Usar a identidade gerenciada para autenticar o trabalho do Azure Stream Analytics para o Power BI](powerbi-output-managed-identity.md).

## <a name="define-a-query-to-analyze-input-data"></a>Definir uma consulta para analisar os dados de entrada

A próxima etapa é criar uma transformação que analisa os dados em tempo real. Defina a consulta de transformação usando a [Linguagem de consulta do Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). A consulta usada neste tutorial detecta chamadas fraudulentas nos dados do telefone.

Neste exemplo, as chamadas fraudulentas são feitas pelo mesmo usuário em cinco segundos, mas em lugares diferentes. Por exemplo, o mesmo usuário não pode legitimamente fazer uma chamada da Austrália e dos Estados Unidos ao mesmo tempo. Para definir a consulta de transformação para o trabalho do Stream Analytics:

1. No portal do Azure, abra o painel **Todos os recursos** e navegue até o trabalho **ASATutorial** do Stream Analytics criado anteriormente.

2. Na seção **Topologia do Trabalho** do trabalho do Stream Analytics, selecione a opção **Consulta**. A janela de consulta listará entradas e saídas configuradas para o trabalho e permitirá criar uma consulta a fim de transformar o fluxo de entrada.

3. Substitua a consulta existente no editor pela seguinte consulta, que realiza uma autojunção em um intervalo de cinco segundos de dados de chamadas:

   ```sql
   SELECT System.Timestamp AS WindowEnd, COUNT(*) AS FraudulentCalls
   INTO "MyPBIoutput"
   FROM "CallStream" CS1 TIMESTAMP BY CallRecTime
   JOIN "CallStream" CS2 TIMESTAMP BY CallRecTime
   ON CS1.CallingIMSI = CS2.CallingIMSI
   AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
   WHERE CS1.SwitchNum != CS2.SwitchNum
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

   Para verificar chamadas fraudulentas, você pode fazer a autojunção dos dados de streaming com base no valor `CallRecTime`. Em seguida, você pode procurar registros de chamada em que o valor `CallingIMSI` (o número de origem) é o mesmo, mas o valor `SwitchNum` (país/região de origem) é diferente. Quando você usa uma operação JOIN com os dados de streaming, a junção deve fornecer alguns limites sobre a distância de tempo que as linhas correspondentes podem ter umas das outras. Como o fluxo de dados é infinito, os limites de tempo para a relação são especificados dentro da cláusula **ON** da junção, usando a função [DATEDIFF](/stream-analytics-query/datediff-azure-stream-analytics).

   Essa consulta é como uma junção SQL normal, exceto pela função **DATEDIFF**. A função **DATEDIFF** usada nessa consulta é específica do Stream Analytics e ela deverá ser exibida dentro na cláusula `ON...BETWEEN`.

4. **Salve** a consulta.

   ![Definir a consulta do Stream Analytics no portal](media/stream-analytics-manage-job/define-stream-analytics-query.png)

## <a name="test-your-query"></a>Testar a consulta

É possível testar uma consulta no editor de consultas. Execute as seguintes etapas para testar a consulta:

1. Verifique se o aplicativo TelcoGenerator está em execução e gerando os registros de chamada.

2. Escolha **Testar** para testar a consulta. Você deve ver o seguintes resultados:

   ![Saída do teste de consulta do Stream Analytics](media/stream-analytics-manage-job/sample-test-output-restuls.png)

## <a name="start-the-job-and-visualize-output"></a>Iniciar o trabalho e visualizar a saída

1. Para iniciar o trabalho, acesse a **Visão Geral** do trabalho e clique em **Iniciar**.

2. Selecione **Agora** para a hora de início da saída do trabalho e selecione **Iniciar**. Você pode exibir o status do trabalho na barra de notificação.

3. Depois que o trabalho for concluído com êxito, navegue até o [Power BI](https://powerbi.com/) e entre com sua conta corporativa ou de estudante. Se a consulta do trabalho do Stream Analytics estiver gerando resultados, o conjunto de dados *ASAdataset* criado estará presente na guia **Conjuntos de Dados**.

4. No workspace do Power BI, escolha **+ Criar** para criar um painel novo chamado *Chamadas Fraudulentas*.

5. Na parte superior da janela, clique em **Editar** e **Adicionar bloco**. Em seguida, escolha **Fluxo de Dados Personalizado** e **Avançar**. Escolha o **ASAdataset** em **Seus Conjuntos de Dados**. Escolha **Cartão** na lista suspensa **Tipo de visualização** e adicione **chamadas fraudulentas** a **Campos**. Escolha **Avançar** para inserir um nome para o bloco e escolha **Aplicar** para criar o bloco.

   ![Criar blocos de dashboard do Power BI](media/stream-analytics-manage-job/create-power-bi-dashboard-tiles.png)

6. Siga a etapa 5 novamente, com as seguintes opções:
   * Quando você chegar em Tipo de Visualização, selecione Gráfico de linhas.
   * Adicionar um eixo e selecione **windowend**.
   * Adicione um valor e selecione **fraudulentcalls**.
   * Para **Janela de tempo para exibir**, selecione os últimos 10 minutos.

7. Seu painel deverá se parecer com o exemplo a seguir depois que ambos os blocos forem adicionados. Observe que, se você estiver executando o aplicativo de remetente do hub de eventos e o Stream Analytics, o dashboard do Power BI será atualizado periodicamente de acordo com a chegada de novos dados.

   ![Exibir resultados no dashboard do Power BI](media/stream-analytics-manage-job/power-bi-results-dashboard.png)

## <a name="embedding-your-power-bi-dashboard-in-a-web-application"></a>Como inserir seu dashboard do Power BI em um aplicativo Web

Nesta parte do tutorial, você usará um aplicativo Web [ASP.NET](https://asp.net/) de exemplo criado pela equipe do Power BI para inserir seu dashboard. Para saber mais sobre a inserção de painéis, confira o artigo [Inserindo com o Power BI](/power-bi/developer/embedding).

Para configurar o aplicativo, acesse o repositório [PowerBI-Developer-Samples](https://github.com/Microsoft/PowerBI-Developer-Samples) do GitHub e siga as instruções na seção **Usuário Tem Dados** (use as URLs de redirecionamento e de página inicial da subseção **integrate-web-app**). Como estamos usando painel de exemplo, use o código de exemplo **integrate-web-app** localizado no [repositório GitHub](https://github.com/microsoft/PowerBI-Developer-Samples/tree/master/.NET%20Framework/Embed%20for%20your%20organization/).
Quando o aplicativo estiver em execução no seu navegador, siga estas etapas para inserir o painel criado anteriormente na página:

1. Selecione **Entrar no Power BI**, que concede ao aplicativo acesso aos dashboards da sua conta do Power BI.

2. Selecione o botão **Obter Painéis**, que exibe painéis da sua conta em uma tabela. Localize o nome do painel criado anteriormente, **powerbi-embedded-dashboard**, e copie a **EmbedUrl** correspondente.

3. Por fim, cole a **EmbedUrl** no campo de texto correspondente e selecione **Inserir Painel**. Agora você pode exibir o mesmo painel inserido em um aplicativo Web.

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você criou um trabalho simples do Stream Analytics, analisou os dados de entrada e apresentou resultados em um painel do Power BI. Para saber mais sobre trabalhos do Stream Analytics, prossiga para o seguinte tutorial:

> [!div class="nextstepaction"]
> [Executar o Azure Functions com trabalhos do Stream Analytics](stream-analytics-with-azure-functions.md)
