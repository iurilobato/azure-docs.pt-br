---
title: 'BD Cosmos do Azure: criar um aplicativo com o Node.js e com a API do DocumentDB | Microsoft Docs'
description: "Apresenta um exemplo de código Node.js que pode ser usado para se conectar e consultar a API do DocumentDB do BD Cosmos do Azure"
services: cosmosdb
documentationcenter: 
author: mimig1
manager: jhubbard
editor: 
ms.assetid: 9c0f033c-240e-4fee-8421-08907231087f
ms.service: cosmosdb
ms.custom: quick start connect
ms.workload: 
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: hero-article
ms.date: 05/10/2017
ms.author: mimig
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: deab76fadee7b825cb359d0cebf7f84b26fcf3a8
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017


---
# <a name="azure-cosmos-db-build-a-documentdb-api-app-with-nodejs-and-the-azure-portal"></a>BD Cosmos do Azure: criar um aplicativo de API do DocumentDB com o Node.js e com o portal do Azure

O BD Cosmos do Azure é o serviço multimodelo de banco de dados distribuído globalmente da Microsoft. É possível criar e consultar rapidamente documentos, chave/valor e bancos de dados do gráfico. Todos se beneficiam de recursos de escala horizontal e distribuição global no núcleo do BD Cosmos do Azure. 

Este início rápido demonstra como criar uma conta do BD Cosmos do Azure, um banco de dados de documento e uma coleção usando o Portal do Azure. Em seguida, você compila e executa um aplicativo de console criado na [API do Node.js do DocumentDB](../documentdb/documentdb-sdk-node.md).

## <a name="prerequisites"></a>Pré-requisitos

* Antes que possa executar esta amostra, você deverá ter os seguintes pré-requisitos:
    * [Node.js](https://nodejs.org/en/) versão v0.10.29 ou superior
    * [Git](http://git-scm.com/)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-database-account"></a>Crie uma conta de banco de dados

[!INCLUDE [documentdb-create-dbaccount](../../includes/documentdb-create-dbaccount.md)]

## <a name="add-a-collection"></a>Adicionar uma coleção

[!INCLUDE [cosmosdb-create-collection](../../includes/cosmosdb-create-collection.md)]

## <a name="clone-the-sample-application"></a>Clonar o aplicativo de exemplo

Agora, vamos clonar um aplicativo de API do DocumentDB do GitHub, defina a cadeia de conexão e execute-o. Você verá como é fácil trabalhar usando dados de forma programática. 

1. Abra uma janela de terminal do Git, como git bash, e `CD` para um diretório de trabalho.  

2. Execute o comando a seguir para clonar o repositório de exemplo. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-documentdb-nodejs-getting-started.git
    ```

## <a name="review-the-code"></a>Examine o código

Façamos uma rápida análise do que está acontecendo no aplicativo. Abra o arquivo `app.js` e veja que essas linhas de código criam os recursos do BD Cosmos do Azure. 

* O `documentClient` é inicializado.

    ```nodejs
    var client = new documentClient(config.endpoint, { "masterKey": config.primaryKey });
    ```

* Um novo banco de dados é criado.

    ```nodejs
    client.createDatabase(config.database, (err, created) => {
        if (err) reject(err)
        else resolve(created);
    });
    ```

* Uma nova coleção é criada.

    ```nodejs
    client.createCollection(databaseUrl, config.collection, { offerThroughput: 400 }, (err, created) => {
        if (err) reject(err)
        else resolve(created);
    });
    ```

* Alguns documentos são criados.

    ```nodejs
    client.createDocument(collectionUrl, document, (err, created) => {
        if (err) reject(err)
        else resolve(created);
    });
    ```

* Uma consulta SQL no JSON é executada.

    ```nodejs
    client.queryDocuments(
        collectionUrl,
        'SELECT VALUE r.children FROM root r WHERE r.lastName = "Andersen"'
    ).toArray((err, results) => {
        if (err) reject(err)
        else {
            for (var queryResult of results) {
                let resultString = JSON.stringify(queryResult);
                console.log(`\tQuery returned ${resultString}`);
            }
            console.log();
            resolve(results);
        }
    });
    ```    

## <a name="update-your-connection-string"></a>Atualizar sua cadeia de conexão

Agora, volte ao Portal do Azure para obter informações sobre a cadeia de conexão e copiá-las para o aplicativo.

1. No [Portal do Azure](http://portal.azure.com/), na sua conta do BD Cosmos do Azure, no painel de navegação esquerdo, clique em **Chaves** e, em seguida, clique em **Chaves de leitura/gravação**. Você usará os botões de cópia no lado direito da tela para copiar o URI e a Chave Primária para o arquivo `config.js` na próxima etapa.

    ![Exibir e copiar uma chave de acesso no Portal do Azure, folha Chaves](./media/create-documentdb-dotnet/keys.png)

2. Abra o arquivo `config.js`. 

3. Copie o valor do URI do portal (usando o botão de cópia) e transforme-o no valor da chave do ponto de extremidade em `config.js`. 

    `config.endpoint = "https://FILLME.documents.azure.com"`

4. Em seguida, copie o valor da CHAVE PRIMÁRIA do portal e transforme-o no valor de `config.primaryKey` em `config.js`. Agora, você atualizou o aplicativo com todas as informações necessárias para se comunicar com o BD Cosmos do Azure. 

    `config.primaryKey "FILLME"`
    
## <a name="run-the-app"></a>Execute o aplicativo
1. Executar `npm install` em um terminal para instalar os módulos npm necessários

2. Execute `node app.js` em um terminal para iniciar o aplicativo de nó.

Agora, é possível voltar ao Data Explorer e ver a consulta, modificar e trabalhar com esses novos dados. 

## <a name="review-slas-in-the-azure-portal"></a>Examinar SLAs no Portal do Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmosdb-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpar recursos

Se você não continuar usando este aplicativo, exclua todos os recursos criados por esse início rápido no portal do Azure com as seguintes etapas:

1. No menu à esquerda no Portal do Azure, clique em **Grupos de recursos** e depois clique no nome do recurso criado. 
2. Em sua página de grupo de recursos, clique em **Excluir**, digite o nome do recurso para excluir na caixa de texto e depois clique em **Excluir**.

## <a name="next-steps"></a>Próximas etapas

Neste início rápido, você aprendeu como criar uma conta do BD Cosmos do Azure, como criar uma coleção usando o Data Explorer e como executar um aplicativo. Agora, é possível importar outros dados para sua conta do BD Cosmos. 

> [!div class="nextstepaction"]
> [Importar dados no BD Cosmos do Azure](../documentdb/documentdb-import-data.md)



