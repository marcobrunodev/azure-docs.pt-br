---
title: Analisando o vídeo ao vivo sem qualquer gravação-Azure
description: Um grafo de mídia pode ser usado apenas para extrair análises de um fluxo de vídeo ao vivo, sem precisar gravá-la na borda ou na nuvem. Este artigo discute esse conceito.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 5dda18b68cb19d29623f2120fe07d7cc617f0c2f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90893028"
---
# <a name="analyzing-live-video-without-any-recording"></a>Analisando o vídeo ao vivo sem nenhuma gravação

## <a name="suggested-pre-reading"></a>Pré-leitura sugerida 

* [Conceito de grafo de mídia](media-graph-concept.md)
* [Gravação de vídeo baseada em eventos](event-based-video-recording-concept.md)

## <a name="overview"></a>Visão geral  

Você pode usar um grafo de mídia para analisar o vídeo ao vivo, sem registrar nenhuma parte do vídeo em um arquivo ou um ativo. Os gráficos de mídia mostrados abaixo são semelhantes aos do artigo sobre a [gravação de vídeo baseada em evento](event-based-video-recording-concept.md), mas sem um nó do coletor de ativos ou nó de coletor de arquivos.

### <a name="motion-detection"></a>Detecção de movimento

O grafo de mídia mostrado abaixo consiste em um nó de [origem RTSP](media-graph-concept.md#rtsp-source) , um nó de [processador de detecção de movimento](media-graph-concept.md#motion-detection-processor) e um nó de coletor de mensagens do [Hub IOT](media-graph-concept.md#iot-hub-message-sink) . A representação JSON da topologia do grafo desse grafo de mídia pode ser encontrada [aqui](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/motion-detection/topology.json). Esse grafo permite que você detecte o movimento na transmissão de vídeo ao vivo de entrada e retransmita os eventos de movimento para outros aplicativos e serviços por meio do nó de coletor de mensagens do Hub IoT. Os aplicativos ou serviços externos podem disparar um alerta ou enviar uma notificação para o pessoal apropriado.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Análise de Vídeo ao Vivo com base na detecção de movimento":::

### <a name="analyzing-video-using-a-custom-vision-model"></a>Analisando vídeo usando um modelo de visão personalizada 

O grafo de mídia mostrado abaixo permite que você analise um fluxo de vídeo ao vivo usando um modelo de visão personalizada empacotado em um módulo separado. A representação JSON da topologia do grafo desse grafo de mídia pode ser encontrada [aqui](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/httpExtension/topology.json). Você pode ver alguns exemplos [aqui](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis) sobre o encapsulamento de modelos em IOT Edge módulos que são executados como um serviço de inferência.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detected-frames.svg" alt-text="Análise de Vídeo ao Vivo com base na detecção de movimento":::

Nesse grafo de mídia, o nó do processador de filtro de taxa de quadros reduz a taxa de quadros do fluxo de vídeo ao vivo de entrada antes de enviá-lo para um nó de [processador de extensão http](media-graph-concept.md#http-extension-processor) , que envia quadros de imagem (em formatos JPEG, BMP ou png) para um serviço de inferência externo em relação ao REST. Os resultados do serviço de inferência externa são recuperados pelo nó de extensão HTTP e retransmitidos para o Hub de IoT Edge por meio do nó de coletor de mensagens do Hub IoT. Esse tipo de grafo de mídia pode ser usado para criar soluções para uma variedade de cenários, como entender a distribuição de série temporal de veículos em uma interseção, entender o padrão de tráfego do consumidor em uma loja de varejo e assim por diante.

Um aprimoramento deste exemplo é usar um processador de detector de movimento à frente do nó do processador de filtro de taxa de quadros. Isso reduzirá a carga no serviço de inferência, já que ela é usada somente quando há atividade de movimento no vídeo.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/custom-model.svg" alt-text="Análise de Vídeo ao Vivo com base na detecção de movimento":::

## <a name="next-steps"></a>Próximas etapas

[Gravação de vídeo contínua](continuous-video-recording-concept.md)
