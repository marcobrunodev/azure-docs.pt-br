---
author: roygara
ms.service: storage
ms.topic: include
ms.date: 08/10/2020
ms.author: rogarana
ms.openlocfilehash: 8dcb58499113b0b7ae0814419f0a76965a0ed945
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681005"
---
#### <a name="additional-premium-file-share-level-limits"></a>Limites de nível de compartilhamento de arquivo Premium adicionais

|Área  |Destino  |
|---------|---------|
|Aumento/diminuição de tamanho mínimo    |1 GiB      |
|IOPS de linha de base    |1 IOPS por GiB, até 100.000|
|Intermitência de IOPS    |3x IOPS por GiB, até 100.000|
|Taxa de egresso         |60 MiB/s + 0, 6 * GiB provisionados        |
|Taxa de entrada| 40 MiB/s + 0, 4 * GiB provisionados |

#### <a name="file-level-limits"></a>Limites de nível de arquivo

|Área  |Arquivo padrão  |Arquivo Premium  |
|---------|---------|---------|
|Tamanho     |1 TiB         |4 TiB         |
|IOPS máxima por arquivo      |1,000         |Até 8.000 *         |
|Identificadores simultâneos     |2.000         |2.000         |
|Saída     |Consulte valores de taxa de transferência de arquivo padrão         |300 MiB/s (até 1 GiB/s com versão prévia do SMB multicanal) * *         |
|Entrada     |Consulte valores de taxa de transferência de arquivo padrão         |200 MiB/s (até 1 GiB/s com versão prévia do SMB multicanal) * *        |
|Taxa de transferência     |Até 60 MiB/s         |Consulte valores de entrada/saída do arquivo Premium         |

\*<sup>Aplica-se a leitura e gravação de IOS (normalmente tamanhos menores de e/s <= 64K). Operações de metadados, além de leituras e gravações, podem ser menores. </sup>

\*\*<sup>Sujeito a limites de rede do computador, largura de banda disponível, tamanhos de e/s, profundidade da fila e outros fatores. Para obter detalhes, consulte [desempenho de Multichannel do SMB](../articles/storage/files/storage-files-smb-multichannel-performance.md). </sup>
