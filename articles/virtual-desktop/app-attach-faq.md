---
title: Perguntas frequentes sobre o aplicativo MSIX de área de trabalho virtual do Windows – Azure
description: Perguntas frequentes sobre o MSIX app attach para a área de trabalho virtual do Windows.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 73fb9bf436c043e903977fafbb5a502e2edc5488
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518678"
---
# <a name="msix-app-attach-faq"></a>Perguntas frequentes sobre anexação de aplicativo MSIX

Este artigo responde às perguntas frequentes sobre o MSIX app attach para a área de trabalho virtual do Windows.

## <a name="whats-the-difference-between-msix-and-msix-app-attach"></a>Qual é a diferença entre a anexação de aplicativo MSIX e MSIX?

MSIX é um formato de empacotamento para aplicativos, enquanto a anexação do aplicativo MSIX é o recurso que fornece pacotes MSIX para sua implantação.

## <a name="does-msix-app-attach-use-fslogix"></a>A anexação do aplicativo MSIX usa FSLogix?

A anexação de aplicativo MSIX não usa FSLogix. No entanto, o MSIX app Attach e o FSLogix são projetados para trabalhar juntos para proporcionar uma experiência de usuário tranqüila.

## <a name="can-i-use-msix-app-attach-outside-of-windows-virtual-desktop"></a>Posso usar o anexo de aplicativo MSIX fora da área de trabalho virtual do Windows?

As APIs que o Power MSIX app Attach estão disponíveis para o Windows 10 Enterprise. Essas APIs podem ser usadas fora da área de trabalho virtual do Windows. No entanto, não há nenhum plano de gerenciamento para anexação de aplicativo MSIX fora da área de trabalho virtual do Windows.

## <a name="how-do-i-get-an-msix-package"></a>Como fazer obter um pacote MSIX?

Seu fornecedor de software lhe dará um pacote MSIX. Você também pode converter pacotes não MSIX em MSIX. Saiba mais em [como mover seus instaladores existentes para o MSIX](/windows/msix/packaging-tool/create-an-msix-overview#how-to-move-your-existing-installers-to-msix).

## <a name="which-operating-systems-support-msix-app-attach"></a>Quais sistemas operacionais dão suporte ao anexo do aplicativo MSIX?

Windows 10 Enterprise e Windows 10 Enterprise Multi-Session, versão 2004 ou posterior.

## <a name="is-msix-app-attach-currently-generally-available"></a>O anexo do aplicativo MSIX está atualmente disponível?

O MSIX app Attach faz parte do Windows 10 Enterprise e Windows 10 Enterprise Multi-Session, versão 2004 ou posterior. Atualmente, ambos os sistemas operacionais estão disponíveis. 

## <a name="can-i-use-msix-app-attach-outside-of-windows-virtual-desktop"></a>Posso usar o anexo de aplicativo MSIX fora da área de trabalho virtual do Windows?

As APIs de anexação de aplicativo MSIX e MSIX fazem parte do Windows 10 Enterprise e Windows 10 Enterprise Multi-Session, versão 2004 e posterior. Atualmente, não fornecemos software de gerenciamento para anexação de aplicativo MSIX fora da área de trabalho virtual do Windows.

## <a name="can-i-run-two-versions-of-the-same-application-at-the-same-time"></a>Posso executar duas versões do mesmo aplicativo ao mesmo tempo?

Para que duas versões dos mesmos aplicativos MSIX sejam executadas simultaneamente, a família de pacotes MSIX definida no arquivo de appxmanifest.xml deve ser diferente para cada aplicativo.

## <a name="should-i-disable-auto-update-when-using-msix-app-attach"></a>Devo desabilitar a atualização automática ao usar o anexo de aplicativo do MSIX?

Sim. A anexação de aplicativo MSIX não dá suporte à atualização automática para aplicativos MSIX.

## <a name="how-do-permissions-work-with-msix-app-attach"></a>Como as permissões funcionam com o anexo de aplicativo do MSIX?

Todas as VMs (máquinas virtuais) em um pool de hosts que usa o anexo de aplicativo MSIX devem ter permissões de leitura no compartilhamento de arquivos em que as imagens MSIX são armazenadas. Se ele também usar arquivos do Azure, será necessário conceder o RBAC (controle de acesso baseado em função) e as novas permissões de NTFS (sistema de arquivos de tecnologia).

## <a name="can-i-use-msix-app-attach-for-http-or-https"></a>Posso usar o anexo de aplicativo MSIX para HTTP ou HTTPs?

Todas as VMs que fazem parte de um pool de hosts que usa o anexo de aplicativo MSIX devem ter permissões de leitura no compartilhamento de arquivos em que as imagens MSIX são armazenadas. Se os arquivos do Azure estiverem sendo usados as permissões de RBAC e NTFS devem ser concedidas.

## <a name="can-i-restage-the-same-msix-application"></a>Posso descansar o mesmo aplicativo MSIX?

Sim. Você pode colocar os aplicativos que já foram repreparados e isso não deve causar erros.

## <a name="does-msix-app-attach-support-self-signed-certificates"></a>O MSIX app Attach dá suporte a certificados autoassinados?

Não há suporte para o uso do MSIX app Attach via HTTP ou HTTPs no momento.


## <a name="next-steps"></a>Próximas etapas

Se você quiser saber mais sobre o anexo de aplicativo do MSIX, Confira nossa [visão geral](what-is-app-attach.md) e [Glossário](app-attach-glossary.md). Caso contrário, comece a [Configurar a anexação de aplicativo](app-attach.md).
