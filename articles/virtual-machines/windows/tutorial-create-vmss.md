---
title: "Criar um conjunto de dimensionamento de máquinas virtuais para Windows no Azure | Microsoft Docs"
description: "Criar e implantar um aplicativo altamente disponível em VMs Windows usando um conjunto de dimensionamento de máquinas virtuais"
services: virtual-machine-scale-sets
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
tags: 
ms.assetid: 
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: 
ms.topic: article
ms.date: 05/01/2017
ms.author: iainfou
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: bbd4f044d85f2e22f27edc44b91fd42aef304ed2
ms.contentlocale: pt-br
ms.lasthandoff: 05/03/2017

---

# <a name="create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-windows"></a>Criar um conjunto de dimensionamento de máquinas virtuais e implantar um aplicativo altamente disponível no Windows
Neste tutorial, você aprenderá como conjuntos de escala de máquina virtual no Azure permitem que você dimensionar rapidamente o número de máquinas virtuais (VMs) que executam seu aplicativo. Um conjunto de dimensionamento de máquinas virtuais permite implantar e gerenciar um conjunto de máquinas virtuais idênticas de dimensionamento automático. Você pode dimensionar o número de VMs no conjunto de escala manualmente ou definir regras para dimensionamento automático com base no uso da CPU, a demanda por memória ou tráfego de rede. Para ver um conjunto de dimensionamento de máquinas virtuais em ação, crie um site básico do IIS que seja executado entre várias VMs Windows.

As etapas neste tutorial podem ser concluídas usando o módulo mais recente do [Azure PowerShell](/powershell/azureps-cmdlets-docs/).


## <a name="scale-set-overview"></a>Visão geral do conjunto de escala
Os conjuntos de dimensionamento usam conceitos semelhantes que você aprendeu no tutorial anterior para [criar VMs altamente disponíveis](tutorial-availability-sets.md). As VMs em um conjunto de dimensionamento são distribuídas entre domínios de falha e de atualização, assim como as VMs em um conjunto de disponibilidade.

VMs são criadas conforme necessário em um conjunto de escala. Você define regras de dimensionamento automático para controlar como e quando as VMs são adicionadas ou removidas do conjunto de dimensionamento. Essas regras podem ser disparados com base em métricas como carga de CPU, utilização de memória ou tráfego de rede.

Escala define suporte até 1.000 VMs quando você usar uma imagem da plataforma Windows Azure. Para cargas de trabalho com requisitos significativos de personalização de VM ou instalação, você pode querer [criar uma imagem de VM personalizada](tutorial-custom-images.md). Você pode criar até 100 VMs em uma escala definida ao usar uma imagem personalizada.


## <a name="create-an-app-to-scale"></a>Criar um aplicativo para escala
Antes de criar um conjunto de dimensionamento, crie um grupo de recursos com [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup). O seguinte exemplo cria um grupo de recursos chamado *myResourceGroupAutomate* no local *westus*:

```powershell
New-AzureRmResourceGroup -ResourceGroupName myResourceGroupScaleSet -Location westus
```

Em um tutorial anterior, você aprendeu a [automatizar a configuração da VM](tutorial-automate-vm-deployment.md) usando a Extensão de Script Personalizado. Crie uma configuração de conjunto de dimensionamento e aplique a Extensão de Script Personalizado para instalar e configurar o IIS:

```powershell
# Create a config object
$vmssConfig = New-AzureRmVmssConfig `
    -Location WestUS `
    -SkuCapacity 2 `
    -SkuName Standard_DS2 `
    -UpgradePolicyMode Automatic

# Define the script for your Custom Script Extension to run
$publicSettings = @{
    "fileUris" = (,"https://raw.githubusercontent.com/iainfoulds/azure-samples/master/automate-iis.ps1");
    "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}

# Use Custom Script Extension to install IIS and configure basic website
Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmssConfig `
    -Name "customScript" `
    -Publisher "Microsoft.Compute" `
    -Type "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -Setting $publicSettings
