---
title: Exemplo de script da CLI do Azure - Criar uma VM do Windows Server 2016 com o IIS usando a DSC | Microsoft Docs
description: Exemplo de script da CLI do Azure - Criar uma VM do Windows Server 2016 com o IIS usando a DSC
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rickstercdn
manager: timlt
editor: tysonn
tags: 
ms.assetid: 
ms.service: virtual-machines-Windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
ms.translationtype: Human Translation
ms.sourcegitcommit: 6ea03adaabc1cd9e62aa91d4237481d8330704a1
ms.openlocfilehash: ffcca64e078840050f38b68cd23036dbbbfaee0a
ms.contentlocale: pt-br
ms.lasthandoff: 04/06/2017

---

# <a name="create-a-vm-with-iis-using-dsc"></a>Criar uma máquina virtual com o IIS usando a DSC

Esse script cria uma máquina virtual e usa a extensão de script personalizado de DSC da Máquina Virtual do Azure para instalar e configurar o IIS. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de exemplo

[!code-azurecli[principal](../../../cli_scripts/virtual-machine/create-windows-iis-using-dsc/create-windows-iis-using-dsc.sh "Criação rápida de VM")]

## <a name="clean-up-deployment"></a>Limpar implantação 

Execute o comando a seguir para remover o grupo de recursos, a VM e todos os recursos relacionados.

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explicação sobre o script

Este script usa os comandos a seguir para criar um grupo de recursos, uma máquina virtual e todos os recursos relacionados. Cada comando na tabela redireciona para a documentação específica do comando.

| Command | Observações |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | Cria um grupo de recursos no qual todos os recursos são armazenados. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#create) | Cria a máquina virtual e a conecta a placa de rede, a rede virtual, a sub-rede e o NSG. Este comando também especifica a imagem de máquina virtual a ser usada e as credenciais administrativas.  |
| [az vm extension set](https://docs.microsoft.com/cli/azure/vm#create) | Adicione a Extensão de Script Personalizado à máquina virtual que invoca um script para instalar o IIS. |
| [az vm open-port](https://docs.microsoft.com/cli/azure/vm#open-port) | Cria uma regra de grupo de segurança de rede para permitir o tráfego de entrada. Neste exemplo, a porta 80 está aberta para tráfego HTTP. |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#set) | Exclui um grupo de recursos, incluindo todos os recursos aninhados. |

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre a CLI do Azure, veja a [documentação da CLI do Azure](https://docs.microsoft.com/cli/azure/overview).

Exemplos de script CLI máquinas virtuais adicionais podem ser encontrados na [documentação da VM do Windows do Azure](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
