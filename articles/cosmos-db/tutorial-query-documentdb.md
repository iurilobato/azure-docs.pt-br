---
title: Como consultar com SQL no Azure Cosmos DB? | Microsoft Docs
description: Aprenda a consulta com os dados do DocumentDB com o SQL no Azure Cosmos DB
services: cosmosdb
documentationcenter: 
author: mimig1
manager: jhubbard
editor: 
tags: 
ms.assetid: 
ms.service: cosmosdb
ms.custom: tutorial-develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: 
ms.date: 05/10/2017
ms.author: mimig
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: dd34ff43e78175b0d6a6e38bbd1303070f6549ab
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017


---

# <a name="azure-cosmos-db-how-to-query-using-sql"></a>Azure Cosmos DB: Como consultar usando SQL?

A [API do DocumentDB](../documentdb/documentdb-introduction.md) do Azure Cosmos DB oferece suporte à consulta de documentos usando SQL. Este artigo fornece um exemplo de documento e dois exemplos de consultas SQL e resultados.

Este artigo aborda as seguintes tarefas: 

> [!div class="checklist"]
> * Consultar dados com SQL

## <a name="sample-document"></a>Exemplo de documento

As consultas de SQL neste artigo usam o seguinte exemplo de documento.

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam", 
        "givenName": "Jesse", 
        "gender": "female", "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      { 
        "familyName": "Miller", 
         "givenName": "Lisa", 
         "gender": "female", 
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```
## <a name="where-can-i-run-sql-queries"></a>Onde é possível executar consultas SQL?

Você pode executar consultas usando o Data Explorer no Portal do Azure, por meio da [API REST e SDKs](../documentdb/documentdb-query-collections-query-explorer.md) e até mesmo o [Playground de consultas](https://www.documentdb.com/sql/demo), que executa consultas em um conjunto existente de dados de exemplo.

Para saber mais sobre consultas SQL, confira:
* [Consulta e sintaxe SQL](../documentdb/documentdb-sql-query.md)

## <a name="prerequisites"></a>Pré-requisitos

Este tutorial presume que você tem uma conta e uma coleção do Azure Cosmos DB. Não tenho nenhum deles? Complete o [Guia de início rápido de cinco minutos](create-mongodb-nodejs.md) ou o [tutorial de desenvolvedor](tutorial-develop-mongodb.md) para criar uma conta e uma coleção.

## <a name="example-query-1"></a>Exemplo de consulta 1

Com base no exemplo de documento de família acima, a consulta SQL a seguir retorna os documentos cujo campo de id corresponde a `WakefieldFamily`. Por se tratar de uma instrução `SELECT *`, a saída da consulta será todo o documento JSON:

**Consulta**

    SELECT * 
    FROM Families f 
    WHERE f.id = "WakefieldFamily"

**Resultados**

    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]

## <a name="example-query-2"></a>Exemplo de consulta 2

A próxima consulta retorna todos os nomes dos filhos na família cuja identificação corresponde ao `WakefieldFamily` organizado por nota.

**Consulta**

    SELECT c.givenName 
    FROM Families f 
    JOIN c IN f.children 
    WHERE f.id = 'WakefieldFamily'
    ORDER BY f.children.grade ASC

**Resultados**

    [
      { "givenName": "Jesse" }, 
      { "givenName": "Lisa"}
    ]


## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você fez o seguinte:

> [!div class="checklist"]
> * Aprendeu a consultar usando SQL  

Agora você pode prosseguir para o próximo tutorial e aprender a distribuir seus dados globalmente.

> [!div class="nextstepaction"]
> [Distribuir os dados globalmente](tutorial-global-distribution-documentdb.md)


