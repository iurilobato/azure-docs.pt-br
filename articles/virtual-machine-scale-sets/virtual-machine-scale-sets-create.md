---
title: "Criar um conjunto de dimensionamento de máquinas virtuais do Azure | Microsoft Docs"
description: "Crie e implante um conjunto de dimensionamento de máquinas virtuais Linux ou Windows do Azure com a CLI do Azure, o PowerShell, um modelo ou o Visual Studio."
services: virtual-machine-scale-sets
documentationcenter: 
author: Thraka
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: article
ms.date: 03/30/2017
ms.author: adegeo
translationtype: Human Translation
ms.sourcegitcommit: aaf97d26c982c1592230096588e0b0c3ee516a73
ms.openlocfilehash: 736918ea310f276d961fa396f719b2b7809f0c0f
ms.lasthandoff: 04/27/2017

---

# <a name="create-and-deploy-a-virtual-machine-scale-set"></a>Criar e implantar um conjunto de dimensionamento de máquinas virtuais
Os conjuntos de escala de máquina virtual facilitam a implantação e o gerenciamento de máquinas virtuais idênticas como um conjunto. Os conjuntos de dimensionamento fornecem uma camada de computação altamente escalonável e personalizável para aplicativos de hiperescala e suporte a imagens da plataforma Windows, imagens da plataforma Linux, imagens personalizadas e extensões. Para obter mais informações sobre conjuntos de dimensionamento, consulte [Conjuntos de dimensionamento de máquinas virtuais](virtual-machine-scale-sets-overview.md).

Este tutorial mostra como criar um conjunto de dimensionamento de máquinas virtuais **sem** usar o portal do Azure. Para obter informações sobre como usar o portal do Azure, consulte [Como criar um conjunto de dimensionamento de máquinas virtuais com o portal do Azure](virtual-machine-scale-sets-portal-create.md).

>[!NOTE]
>Para saber mais sobre os recursos do Azure Resource Manager, confira [Azure Resource Manager vs. Implantação clássica](../azure-resource-manager/resource-manager-deployment-model.md).

## <a name="sign-in-to-azure"></a>Entrar no Azure

Se você estiver usando a CLI 2.0 do Azure ou o Azure PowerShell para criar um conjunto de dimensionamento, primeiro precisará se conectar à sua assinatura.

Para obter mais informações sobre como instalar, configurar e se conectar ao Azure com a CLI do Azure ou o PowerShell, consulte [Introdução à CLI 2.0 do Azure](/cli/azure/get-started-with-azure-cli.md) ou [Introdução aos cmdlets do Azure PowerShell](/powershell/azure/overview).

```azurecli
az login
```

```powershell
Login-AzureRmAccount
```

## <a name="create-a-resource-group"></a>Criar um grupo de recursos

Primeiro, você precisa criar um grupo de recursos ao qual o conjunto de dimensionamento de máquinas virtuais é associado.

```azurecli
az group create --location westus2 --name vmss-test-1
```

```powershell
New-AzureRmResourceGroup -Location westus2 -Name vmss-test-1
```

## <a name="create-from-azure-cli"></a>Criar com a CLI do Azure

Com a CLI do Azure, você pode criar um conjunto de dimensionamento de máquinas virtuais com esforço mínimo. Se você omitir valores padrão, eles serão fornecidos para você. Por exemplo, se você não especificar as informações de rede virtual, uma rede virtual será criada para você. Se você omitir as seguintes partes, elas serão criadas para você: 
- Um balanceador de carga
- Uma rede virtual
- Um endereço IP público

Ao escolher a imagem de máquina virtual que você deseja usar no conjunto de dimensionamento de máquinas virtuais, você tem algumas opções:

- URN  
O identificador de um recurso:  
**Win2012R2Datacenter**

- Alias do URN  
O nome amigável de um URN:  
**MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest**

- ID de recurso personalizado  
O caminho para um recurso do Azure:  
**/subscriptions/subscription-guid/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/images/MyImage**

- Recurso da Web  
O caminho para um URI do HTTP:  
**http://contoso.blob.core.windows.net/vhds/osdiskimage.vhd**

>[!TIP]
>Você pode encontrar a lista de imagens disponíveis com `az vm image list`.

Para criar um conjunto de dimensionamento de máquinas virtuais, você deve especificar o seguinte:

- Grupo de recursos 
- Nome
- Imagem do sistema operacional
- Informações de autenticação 
 
O exemplo a seguir cria um conjunto de dimensionamento de máquinas virtuais básico (essa etapa pode levar alguns minutos).

```azurecli
az vmss create --resource-group vmss-test-1 --name MyScaleSet --image UbuntuLTS --authentication-type password --admin-username azureuser --admin-password P@ssw0rd!
```

Após a conclusão do comando, o conjunto de dimensionamento de máquinas virtuais será criado. Talvez seja necessário obter o endereço IP da máquina virtual, de forma que você possa conectar-se a ela. É possível obter várias informações diferentes sobre a máquina virtual (incluindo o endereço IP) com o comando a seguir. 

```azurecli
az vmss list-instance-connection-info --resource-group vmss-test-1 --name MyScaleSet
```

## <a name="create-from-powershell"></a>Criar com o PowerShell

O PowerShell é mais complicado de usar do que a CLI do Azure. Enquanto a CLI do Azure fornece padrões para recursos relacionados à rede (balanceadores de carga, endereços IP e redes virtuais), isso não ocorre com o PowerShell. Referenciar uma imagem com o PowerShell também é um pouco mais complicado. Você pode obter imagens com os seguintes cmdlets:

