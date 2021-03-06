---
title: "Script da CLI do Azure – obter uma cadeia de conexão do BD Cosmos do Azure para aplicativos MongoDB | Microsoft Docs"
description: "Exemplo de script da CLI do Azure – obter uma cadeia de conexão do BD Cosmos do Azure para aplicativos MongoDB"
services: cosmosdb
documentationcenter: cosmosdb
author: mimig1
manager: jhubbard
editor: 
tags: azure-service-management
ms.assetid: 
ms.service: cosmosdb
ms.custom: sample
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: cosmosdb
ms.workload: database
ms.date: 05/10/2017
ms.author: mimig
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 58e16cfc83c2057168428e8e85a6ebd143d2cd07
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017

---

# <a name="get-an-azure-cosmos-db-connection-string-for-mongodb-apps-using-the-azure-cli"></a>Obtenha uma cadeia de conexão do BD Cosmos do Azure para aplicativos MongoDB usando a CLI do Azure

Este exemplo obtém uma cadeia de conexão do BD Cosmos do Azure para aplicativos MongoDB usando a CLI do Azure. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Script de exemplo

[!code-azurecli[principal](../../../cli_scripts/cosmosdb/secure-cosmosdb-get-mongodb-connection-string/secure-cosmosdb-get-mongodb-connection-string.sh?highlight=36-39 "Obter cadeia de conexão do BD Cosmos do Azure para aplicativos MongoDB")]

## <a name="clean-up-deployment"></a>Limpar implantação

Após executar o exemplo de script, o comando a seguir pode ser usado para remover o grupo de recursos e todos os recursos associados a ele.

```azurecli
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicação sobre o script

Este script usa os seguintes comandos. Cada comando na tabela redireciona para a documentação específica do comando.

| Command | Observações |
|---|---|
| [az group create](/cli/azure/group#create) | Cria um grupo de recursos no qual todos os recursos são armazenados. |
| [az cosmosdb update](/cli/azure/cosmosdb/name#update) | Atualiza uma conta do BD Cosmos do Azure. |
| [az cosmosdb list-connection-strings](/cli/azure/cosmosdb/list-connection-strings) | Obtém a cadeia de conexão para a conta.|
| [az group delete](/cli/azure/resource#delete) | Exclui um grupo de recursos, incluindo todos os recursos aninhados. |

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre a CLI do Azure, veja a [documentação da CLI do Azure](https://docs.microsoft.com/cli/azure/overview).

Exemplos adicionais de scripts da CLI do BD Cosmos do Azure podem ser encontrados na [Documentação da CLI do BD Cosmos do Azure](../cli-samples.md).

