1. Em uma nova janela, entre no [portal do Azure](https://portal.azure.com/).
2. No menu à esquerda, clique em **Novo**, clique em **Bancos de Dados** e, em seguida, clique em **BD Cosmos do Azure**.
   
   ![Captura de tela do Portal do Azure, realçando Mais Serviços e BD Cosmos do Azure](./media/cosmosdb-create-dbaccount-mongodb/create-nosql-db-databases-json-tutorial-1.png)

3. Na folha **Nova conta**, especifique a configuração desejada para a conta do BD Cosmos do Azure. 

    Com o BD Cosmos do Azure, você pode escolher um dos quatro modelos de programação: Gremlin (gráfico), MongoDB, SQL (DocumentDB) e Tabela (chave-valor). 
       
    Neste início rápido, programaremos a API do MongoDB de modo que você escolherá **MongoDB** quando preencher o formulário. Mas se você tiver dados gráficos de um aplicativo de mídia social, dados de documento de um aplicativo de catálogo ou dados de chave/valor (tabela), perceba que o BD Cosmos do Azure poderá fornecer uma plataforma de serviço de banco de dados altamente disponível, distribuída globalmente para todos os aplicativos críticos.

    Preencha a nova folha de conta usando as informações na captura de tela como guia. Você escolherá valores exclusivos quando configurar sua conta, de modo que seus valores não corresponderão exatamente à captura de tela
 
    ![Captura de tela da folha Novo BD Cosmos do Azure](./media/cosmosdb-create-dbaccount-mongodb/create-nosql-db-databases-json-tutorial-2.png)
   
    Configuração|Valor sugerido|Descrição
    ---|---|---
    ID|*Valor exclusivo*|Um nome exclusivo que você escolhe para identificar a conta do BD Cosmos do Azure. *documents.Azure.com* é acrescentado à ID que você fornece para criar o URI, portanto, use uma ID exclusiva mas identificável. A ID pode conter somente letras minúsculas, números e o caractere '-', e deve ter entre 3 e 50 caracteres.
    AP|MongoDB|Programaremos para a [API do MongoDB](../articles/documentdb/documentdb-protocol-mongodb.md) posteriormente neste artigo.|
    Assinatura|*Sua assinatura*|A assinatura do Azure que você deseja usar para a conta do BD Cosmos do Azure. 
    Grupo de recursos|*O mesmo valor que a ID*|O novo nome de grupo de recursos para sua conta. Para simplificar, você pode usar um nome igual à sua ID. 
    Local|*A região mais próxima de seus usuários*|A localização geográfica na qual hospedar a sua conta do BD Cosmos do Azure. Escolha o local mais próximo dos usuários para fornecer a eles acesso mais rápido aos dados.

4. Clique em **Criar** para criar a conta.
5. Na barra de ferramentas, clique em **Notificações** para monitorar o processo de implantação.

    ![Notificação de implantação iniciada](./media/cosmosdb-create-dbaccount-mongodb/azure-documentdb-nosql-notification.png)

6.  Quando a implantação for concluída, abra a nova conta no bloco Todos os Recursos. 

    ![Conta do DocumentDB no bloco Todos os Recursos](./media/cosmosdb-create-dbaccount-mongodb/azure-documentdb-all-resources.png)