1. Get-AzureRMVMImagePublisher
2. Get-AzureRMVMImageOffer
3. Get-AzureRmVMImageSku

O trabalho dos cmdlets pode ser colocado em sequência. Veja um exemplo de como obter todas as imagens para a região **Oeste dos EUA 2**, com um distribuidor que contém o nome **microsoft**.

```powershell
Get-AzureRMVMImagePublisher -Location WestUS2 | Where-Object PublisherName -Like *microsoft* | Get-AzureRMVMImageOffer | Get-AzureRmVMImageSku | Select-Object PublisherName, Offer, Skus
```

```
PublisherName              Offer                    Skus
-------------              -----                    ----
microsoft-ads              linux-data-science-vm    linuxdsvm
microsoft-ads              standard-data-science-vm standard-data-science-vm
MicrosoftAzureSiteRecovery Process-Server           Windows-2012-R2-Datacenter
MicrosoftBizTalkServer     BizTalk-Server           2013-R2-Enterprise
MicrosoftBizTalkServer     BizTalk-Server           2013-R2-Standard
MicrosoftBizTalkServer     BizTalk-Server           2016-Developer
MicrosoftBizTalkServer     BizTalk-Server           2016-Enterprise
...
```

O fluxo de trabalho para criar um conjunto de dimensionamento de máquinas virtuais é o seguinte:

1. Criar um objeto de configuração que retenha informações sobre o conjunto de dimensionamento.
2. Referenciar a imagem base do sistema operacional.
3. Definir as configurações do sistema operacional: autenticação, prefixo de nome da VM e usuário/frase secreta.
4. Configurar a rede.
5. Criar o conjunto de dimensionamento.

Este exemplo cria um conjunto de dimensionamento básico de duas instâncias para um computador com o Windows Server 2016 instalado.

```powershell
# Create a config object
$vmssConfig = New-AzureRmVmssConfig -Location WestUS2 -SkuCapacity 2 -SkuName Standard_A0  -UpgradePolicyMode Automatic

# Reference a virtual machine image from the gallery
Set-AzureRmVmssStorageProfile $vmssConfig -ImageReferencePublisher MicrosoftWindowsServer -ImageReferenceOffer WindowsServer -ImageReferenceSku 2016-Datacenter -ImageReferenceVersion latest

# Set up information for authenticating with the virtual machine
Set-AzureRmVmssOsProfile $vmssConfig -AdminUsername azureuser -AdminPassword P@ssw0rd! -ComputerNamePrefix myvmssvm

# Create the virtual network resources
$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name "my-subnet" -AddressPrefix 10.0.0.0/24
$vnet = New-AzureRmVirtualNetwork -Name "my-network" -ResourceGroupName "vmss-test-1" -Location "westus2" -AddressPrefix 10.0.0.0/16 -Subnet $subnet
$ipConfig = New-AzureRmVmssIpConfig -Name "my-ip-address" -LoadBalancerBackendAddressPoolsId $null -SubnetId $vnet.Subnets[0].Id

# Attach the virtual network to the config object
Add-AzureRmVmssNetworkInterfaceConfiguration -VirtualMachineScaleSet $vmssConfig -Name "network-config" -Primary $true -IPConfiguration $ipConfig

# Create the scale set with the config object (this step might take a few minutes)
New-AzureRmVmss -ResourceGroupName vmss-test-1 -Name my-scale-set -VirtualMachineScaleSet $vmssConfig
```

## <a name="create-from-a-template"></a>Criar usando um modelo

Você pode implantar um conjunto de dimensionamento de máquinas virtuais usando um modelo do Azure Resource Manager. É possível criar seu próprio modelo ou usar um do [repositório de modelos](https://azure.microsoft.com/resources/templates/?term=vmss). Esses modelos podem ser implantados diretamente em sua assinatura do Azure.

>[!NOTE]
>Para criar seu próprio modelo, crie um arquivo de texto JSON. Para obter informações gerais sobre como criar e personalizar um modelo, consulte [Modelos do Azure Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md).

Um modelo de exemplo está disponível [no GitHub](https://github.com/gatneil/mvss/tree/minimum-viable-scale-set). Para obter mais informações sobre como criar e usar essa amostra, consulte [Conjunto de dimensionamento mínimo viável](.\virtual-machine-scale-sets-mvss-start.md).

## <a name="create-from-visual-studio"></a>Criar com o Visual Studio

Com o Visual Studio, você pode criar um projeto do grupo de recursos do Azure e adicionar a ele um modelo do conjunto de dimensionamento de máquinas virtuais. Você pode escolher se deseja importá-lo do GitHub ou da Galeria de Aplicativos Web do Azure. Um script de implantação do PowerShell também é gerado para você. Para obter mais informações, consulte [Como criar um conjunto de dimensionamento de máquinas virtuais com o Visual Studio](virtual-machine-scale-sets-vs-create.md).

## <a name="create-from-the-azure-portal"></a>Criar com o portal do Azure

O portal do Azure é uma maneira conveniente de criar rapidamente um conjunto de dimensionamento. Para obter mais informações, consulte [Como criar um conjunto de dimensionamento de máquinas virtuais com o portal do Azure](virtual-machine-scale-sets-portal-create.md).

## <a name="next-steps"></a>Próximas etapas

Saiba mais sobre [discos de dados](virtual-machine-scale-sets-attached-disks.md).

Saiba como [gerenciar seus aplicativos](virtual-machine-scale-sets-deploy-app.md).

