---
title: "Criar sua primeira função na CLI do Azure | Microsoft Docs"
description: "Aprenda a criar sua primeira Função do Azure para a execução sem servidor usando a CLI do Azure."
services: functions
keywords: 
author: ggailey777
ms.author: glenga
ms.assetid: 674a01a7-fd34-4775-8b69-893182742ae0
ms.date: 05/02/2017
ms.topic: hero-article
ms.service: functions
ms.devlang: azure-cli
manager: erikre
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: ef9d65ae5ad0792230e1b8c0d7ed123c129f0f59
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017

---

# <a name="create-your-first-function-using-the-azure-cli"></a>Criar sua primeira função usando a CLI do Azure

Este tutorial de início rápido explica como usar o Azure Functions para criar sua primeira função. É possível usar a CLI do Azure para criar um aplicativo de funções, que é a infraestrutura sem servidor que hospeda sua função. O código de função em si é implantado de um repositório de exemplo do GitHub.    

Você pode seguir as etapas abaixo usando um computador Mac, Windows ou Linux. Deve levar apenas cinco minutos, aproximadamente, para concluir todas as etapas neste tópico.

## <a name="prerequisites"></a>Pré-requisitos 

Antes de executar este exemplo, você deve ter o seguinte:

+ Uma conta do [GitHub](https://github.com) ativa. 
+ A [CLI do Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) instalada. 
+ Uma assinatura ativa do Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="log-in-to-azure"></a>Fazer logon no Azure

Entre na sua assinatura do Azure usando o comando [az login](/cli/azure/#login) e siga as instruções na tela. 

```azurecli
az login
```

## <a name="create-a-resource-group"></a>Criar um grupo de recursos

Crie um grupo de recursos com [az group create](/cli/azure/group#create). Um grupo de recursos do Azure é um contêiner lógico no qual os recursos do Azure, como os aplicativos de funções, bancos de dados e contas de armazenamento, são implantados e gerenciados.

O seguinte exemplo cria um grupo de recursos chamado `myResourceGroup`:

```azurecli
az group create --name myResourceGroup --location westeurope
```
## <a name="create-an-azure-storage-account"></a>Criar uma conta de Armazenamento do Azure

O Functions usa uma conta de Armazenamento do Azure para manter o estado e outras informações sobre suas funções. Crie uma conta de armazenamento no grupo de recursos que você criou ao utilizar o comando [az storage account create](/cli/azure/storage/account#create).

No seguinte comando, substitua seu próprio nome da conta de armazenamento globalmente exclusivo quando vir o espaço reservado `<storage_name>`. Os nomes da conta de armazenamento devem ter entre 3 e 24 caracteres e podem conter apenas números e letras minúsculas.

```azurecli
az storage account create --name <storage_name> --location westeurope --resource-group myResourceGroup --sku Standard_LRS
```

Depois que a conta de armazenamento for criada, a CLI do Azure mostrará informações semelhantes ao exemplo a seguir:

```json
{
  "creationTime": "2017-04-15T17:14:39.320307+00:00",
  "id": "/subscriptions/bbbef702-e769-477b-9f16-bc4d3aa97387/resourceGroups/myresourcegroup/...",
  "kind": "Storage",
  "location": "westeurope",
  "name": "myfunctionappstorage",
  "primaryEndpoints": {
    "blob": "https://myfunctionappstorage.blob.core.windows.net/",
    "file": "https://myfunctionappstorage.file.core.windows.net/",
    "queue": "https://myfunctionappstorage.queue.core.windows.net/",
    "table": "https://myfunctionappstorage.table.core.windows.net/"
  },
     ....
    // Remaining output has been truncated for readability.
}
```

## <a name="create-a-function-app"></a>Criar um aplicativo de funções

Você deve ter um aplicativo de funções para hospedar a execução de suas funções. O aplicativo de funções fornece um ambiente para execução sem servidor do seu código de função. Ele permite que você agrupe funções como uma unidade lógica para facilitar o gerenciamento, a implantação e o compartilhamento de recursos. Crie um aplicativo de funções ao usar o comando [az functionapp create](/cli/azure/functionapp#create). 

No seguinte comando, substitua seu próprio nome de aplicativo de funções exclusivo quando vir o espaço reservado `<app_name>` e o nome da conta de armazenamento para `<storage_name>`. O `<app_name>` é usado como domínio DNS padrão para o aplicativo de funções, portanto, o nome deve ser exclusivo entre todos os aplicativos no Azure. 

```azurecli
az functionapp create --name <app_name> --storage-account  <storage_name>  --resource-group myResourceGroup --consumption-plan-location westeurope
```
Por padrão, um aplicativo de funções é criado com o Plano de hospedagem de consumo, o que significa que os recursos são adicionados dinamicamente, conforme exigido por suas funções e você paga apenas quando funções estão em execução. Para obter mais informações, consulte [Escolher o plano de hospedagem correto](functions-scale.md). 

Depois que o aplicativo de funções for criado, a CLI do Azure mostrará informações semelhantes ao exemplo a seguir:

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "containerSize": 1536,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "quickstart.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "quickstart.azurewebsites.net",
    "quickstart.scm.azurewebsites.net"
  ],
   ....
    // Remaining output has been truncated for readability.
}
```

Agora que você tem um aplicativo de funções, é possível implantar o código de função real do repositório GitHub de exemplo.

## <a name="deploy-your-function-code"></a>Implantar o código de função  

Há várias maneiras de criar o código de função no novo aplicativo de funções. Este tópico se conecta a um repositório de exemplo no GitHub. Assim como anteriormente, no código a seguir, substitua o espaço reservado `<app_name>` pelo nome do aplicativo de funções que você criou. 

```azurecli
az functionapp deployment source config --name <app_name> --resource-group myResourceGroup --repo-url https://github.com/Azure-Samples/functions-quickstart --branch master --manual-integration
```
Após a definição de fonte de implantação, a CLI do Azure mostra informações semelhantes ao exemplo a seguir (valores nulos removidos para facilitar a leitura):

```json
{
  "branch": "master",
  "deploymentRollbackEnabled": false,
  "id": "/subscriptions/bbbef702-e769-477b-9f16-bc4d3aa97387/resourceGroups/myResourceGroup/...",
  "isManualIntegration": true,
  "isMercurial": false,
  "location": "West Europe",
  "name": "quickstart",
  "repoUrl": "https://github.com/Azure-Samples/functions-quickstart",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.Web/sites/sourcecontrols"
}
```

## <a name="test-the-function"></a>Testar a função

Use o cURL para testar a função implantada em um computador Mac ou Linux, ou usando Bash no Windows. Execute o seguinte comando cURL, substituindo o espaço reservado `<app_name>` pelo nome do aplicativo de funções. Acrescente a cadeia de caracteres de consulta `&name=<yourname>` à URL.

```bash
curl http://<app_name>.azurewebsites.net/api/HttpTriggerJS1?name=<yourname>
```  

![Resposta de função mostrada em um navegador.](./media/functions-create-first-azure-function-azure-cli/functions-azure-cli-function-test-curl.png)  

Caso não tenha o cURL disponível na linha de comando, basta digitar a mesma URL no endereço do seu navegador da Web. Novamente, substitua o espaço reservado `<app_name>` pelo nome do aplicativo de funções e acrescente a cadeia de caracteres de consulta `&name=<yourname>` à URL e execute a solicitação. 

    http://<app_name>.azurewebsites.net/api/HttpTriggerJS1?name=<yourname>
   
![Resposta de função mostrada em um navegador.](./media/functions-create-first-azure-function-azure-cli/functions-azure-cli-function-test-browser.png)  

## <a name="clean-up-resources"></a>Limpar recursos

Outros inícios rápidos nessa coleção aproveitam esse início rápido. Se você planeja continuar trabalhando com inícios rápidos subsequentes ou com os tutoriais, não limpe os recursos criados nesse início rápido. Caso contrário, use os comandos a seguir para excluir todos os recursos criados por esse início rápido:

```azurecli
az group delete --name myResourceGroup
```
Quando solicitado, digite `y`.

## <a name="next-steps"></a>Próximas etapas

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]