```

## <a name="create-scale-load-balancer"></a>Criar um balanceador de carga de dimensionamento
Um Azure Load Balancer é um balanceador de carga de Camada 4 (TCP, UDP) que fornece alta disponibilidade distribuindo o tráfego de entrada entre VMs íntegros. Uma investigação de integridade do balanceador de carga monitora uma determinada porta em cada VM e distribui o tráfego somente para uma VM operacional. Para saber mais, confira o próximo tutorial sobre [como balancear a carga de máquinas virtuais Windows](tutorial-load-balancer.md).

Crie um balanceador de carga que tenha um endereço IP público e distribua o tráfego da Web na porta 80:

```powershell
# Create a public IP address
$publicIP = New-AzureRmPublicIpAddress `
  -ResourceGroupName myResourceGroupScaleSet `
  -Location westus `
  -AllocationMethod Static `
  -Name myPublicIP

# Create a frontend and backend IP pool
$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig `
  -Name myFrontEndPool `
  -PublicIpAddress $publicIP
$backendPool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name myBackEndPool

# Create the load balancer
$lb = New-AzureRmLoadBalancer `
  -ResourceGroupName myResourceGroupScaleSet `
  -Name myLoadBalancer `
  -Location westus `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool

# Create a load balancer health probe on port 80
Add-AzureRmLoadBalancerProbeConfig -Name myHealthProbe `
  -LoadBalancer $lb `
  -Protocol tcp `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2

# Create a load balancer rule to distribute traffic on port 80
Add-AzureRmLoadBalancerRuleConfig `
  -Name myLoadBalancerRule `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80

# Update the load balancer configuration
Set-AzureRmLoadBalancer -LoadBalancer $lb
```

## <a name="create-a-scale-set"></a>Criar um conjunto de escala
Agora, crie um conjunto de dimensionamento de máquinas virtuais com [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvm). O seguinte exemplo cria um conjunto de dimensionamento chamado *myScaleSet*:

```powershell
# Reference a virtual machine image from the gallery
Set-AzureRmVmssStorageProfile $vmssConfig `
  -ImageReferencePublisher MicrosoftWindowsServer `
  -ImageReferenceOffer WindowsServer `
  -ImageReferenceSku 2016-Datacenter `
  -ImageReferenceVersion latest

# Set up information for authenticating with the virtual machine
Set-AzureRmVmssOsProfile $vmssConfig `
  -AdminUsername azureuser `
  -AdminPassword P@ssword! `
  -ComputerNamePrefix myVM

# Create the virtual network resources
$subnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.0.0/24
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myVnet" `
  -Location "westus" `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnet
$ipConfig = New-AzureRmVmssIpConfig `
  -Name "myIPConfig" `
  -LoadBalancerBackendAddressPoolsId $lb.BackendAddressPools[0].Id `
  -SubnetId $vnet.Subnets[0].Id

# Attach the virtual network to the config object
Add-AzureRmVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name "network-config" `
  -Primary $true `
  -IPConfiguration $ipConfig

# Create the scale set with the config object (this step might take a few minutes)
New-AzureRmVmss `
  -ResourceGroupName myResourceGroupScaleSet `
  -Name myScaleSet `
  -VirtualMachineScaleSet $vmssConfig     
```

Leva alguns minutos para criar e configurar todos os recursos e as VMs do conjunto de dimensionamento.


## <a name="test-your-app"></a>Testar seu aplicativo
Para ver seu site do IIS em ação, obtenha o endereço IP público do balanceador de carga com [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress). O seguinte exemplo obtém o endereço IP de *myPublicIP*, criado como parte do conjunto de dimensionamento:

```powershell
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupScaleSet -Name myPublicIP | select IpAddress
```

Insira o endereço IP público em um navegador da Web. O aplicativo é exibido, incluindo o nome do host da VM para a qual o balanceador de carga distribui o tráfego:

![Execução do site do IIS](./media/tutorial-create-vmss/running-iis-site.png)

Para ver a escala definida em ação, você pode forçar atualização seu navegador da web para ver o balanceador de carga distribuir tráfego entre todas as VMs que executam seu aplicativo.


