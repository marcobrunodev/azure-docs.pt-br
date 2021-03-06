---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/06/2019
ms.author: alkohli
ms.openlocfilehash: 6e603442bdecf9a627172e24390b06cef573f475
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027682"
---
Dependendo do sistema operacional do cliente, os procedimentos para se conectar remotamente ao dispositivo são diferentes.

### <a name="remotely-connect-from-a-windows-client"></a>Conectar remotamente de um cliente Windows

Antes de começar, verifique se o seu cliente Windows está executando o Windows PowerShell 5,0 ou posterior.

Siga estas etapas para se conectar remotamente de um cliente Windows.

1. Execute uma sessão do Windows PowerShell como administrador.
2. Verifique se o serviço de Gerenciamento Remoto do Windows está em execução no seu cliente. No prompt de comando, digite:

    `winrm quickconfig`

3. Atribua uma variável ao endereço IP do dispositivo.

    $ip = "<device_ip>"

    Substitua `<device_ip>` pelo endereço IP do seu dispositivo.

4. Para adicionar o endereço IP do seu dispositivo à lista de hosts confiáveis do cliente, digite o seguinte comando:

    `Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force`

5. Inicie uma sessão do Windows PowerShell no dispositivo:

    `Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell`

6. Forneça a senha quando solicitado. Use a mesma senha usada para entrar na interface do usuário da Web local. A senha padrão da interface do usuário da Web local é *password1*. Quando você se conectar com êxito ao dispositivo usando o PowerShell remoto, você verá a seguinte saída de exemplo:  

    ```
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    
    PS C:\WINDOWS\system32> winrm quickconfig
    WinRM service is already running on this machine.
    PS C:\WINDOWS\system32> $ip = "10.100.10.10"
    PS C:\WINDOWS\system32> Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force
    PS C:\WINDOWS\system32> Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell

    WARNING: The Windows PowerShell interface of your device is intended to be used only for the initial network configuration. Please engage Microsoft Support if you need to access this interface to troubleshoot any potential issues you may be experiencing. Changes made through this interface without involving Microsoft Support could result in an unsupported configuration.
    [10.100.10.10]: PS>
    ```

### <a name="remotely-connect-from-a-linux-client"></a>Conectar remotamente de um cliente Linux

No cliente Linux que você usará para se conectar:

- [Instale o PowerShell Core mais recente para Linux](/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6) do GitHub para obter o recurso de comunicação remota do SSH. 
- [Instale apenas o `gss-ntlmssp` pacote do módulo NTLM](https://github.com/Microsoft/omi/blob/master/Unix/doc/setup-ntlm-omi.md). Para clientes do Ubuntu, use o seguinte comando:
    - `sudo apt-get install gss-ntlmssp`

Para obter mais informações, acesse [comunicação remota do PowerShell por SSH](/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-6).

Siga estas etapas para se conectar remotamente de um cliente NFS.

1. Para abrir a sessão do PowerShell, digite:

    `sudo pwsh`
 
2. Para se conectar usando o cliente remoto, digite:

    `Enter-PSSession -ComputerName $ip -Authentication Negotiate -ConfigurationName Minishell -Credential ~\EdgeUser`

    Quando solicitado, forneça a senha usada para entrar em seu dispositivo.
 
> [!NOTE]
> Este procedimento não funciona em Mac OS.