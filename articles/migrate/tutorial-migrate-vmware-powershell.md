---
title: Migrar VMs VMware para o Azure (sem agente) – PowerShell
description: Saiba como executar uma migração sem agente de VMs VMware com as Migrações para Azure por meio do PowerShell.
services: ''
author: rahugup
manager: bsiva
ms.topic: tutorial
ms.date: 10/1/2020
ms.author: rahugup
ms.openlocfilehash: 3cd8d29a498a6a00fa8fff679afc969b339934b1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494334"
---
# <a name="migrate-vmware-vms-to-azure-agentless---powershell"></a>Migrar VMs VMware para o Azure (sem agente) – PowerShell

Neste artigo, você aprende a migrar VMs VMware descobertas com o método sem agente usando o Azure PowerShell para [Migrações para Azure: Migração de Servidor](migrate-services-overview.md#azure-migrate-server-migration-tool). 

Você aprenderá como:

> [!div class="checklist"]
> * Recuperar VMs VMware descobertas em um projeto das Migrações para Azure.
> * Iniciar a replicação de VMs.
> * Atualizar propriedades para replicar VMs.
> * Monitorar a replicação.
> * Executar uma migração de teste para verificar se tudo está funcionando conforme o esperado.
> * Executar uma migração de VM completa.

> [!NOTE]
> Os tutoriais mostram o caminho de implantação mais simples para um cenário para que você possa configurar rapidamente uma prova de conceito. Os tutoriais usam opções padrão quando possível e não mostram todas as configurações e todos os caminhos possíveis.

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

Antes de iniciar este tutorial, você deverá:

1. [Concluir o tutorial de descoberta](tutorial-discover-vmware.md) para preparar o Azure e o VMware para migração.
2. Recomendamos que você conclua o segundo tutorial para [avaliar as VMs VMware](./tutorial-assess-vmware-azure-vm.md) antes de migrá-las para o Azure.
3. Você precisa ter o módulo `Az` do Azure PowerShell. Se precisar instalar ou atualizar o Azure PowerShell, siga este [guia para instalar e configurar o Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="install-azure-migrate-powershell-module"></a>Instalar o módulo do PowerShell das Migrações para Azure

O módulo do PowerShell das Migrações para Azure está disponível na versão prévia. Você precisará instalar o módulo do PowerShell usando o comando a seguir. 

```azurepowershell
Install-Module -Name Az.Migrate 
```

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Entrar na assinatura do Microsoft Azure

Entre em sua assinatura do Azure com o cmdlet `Connect-AzAccount`.

```azurepowershell
Connect-AzAccount
```

Selecione sua assinatura do Azure. Use o cmdlet `Get-AzSubscription` para obter a lista de assinaturas do Azure a que você tem acesso. Selecione a assinatura do Azure que tem seu projeto das Migrações para Azure com o qual trabalhar usando o cmdlet `Set-AzContext`.

```azurepowershell
Set-AzContext -SubscriptionId "xxxx-xxxx-xxxx-xxxx"
```

## <a name="retrieve-the-azure-migrate-project"></a>Recuperar o projeto das Migrações para Azure

Um projeto das Migrações para Azure é usado para armazenar metadados de descoberta, avaliação e migração coletados do ambiente que você está avaliando ou migrando.
Em um projeto, é possível acompanhar os ativos descobertos, orquestrar avaliações e executar migrações.

Como parte dos pré-requisitos, você já teria criado um projeto das Migrações para Azure. Use o cmdlet `Get-AzMigrateProject` para recuperar detalhes de um projeto das Migrações para Azure. Você precisará especificar o nome do projeto das Migrações para Azure (`Name`) e o nome do grupo de recursos desse projeto (`ResourceGroupName`).

```azurepowershell
# Get resource group of the Azure Migrate project
$ResourceGroup = Get-AzResourceGroup -Name "MyResourceGroup"

# Get details of the Azure Migrate project
$MigrateProject = Get-AzMigrateProject -Name "MyMigrateProject" -ResourceGroupName $ResourceGroup.ResourceGroupName

# View Azure Migrate project details
$MigrateProject | ConvertTo-JSON
```

## <a name="retrieve-discovered-vms-in-an-azure-migrate-project"></a>Recuperar VMs descobertas em um projeto das Migrações para Azure

As Migrações para Azure usam um [dispositivo leve das Migrações para Azure](migrate-appliance-architecture.md). Como parte dos pré-requisitos, você teria implantado o dispositivo das Migrações para Azure como uma VM VMware.

Para recuperar uma VM VMware específica em um projeto das Migrações para Azure, especifique o nome (`ProjectName`) e o grupo de recursos (`ResourceGroupName`) do projeto das Migrações para Azure e o nome da VM (`DisplayName`). 

> [!NOTE]
> **O valor do parâmetro de nome da VM (`DisplayName`) diferencia maiúsculas de minúsculas**.

```azurepowershell
# Get a specific VMware VM in an Azure Migrate project
$DiscoveredServer = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -DisplayName "MyTestVM"

# View discovered server details
$DiscoveredServer | ConvertTo-JSON
```
Migraremos essa VM como parte deste tutorial.

Você também pode recuperar todas as VMs VMware em um projeto das Migrações para Azure usando os parâmetros `ProjectName` e `ResourceGroupName`.

```azurepowershell
# Get all VMware VMs in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName 
```
Se tiver vários dispositivos em um projeto das Migrações para Azure, você poderá usar os parâmetros `ProjectName`, `ResourceGroupName` e `ApplianceName` para recuperar todas as VMs descobertas usando um dispositivo específico das Migrações para Azure. 

```azurepowershell
# Get all VMware VMs discovered by an Azure Migrate Appliance in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -ApplianceName "MyMigrateAppliance" |Format-Table DisplayName, Name, Type

```

## <a name="initialize-replication-infrastructure"></a>Inicializar a infraestrutura de replicação

[Migrações para Azure: Migração de Servidor](migrate-services-overview.md#azure-migrate-server-migration-tool) aproveita vários recursos do Azure para migrar VMs. A Migração de Servidor provisiona os recursos a seguir no mesmo grupo de recursos que o projeto.

- **Barramento de serviço**: Migração de Servidor usa o Barramento de Serviço para enviar mensagens de orquestração de replicação para o dispositivo.
- **Conta de armazenamento de gateway**: a Migração de Servidor usa a conta de armazenamento de gateway para armazenar informações de estado de armazenamento sobre as VMs que estão sendo replicadas.
- **Conta de armazenamento de log**: o dispositivo das Migrações para Azure carrega os logs de replicação das VMs em uma conta de armazenamento de log. As Migrações para Azure aplicam as informações de replicação aos discos gerenciados de réplica.
- **Cofre de chaves**: O dispositivo das Migrações para Azure usa o cofre de chaves para gerenciar as cadeias de conexão do barramento de serviço e as chaves de acesso das contas de armazenamento usadas na replicação.

Antes de replicar a primeira VM no projeto das Migrações para Azure, execute o script a seguir para provisionar a infraestrutura de replicação. O script provisiona e configura os recursos mencionados anteriormente para que você possa começar a migrar suas VMs VMware.

> [!NOTE]
> Um projeto das Migrações para Azure dá suporte a migrações para apenas uma região do Azure. Após executar o script, você não poderá alterar a região de destino para a qual deseja migrar as VMs VMware.
> Você precisará executar o script `Initialize-AzMigrateReplicationInfrastructure` se configurar um novo dispositivo em seu projeto das Migrações para Azure. 

No artigo, inicializaremos a infraestrutura de replicação para que possamos migrar nossas VMs para a região `Central US`. Você pode [baixar o arquivo](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/migrate-at-scale-vmware-agentles) do repositório do GitHub ou executá-lo usando o snippet a seguir. 

```azurepowershell
# Download the script from Azure Migrate GitHub repository 
Invoke-WebRequest https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-migrate/migrate-at-scale-vmware-agentles/Initialize-AzMigrateReplicationInfrastructure.ps1 -OutFile .\AzMigrateReplicationinfrastructure.ps1

# Run the script for initializing replication infrastructure for the current Migrate project
.\AzMigrateReplicationInfrastructure.ps1 -ResourceGroupName $ResourceGroup.ResourceGroupName -ProjectName $MigrateProject.Name -Scenario agentlessVMware -TargetRegion "CentralUS" 
```


## <a name="replicate-vms"></a>Replicar VMs

Após concluir a descoberta e inicializar a infraestrutura de replicação, você pode começar a replicação das VMs VMware para o Azure. Você pode executar até 300 replicações simultaneamente.

Você pode especificar as propriedades da replicação conforme mostrado a seguir.

- **Assinatura e grupo de recursos de destino** – especifique a assinatura e o grupo de recursos para os quais a VM deve ser migrada fornecendo a ID do grupo de recursos usando o parâmetro `TargetResourceGroupId`. 
- **Rede virtual e sub-rede de destino** – especifique a ID da Rede Virtual do Azure e o nome da sub-rede para a qual a VM deve ser migrada usando os parâmetros `TargetNetworkId` e `TargetSubnetName`, respectivamente. 
- **Nome da VM de destino** – especifique o nome da VM do Azure a ser criada usando o parâmetro `TargetVMName`.
- **Tamanho da VM de destino** – especifique o tamanho da VM do Azure a ser usada para replicação da VM usando o parâmetro `TargetVMSize`. Por exemplo, para migrar uma VM para a VM D2_v2 no Azure, especifique o valor de `TargetVMSize` como "Standard_D2_v2".  
- **Licença** – para usar o Benefício Híbrido do Azure para computadores Windows Server cobertos por assinaturas ativas do Software Assurance ou do Windows Server, especifique o valor do parâmetro `LicenseType` como "WindowsServer". Caso contrário, especifique o valor do parâmetro `LicenseType` como "NoLicenseType".
- **Disco do SO** – especifique o identificador exclusivo do disco que tem o carregador de inicialização e o instalador do sistema operacional. A ID do disco a ser usada é a propriedade do UUID (identificador exclusivo) do disco, recuperada usando o cmdlet `Get-AzMigrateServer`.
- **Tipo de disco** – especifique o valor do parâmetro `DiskType` da maneira exibida a seguir.
    - Para usar discos gerenciados Premium, especifique "Premium_LRS" como valor do parâmetro `DiskType`. 
    - Para usar discos SSD Standard, especifique "StandardSSD_LRS" como valor do parâmetro `DiskType`. 
    - Para usar discos HDD Standard, especifique "Standard_LRS" como valor para o parâmetro `DiskType`. 
- **Redundância de infraestrutura** – especifique a opção de redundância da infraestrutura da maneira mostrada a seguir. 
    - Zona de Disponibilidade para fixar o computador migrado para uma Zona de Disponibilidade específica na região. Use essa opção para distribuir servidores que formam uma camada de aplicativo de vários nós entre Zonas de Disponibilidade diferentes. Essa opção estará disponível somente se a região de destino selecionada para a migração der suporte a Zonas de Disponibilidade. Para usar zonas de disponibilidade, especifique o valor da zona de disponibilidade para o parâmetro `TargetAvailabilityZone`.
    - Conjunto de Disponibilidade para colocar o computador migrado em um conjunto de disponibilidade. O Grupo de Recursos de destino selecionado precisa ter um ou mais conjuntos de disponibilidade para usar essa opção. Para usar o conjunto de disponibilidade, especifique a ID do conjunto de disponibilidade para o parâmetro `TargetAvailabilitySet`. 

### <a name="replicate-vms-with-all-disks"></a>Replicar VMs com todos os discos
Neste tutorial, replicaremos todos os discos da VM descoberta e especificaremos um novo nome para a VM no Azure. Especificamos o primeiro disco do servidor descoberto como Disco do SO e migramos todos os discos como HDD Standard. O disco do sistema operacional é o disco que tem o carregador de inicialização e o instalador do sistema operacional. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

```azurepowershell
# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name "MyTargetResourceGroup"

# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name "MyVirtualNetwork"

# Start replication for a discovered VM in an Azure Migrate project
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType "NoLicenseType" -OSDiskID $DiscoveredServer.Disk[0].Uuid -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -DiskType "Standard_LRS" -TargetVMName "MyMigratedTestVM" -TargetVMSize "Standard_DS2_v2"

# Track job status to check for completion
while (($MigrateJob.State -eq "InProgress") -or ($MigrateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $MigrateJob.State
```

### <a name="replicate-vms-with-select-disks"></a>Replicar VMs com discos selecionados
Você também pode replicar seletivamente os discos da VM descoberta usando o cmdlet `New-AzMigrateDiskMapping` e fornecendo isso como entrada para o parâmetro `DiskToInclude` no cmdlet `New-AzMigrateServerReplication`. Também é possível usar o cmdlet `New-AzMigrateDiskMapping` para especificar tipos de disco de destino diferentes para cada disco individual a ser replicado. 

Especifique valores para os parâmetros a seguir do cmdlet `New-AzMigrateDiskMapping`.

- **DiskId** – especifique o identificador exclusivo do disco a ser migrado. A ID do disco a ser usada é a propriedade do UUID (identificador exclusivo) do disco, recuperada usando o cmdlet `Get-AzMigrateServer`.  
- **IsOSDisk** – especifique "true" se o disco a ser migrado for o disco do SO da VM; caso contrário, "false".
- **DiskType** – especifique o tipo de disco a ser usado no Azure. 

No exemplo a seguir, replicaremos somente dois discos da VM descoberta. Especificaremos o disco do SO e usaremos tipos de disco diferentes para cada disco a ser replicado. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

```azurepowershell
# View disk details of the discovered server
$DiscoveredServer.Disk | ConvertTo-JSON

# Create a new disk mapping for the disks to be replicated
$DisksToReplicate = @()
$OSDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[0].Uuid -DiskType "StandardSSD_LRS" -IsOSDisk "true"
$DataDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[1].Uuid -DiskType "Premium_LRS" -IsOSDisk "false"

$DisksToReplicate += $OSDisk
$DisksToReplicate += $DataDisk 

# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name "MyTargetResourceGroup"

# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name "MyVirtualNetwork"

# Start replication for the VM
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType "NoLicenseType" -DiskToInclude $DisksToReplicate -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -TargetVMName "MyMigratedTestVM" -TargetVMSize "Standard_DS2_v2"

# Track job status to check for completion
while (($MigrateJob.State -eq "InProgress") -or ($MigrateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $MigrateJob.State
```

## <a name="monitor-replication"></a>Monitorar a replicação 

A replicação ocorre da seguinte maneira:

- Quando o trabalho Iniciar Replicação é concluído com êxito, os computadores começam sua replicação inicial para o Azure.
- Durante a replicação inicial, um instantâneo de VM é criado. Os dados do disco do instantâneo são replicados para discos gerenciados de réplica no Azure.
- Depois que a replicação inicial é concluída, a replicação delta é iniciada. As alterações incrementais em discos locais são replicadas periodicamente para os discos de réplica no Azure.

Acompanhe o status da replicação usando o cmdlet `Get-AzMigrateServerReplication`. 

> [!NOTE]
> A ID da VM descoberta e a ID da VM de replicação são identificadores exclusivos diferentes. Os dois identificadores podem ser usados para recuperar detalhes de um servidor de replicação.  

### <a name="monitor-replication-using-discovered-vm-identifier"></a>Monitorar a replicação usando o identificador da VM descoberta
```azurepowershell
# Retrieve the replicating VM details by using the discovered VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -DiscoveredMachineId $DiscoveredServer.ID
```

### <a name="monitor-replication-using-replicating-vm-identifier"></a>Monitorar a replicação o identificador da VM de replicação

```azurepowershell
# List all replicating VMs in an Azure Migrate project and filter the result for selecting the replication VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName | where MachineName -eq $DiscoveredServer.DisplayName

# Retrieve replicating VM details using replicating VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id 
```

Você pode acompanhar as propriedades "Estado da migração" e "Descrição do estado da migração" na saída. 
- Para a replicação inicial, os valores das propriedades estado da migração e descrição do estado da migração serão "InitialSeedingInProgress" e "Replicação inicial", respectivamente. 
- Durante a replicação delta, os valores das propriedades do estado da migração e da descrição do estado da migração serão "Replicando" e "Pronto para migrar", respectivamente.
- Após a conclusão da migração, os valores das propriedades do estado da migração e da descrição do estado da migração serão "Migração bem-sucedida" e Migrado", respectivamente.

```Output
AllowedOperation            : {DisableMigration, TestMigrate, Migrate}
CurrentJobId                : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationJobs/None
CurrentJobName              : None
CurrentJobStartTime         : 1/1/1753 1:01:01 AM
EventCorrelationId          : 9d435c55-4660-41a5-a8ed-dd74213d85fa
Health                      : Normal
HealthError                 : {}
Id                          : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationFabrics/xxx/replicationProtectionContainers/xxx/
                              replicationMigrationItems/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-
                              39b2-1e14-f90584522703
LastTestMigrationStatus     :
LastTestMigrationTime       :
Location                    :
MachineName                 : MyTestVM
MigrationState              : InitialSeedingInProgress
MigrationStateDescription   : Initial replication
Name                        : 10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-39b2-1e14-f90584522703
PolicyFriendlyName          : xxx
PolicyId                    : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationPolicies/xxx
ProviderSpecificDetail      : Microsoft.Azure.PowerShell.Cmdlets.Migrate.Models.Api20180110.VMwareCbtMigrationDetails
TestMigrateState            : None
TestMigrateStateDescription : None
Type                        : Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems
```

Para obter detalhes sobre o progresso da replicação, execute o cmdlet a seguir.

```azurepowershell
$replicatingserver.ProviderSpecificDetail | convertto-json
```
Você pode acompanhar o progresso da replicação inicial usando as propriedades de "Percentual de progresso da propagação inicial" na saída.

```output
    "DataMoverRunAsAccountId": "/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.OffAzure/VMwareSites/xxx/runasaccounts/xxx",
    "FirmwareType":  "BIOS",
    "InitialSeedingProgressPercentage": 20,
    "InstanceType":  "VMwareCbt",
    "LastRecoveryPointReceived":  "\/Date(1601733591427)\/",
    "LicenseType":  "NoLicenseType",
    "MigrationProgressPercentage":  null,
    "MigrationRecoveryPointId":  null,
    "OSType":  "Windows",
    "PerformAutoResync":  "true",
```

A replicação ocorre da seguinte maneira:

- Quando o trabalho Iniciar Replicação é concluído com êxito, os computadores começam sua replicação inicial para o Azure.
- Durante a replicação inicial, um instantâneo de VM é criado. Os dados do disco do instantâneo são replicados para discos gerenciados de réplica no Azure.
- Depois que a replicação inicial é concluída, a replicação delta é iniciada. As alterações incrementais em discos locais são replicadas periodicamente para os discos de réplica no Azure.

## <a name="retrieve-the-status-of-a-job"></a>Recuperar o status de um trabalho

Você pode monitorar o status de um trabalho usando o cmdlet `Get-AzMigrateJob`. 

```azurepowershell
# Retrieve the updated status for a job
$job = Get-AzMigrateJob -InputObject $job
```

## <a name="update-properties-of-a-replicating-vm"></a>Atualizar propriedades de uma VM de replicação

[Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool) permite que você altere as propriedades de destino, como nome, tamanho, grupo de recursos, configuração de NIC e assim por diante, de uma VM de replicação. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

```azurepowershell
# Retrieve the replicating VM details by using the discovered VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -DiscoveredMachineId $DiscoveredServer.ID

# View NIC details of the replicating server
Write-Output $ReplicatingServer.ProviderSpecificDetail.VMNic 
```
As propriedades a seguir podem ser atualizadas para uma VM.

- **Nome da VM** – especifique o nome da VM do Azure a ser criada usando o parâmetro `TargetVMName`.
- **Tamanho da VM** – especifique o tamanho da VM do Azure a ser usada para replicação da VM usando o parâmetro `TargetVMSize`. Por exemplo, para migrar uma VM para a VM D2_v2 no Azure, especifique o valor de `TargetVMSize` como "Standard_D2_v2".  
- **Rede Virtual** – especifique a ID da Rede Virtual do Azure para a qual a VM deve ser migrada usando o parâmetro `TargetNetworkId`. 
- **Grupo de Recursos** – especifique a ID do grupo de recursos para o qual a VM deve ser migrada fornecendo a ID do grupo de recursos usando o parâmetro `TargetResourceGroupId`.
- **Adaptador de Rede** – a configuração de NIC pode ser especificada usando o cmdlet `New-AzMigrateNicMapping`. O objeto passa uma entrada para o parâmetro `NicToUpdate` no cmdlet `Set-AzMigrateServerReplication`. 

    - **Alterara alocação de IP** – para especificar um IP estático para uma NIC, forneça o endereço IPv4 a ser usado como IP estático para a VM usando o parâmetro `TargetNicIP`. Para atribuir dinamicamente um IP para uma NIC, informe "auto" como o valor do parâmetro `TargetNicIP`.
    - Use os valores "Primary", "Secondary" ou "DoNotCreate" para o parâmetro `TargetNicSelectionType` a fim de especificar se a NIC deve ser primária, secundária ou se não deve ser criada na VM migrada. Somente uma NIC pode ser especificada como NIC primária para a VM. 
    - Para tornar uma NIC primária, você também precisará especificar as outras NICs que devem se tornar secundárias ou que não devem ser criadas na VM migrada.  
    - Para alterar a sub-rede da NIC, especifique o nome da sub-rede usando o parâmetro `TargetNicSubnet`.

 - **Zona de Disponibilidade** – para usar zonas de disponibilidade, especifique o valor da zona de disponibilidade para o parâmetro `TargetAvailabilityZone`.
 - **Conjunto de Disponibilidade** – para usar o conjunto de disponibilidade, especifique a ID do conjunto de disponibilidade para o parâmetro `TargetAvailabilitySet`.

No exemplo a seguir, atualizaremos a configuração da NIC tornando a primeira NIC primária e atribuindo um IP estático a ela. Descartaremos a segunda NIC para migração e atualizaremos o nome e o tamanho da VM de destino. 

```azurepowershell
# Specify the NIC properties to be updated for a replicating VM. 
$NicMapping = @()
$NicMapping1 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[0].NicId -TargetNicIP "xxx.xxx.xxx.xxx" -TargetNicSelectionType "Primary"
$NicMapping2 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[1].NicId -TargetNicSelectionType "DoNotCreate"

$NicMapping += $NicMapping1
$NicMapping += $NicMapping2

# Update the name, size and NIC configuration of a replicating server
$UpdateJob = Set-AzMigrateServerReplication -InputObject $ReplicatingServer -TargetVMSize "Standard_DS13_v2" -TargetVMName "MyMigratedVM" -NicToUpdate $NicMapping

# Track job status to check for completion
while (($UpdateJob.State -eq "InProgress") -or ($UpdateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $UpdateJob = Get-AzMigrateJob -InputObject $UpdateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $UpdateJob.State
```

Você também pode listar todos os servidores de replicação em um projeto das Migrações para Azure e, depois, usar o identificador da VM de replicação para atualizar as propriedades da VM.

```azurepowershell
# List all replicating VMs in an Azure Migrate project and filter the result for selecting the replication VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName | where MachineName -eq $DiscoveredServer.DisplayName

# Retrieve replicating VM details using replicating VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id 
```


## <a name="run-a-test-migration"></a>Execute um teste de migração

Quando a replicação delta é iniciada, é possível executar uma migração de teste para as VMs antes de executar uma migração completa para o Azure. É altamente recomendável fazer a migração de teste pelo menos uma vez para cada computador antes de migrá-lo. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

- A execução da migração de teste verifica se a migração funcionará conforme o esperado. A migração de teste não afeta o computador local, que permanece operacional e continua replicando. 
- A migração de teste simula a migração criando uma VM do Azure com o uso dos dados replicados (geralmente, fazendo a migração para uma VNet de não produção em sua assinatura do Azure).
- É possível usar a VM do Azure de teste replicada para validar a migração, executar testes de aplicativo e resolver problemas antes da migração completa.

Selecione a Rede Virtual do Azure a ser usada para testes especificando a ID da rede virtual usando o parâmetro `TestNetworkID`.

```azurepowershell
# Retrieve the Azure virtual network created for testing 
$TestVirtualNetwork = Get-AzVirtualNetwork -Name MyTestVirtualNetwork

# Start test migration for a replicating server
$TestMigrationJob = Start-AzMigrateTestMigration -InputObject $ReplicatingServer -TestNetworkID $TestVirtualNetwork.Id

# Track job status to check for completion
while (($TestMigrationJob.State -eq "InProgress") -or ($TestMigrationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $TestMigrationJob = Get-AzMigrateJob -InputObject $TestMigrationJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TestMigrationJob.State
```

Após a conclusão do teste, limpe a migração de teste usando o cmdlet `Start-AzMigrateTestMigrationCleanup`. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

```azurepowershell
# Clean-up test migration for a replicating server
$CleanupTestMigrationJob = Start-AzMigrateTestMigrationCleanup -InputObject $ReplicatingServer

# Track job status to check for completion
while (($CleanupTestMigrationJob.State -eq "InProgress") -or ($CleanupTestMigrationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $CleanupTestMigrationJob = Get-AzMigrateJob -InputObject $CleanupTestMigrationJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $CleanupTestMigrationJob.State
```

## <a name="migrate-vms"></a>Migrar VMs

Depois de verificar se a migração de teste funciona conforme o esperado, você pode migrar o servidor de replicação usando o cmdlet a seguir. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

Se não quiser desligar o servidor de origem, não use o parâmetro `TurnOffSourceServer`.

```azurepowershell
# Start migration for a replicating server and turn off source server as part of migration
$MigrateJob = Start-AzMigrateServerMigration -InputObject $ReplicatingServer -TurnOffSourceServer 

# Track job status to check for completion
while (($MigrateJob.State -eq "InProgress") -or ($MigrateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $MigrateJob.State
```

## <a name="complete-the-migration"></a>Concluir a migração

1. Após a conclusão da migração, interrompa a replicação para o computador local e limpe as informações do estado de replicação da VM usando o cmdlet a seguir. O cmdlet retorna um trabalho que pode ser rastreado para monitorar o status da operação. 

```azurepowershell
# Stop replication for a migrated server
$StopReplicationJob = Remove-AzMigrateServerReplication -InputObject $ReplicatingServer 

# Track job status to check for completion
while (($StopReplicationJob.State -eq "InProgress") -or ($StopReplicationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $StopReplicationJob = Get-AzMigrateJob -InputObject $StopReplicationJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $StopReplicationJob.State
```

2. Instale o agente do [Windows](../virtual-machines/extensions/agent-windows.md) ou do [Linux](../virtual-machines/extensions/agent-linux.md) da VM do Azure nos computadores migrados.
3. Execute todos os ajustes no aplicativo após a migração, como atualizar as cadeias de conexão de banco de dados e as configurações do servidor Web.
4. Execute o aplicativo final e o teste de aceitação da migração no aplicativo migrado que está sendo executado no Azure.
5. Transfira o tráfego para a instância migrada da VM do Azure.
6. Remova as VMs locais do inventário local de VMs.
7. Remova as VMs locais dos backups locais.
8. Atualize todas as documentações internas para mostrar o novo local e o endereço IP das VMs do Azure. 

## <a name="post-migration-best-practices"></a>Melhores práticas pós-migração

- Para aumentar a resiliência:
    - Proteja os dados fazendo backup das VMs do Azure por meio do serviço Backup do Azure. [Saiba mais](../backup/quick-backup-vm-portal.md).
    - Mantenha as cargas de trabalho em execução e continuamente disponíveis ao replicar as VMs do Azure em uma região secundária com o Site Recovery. [Saiba mais](../site-recovery/azure-to-azure-tutorial-enable-replication.md).
- Para aumentar a segurança:
    - Bloqueie e limite o acesso ao tráfego de entrada com a [Central de Segurança do Azure – Administração just-in-time](../security-center/security-center-just-in-time.md).
    - Restrinja o tráfego de rede a pontos de extremidade com os [Grupos de Segurança de Rede](../virtual-network/network-security-groups-overview.md).
    - Implante o [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) para manter os discos em segurança e proteger os dados contra roubo e acesso não autorizado.
    - Leia mais sobre [como proteger recursos IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) e acesse a [Central de Segurança do Azure](https://azure.microsoft.com/services/security-center/).
- Para monitoramento e gerenciamento:
-  Considere implantar o [Gerenciamento de Custos do Azure](../cost-management-billing/cloudyn/overview.md) para monitorar o uso de recursos e os gastos.