## <a name="management-tasks"></a>Tarefas de gerenciamento
Durante todo o ciclo de vida do conjunto de dimensionamento, você poderá precisar executar uma ou mais tarefas de gerenciamento. Além disso, talvez você deseje criar scripts que automatizam várias tarefas do ciclo de vida. O Azure PowerShell fornece uma maneira rápida de realizar essas tarefas. Estas são algumas tarefas comuns.

### <a name="view-vms-in-a-scale-set"></a>Exibição de VMs em um conjunto de escala
Para exibir uma lista de VMs em execução no conjunto de dimensionamento, use [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm) como se segue:

```powershell
# Get current scale set
$scaleset = Get-AzureRmVmss `
  -ResourceGroupName myResourceGroupScaleSet `
  -VMScaleSetName myScaleSet

# Loop through the instanaces in your scale set
for ($i=0; $i -le ($set.Sku.Capacity - 1); $i++) {
    Get-AzureRmVmssVM -ResourceGroupName myResourceGroupScaleSet `
      -VMScaleSetName myScaleSet `
      -InstanceId $i
}
```


### <a name="increase-or-decrease-vm-instances"></a>Aumentar ou diminuir as instâncias de VM
Para ver o número de instâncias que você tem atualmente em um conjunto de dimensionamento, use [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss) e consulte *sku.capacity*:

```powershell
Get-AzureRmVmss -ResourceGroupName myResourceGroupScaleSet `
    -VMScaleSetName myScaleSet | `
    Select -ExpandProperty Sku
```

Manualmente, é possível aumentar ou diminuir o número de máquinas virtuais no conjunto de dimensionamento com [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss). O seguinte exemplo define o número de VMs no conjunto de dimensionamento como *5*:

```powershell
# Get current scale set
$scaleset = Get-AzureRmVmss `
  -ResourceGroupName myResourceGroupScaleSet `
  -VMScaleSetName myScaleSet

# Set and update the capacity of your scale set
$scaleset.sku.capacity = 5
Update-AzureRmVmss -ResourceGroupName myResourceGroupScaleSet `
    -Name myScaleSet `
    -VirtualMachineScaleSet $scaleset
```

A atualização para o número especificado de instâncias no conjunto de dimensionamento leva alguns minutos.


### <a name="configure-autoscale-rules"></a>Configurar regras de dimensionamento automático
Em vez de dimensionar manualmente o número de instâncias no conjunto de dimensionamento, defina regras de dimensionamento automático. Essas regras monitoram as instâncias no conjunto de dimensionamento e respondem adequadamente com base nas métricas e nos limites que você define. O exemplo a seguir escala horizontalmente o número de instâncias em um quando a carga média da CPU é maior do que 60% por um período superior a 5 minutos. Se a carga média da CPU cair abaixo de 30% em um período superior a 5 minutos, as instâncias serão reduzidas horizontalmente em uma instância:

```powershell
# Define your scale set information
$mySubscriptionId = (Get-AzureRmSubscription).SubscriptionId
$myResourceGroup = "myResourceGroupScaleSet"
$myScaleSet = "myScaleSet"
$myLocation = "West US"

# Create a scale up rule to increase the number instances after 60% average CPU usage exceeded for a 5 minute period
$myRuleScaleUp = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator GreaterThan `
  -MetricStatistic Average `
  -Threshold 60 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Increase `
  -ScaleActionValue 1

# Create a scale down rule to decrease the number of instances after 30% average CPU usage over a 5 minute period
$myRuleScaleDown = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator LessThan `
  -MetricStatistic Average `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Decrease `
  -ScaleActionValue 1

# Create a scale profile with your scale up and scale down rules
$myScaleProfile = New-AzureRmAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rules $myRuleScaleUp,$myRuleScaleDown `
  -Name "autoprofile"

# Apply the autoscale rules
Add-AzureRmAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -AutoscaleProfiles $myScaleProfile
```


## <a name="next-steps"></a>Próximas etapas
Neste tutorial, você aprendeu como criar um conjunto de escala de máquina virtual. Avança para o próximo tutorial para saber mais sobre conceitos de máquinas virtuais de balanceamento de carga.

[Balancear carga de máquinas virtuais](tutorial-load-balancer.md)
