---
title: Copiar ou fazer backup de trabalhos de Azure Stream Analytics
description: Este artigo descreve como copiar ou fazer backup de um trabalho de Azure Stream Analytics.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/11/2019
ms.openlocfilehash: ba63358347cf9722d2cafa35598b9b3b37f49dc3
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93129449"
---
# <a name="copy-or-back-up-azure-stream-analytics-jobs"></a>Copiar ou fazer backup de trabalhos de Azure Stream Analytics

Você pode copiar ou fazer backup de seus trabalhos de Azure Stream Analytics implantados usando o Visual Studio Code ou o Visual Studio. Copiar um trabalho para outra região não copia a hora da última saída. Portanto, você não pode usar a opção [**quando a última parada**](./start-job.md#start-options) ao iniciar o trabalho copiado.

## <a name="before-you-begin"></a>Antes de começar
* Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/).

* Entre no [portal do Azure](https://portal.azure.com/).

* Instale [a extensão de Azure Stream Analytics para Visual Studio Code](quick-create-visual-studio-code.md#install-the-azure-stream-analytics-tools-extension) ou [Azure Stream Analytics Tools para Visual Studio](quick-create-visual-studio-code.md#install-the-azure-stream-analytics-tools-extension).  

## <a name="visual-studio-code"></a>Visual Studio Code

1. Clique no ícone **do Azure** na barra de atividade do Visual Studio Code e, em seguida, expanda **Stream Analytics** nó. Seus trabalhos devem aparecer sob suas assinaturas.

   ![Abrir Stream Analytics Explorer](./media/vscode-explore-jobs/open-explorer.png)

2. Para exportar um trabalho para um projeto local, localize o trabalho que você deseja exportar no **Stream Analytics Explorer** em Visual Studio Code. Em seguida, selecione uma pasta para seu projeto.

    ![Localizar trabalho ASA no Visual Studio Code](./media/vscode-explore-jobs/export-job.png)

    O projeto é exportado para a pasta que você selecionar e adicionou ao seu espaço de trabalho atual.

    ![Exportar trabalho ASA no Visual Studio Code](./media/stream-analytics-manage-job/copy-backup-stream-analytics-jobs.png)

3. Para publicar o trabalho em outra região ou backup usando outro nome, selecione **selecionar em suas assinaturas para publicar** no editor de consultas ( \* . asaql) e siga as instruções.

    ![Publicar no Azure no Visual Studio Code](./media/quick-create-visual-studio-code/submit-job.png)

## <a name="visual-studio"></a>Visual Studio

1. Siga o [trabalho exportar um Azure Stream Analytics implantado para uma instrução de projeto](./stream-analytics-vs-tools.md#export-jobs-to-a-project).

2. Abra o \* arquivo. asaql no editor de consultas, selecione **Enviar para o Azure** no editor de scripts e siga as instruções para publicar o trabalho em outra região ou backup usando um novo nome.

## <a name="next-steps"></a>Próximas etapas

* [Início rápido: criar um trabalho de Stream Analytics usando Visual Studio Code](quick-create-visual-studio-code.md)
* [Início rápido: criar um trabalho de Stream Analytics usando o Visual Studio](stream-analytics-quick-create-vs.md)
* [Implantar um trabalho do Azure Stream Analytics com CI/CD usando o Azure Pipelines](stream-analytics-tools-visual-studio-cicd-vsts.md)