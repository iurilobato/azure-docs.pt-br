---
title: Compilar um aplicativo Web Node.js e MongoDB no Azure | Microsoft Docs
description: "Saiba como fazer com que um aplicativo Node.js funcione no Azure, com conexão a um BD Cosmos com uma cadeia de conexão MongoDB."
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: 
ms.assetid: 0b4d7d0e-e984-49a1-a57a-3c0caa955f0e
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 05/04/2017
ms.author: cephalin
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 25fec75615d2376f3e566b509536eadd03590c0e
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017


---
# <a name="build-a-nodejs-and-mongodb-web-app-in-azure"></a>Compilar um aplicativo Web Node.js e MongoDB no Azure
Este tutorial mostra como criar um aplicativo Web Node.js no Azure e conectá-lo a um banco de dados MongoDB. Quando terminar, você terá um aplicativo MEAN (MongoDB, Express, AngularJS e Node.js) executando em [Aplicativos Web do Serviço de Aplicativo do Azure](app-service-web-overview.md). Para simplificar, o aplicativo de exemplo usa a [estrutura Web MEAN.js](http://meanjs.org/).

![Aplicativo MEAN.js em execução no Serviço de Aplicativo do Azure](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Neste tutorial, você aprenderá a:

> [!div class="checklist"]
> * Criar um banco de dados MongoDB no Azure
> * Conectar um aplicativo Node.js ao MongoDB
> * Implantar o aplicativo no Azure
> * Atualizar o modelo de dados e reimplantar o aplicativo
> * Transmitir logs de diagnóstico do Azure
> * Gerenciar o aplicativo no portal do Azure

## <a name="prerequisites"></a>Pré-requisitos

Antes de executar este exemplo, instale localmente os seguintes pré-requisitos:

1. [Baixe e instale o git](https://git-scm.com/)
1. [Baixe e instale o Node.js e NPM](https://nodejs.org/)
1. [Instale o Gulp.js](http://gulpjs.com/) (exigido pelo [MEAN.js](http://meanjs.org/docs/0.5.x/#getting-started))
1. [Baixe, instale e execute o MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/) 
1. [Baixe e instale o Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="test-local-mongodb"></a>Testar o MongoDB local
Nessa etapa, certifique-se de que seu banco de dados MongoDB local em execução.

Abra a janela do terminal e `cd` para o `bin` diretório da instalação do MongoDB. 

Execute `mongo` no terminal para se conectar ao servidor do MongoDB local.

```bash
mongo
```

Se sua conexão tiver êxito, então seu banco de dados MongoDB já está sendo executado. Caso contrário, certifique-se de que seu banco de dados MongoDB local foi iniciado seguindo as etapas em[Baixe, instale e execute o MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/). Com frequência, o MongoDB está instalado, mas você ainda precisa iniciá-lo executando `mongod`. 

Ao terminar o teste do seu banco de dados MongoDB, digite `Ctrl`+`C` no terminal. 

<a name="step2"></a>

## <a name="create-local-nodejs-app"></a>Criar aplicativo Node.js local
Nessa etapa, você configura o projeto Node.js local.

### <a name="clone-the-sample-application"></a>Clonar o aplicativo de exemplo

Abra a janela do terminal e `cd` para um diretório de trabalho.  

Execute os comandos a seguir para clonar o repositório de exemplo. 

```bash
git clone https://github.com/Azure-Samples/meanjs.git
```

Esse repositório de exemplo contém uma cópia do [repositório do MEAN.js](https://github.com/meanjs/mean). Ele é modificado minimamente para ser executado no Serviço de Aplicativo (para obter mais informações, consulte [LEIAME](https://github.com/Azure-Samples/meanjs/blob/master/README.md)).

### <a name="run-the-application"></a>Executar o aplicativo

Instale os pacotes necessários e inicie o aplicativo.

```bash
cd meanjs
npm install
npm start
```

Quando o aplicativo estiver totalmente carregado, você deverá ver algo semelhante à seguinte mensagem:

```
--
MEAN.JS - Development Environment

Environment:     development
Server:          http://0.0.0.0:3000
Database:        mongodb://localhost/mean-dev
App version:     0.5.0
MEAN.JS version: 0.5.0
--
```

Navegue até `http://localhost:3000` em um navegador. Clique em **Assinar** no menu superior e tente criar um usuário fictício. 

O aplicativo de exemplo MEAN.js armazena dados do usuário no banco de dados. Se você tiver êxito e o MEAN.js entrar automaticamente no usuário criado, significa que seu aplicativo está gravando dados no banco de dados MongoDB local.

![O MEAN.js conecta-se com êxito ao MongoDB](./media/app-service-web-tutorial-nodejs-mongodb-app/mongodb-connect-success.png)

Tente clicar em **Admin** > **Gerenciar Artigos** para adicionar alguns artigos.

Para parar Node.js a qualquer momento, digite `Ctrl`+`C` no terminal. 

## <a name="create-production-mongodb"></a>Criar MongoDB de produção

Ness etapa, você cria um banco de dados MongoDB no Azure. Quando seu aplicativo é implantado no Azure, ele usa esse banco de dados para sua carga de trabalho de produção.

Para o MongoDB, este tutorial usa o [BD Cosmos do Azure](/azure/documentdb/). O BD Cosmos dá suporte a conexões de cliente do MongoDB.

### <a name="log-in-to-azure"></a>Fazer logon no Azure

Agora, você vai usar o Azure CLI 2.0 em uma janela de terminal para criar os recursos necessários para hospedar seu aplicativo Node.js no Serviço de Aplicativo do Azure .  Faça logon na sua assinatura do Azure com o comando [az login](/cli/azure/#login) e siga as instruções na tela. 

```azurecli 
az login 
``` 
   
### <a name="create-a-resource-group"></a>Criar um grupo de recursos

Crie um [grupo de recursos](../azure-resource-manager/resource-group-overview.md) com o [az group create](/cli/azure/group#create). Um grupo de recursos do Azure é um contêiner lógico no qual recursos do Azure, como aplicativos da Web, bancos de dados e contas de armazenamento são implantados e gerenciados. 

O exemplo a seguir cria um grupo de recursos na região da Europa Ocidental.

```azurecli
az group create --name myResourceGroup --location "West Europe"
```

Para ver quais são os valores possíveis para `--location`, use o comando `az appservice list-locations` da CLI do Azure.

### <a name="create-a-cosmos-db-account"></a>Criar uma conta do BD Cosmos

Crie uma conta do BD Cosmos usando o comando [az cosmosdb create](/cli/azure/cosmosdb#create).

No comando a seguir, substitua por seu próprio nome do BD Cosmos exclusivo quando ver o espaço reservado _&lt;cosmosdb_name>_. Esse nome exclusivo é usado como parte de seu ponto de extremidade do BD Cosmos, `https://<cosmosdb_name>.documents.azure.com/`, portanto, o nome precisa ser exclusivo em todas as contas do BD Cosmos no Azure. 

```azurecli
az cosmosdb create \
    --name <cosmosdb_name> \
    --resource-group myResourceGroup \
    --kind MongoDB
```

O parâmetro `--kind MongoDB` habilita conexões do cliente MongoDB.

> [!NOTE]
> _&lt;cosmosdb_name>_ deve conter somente letras minúsculas, números e o caractere _-_, e deve ter entre 3 e 50 caracteres.
>
>

Quando a conta do BD Cosmos é criada, a CLI do Azure mostra informações semelhantes ao exemplo a seguir:

```json
{
  "consistencyPolicy":
  {
    "defaultConsistencyLevel": "Session",
    "maxIntervalInSeconds": 5,
    "maxStalenessPrefix": 100
  },
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb_name>.documents.azure.com:443/",
  "failoverPolicies": 
  ...
  < Output has been truncated for readability >
}
```

## <a name="connect-app-to-production-mongodb"></a>Conectar o aplicativo ao MongoDB de produção

Nesta etapa, você conecta o aplicativo de exemplo MEAN.js ao BD Cosmos que acabou de ser criado usando uma cadeia de conexão do MongoDB. 

### <a name="retrieve-the-database-key"></a>Recuperar a chave do banco de dados

Para se conectar ao BD Cosmos, você precisa da chave do banco de dados. Use o comando [az cosmosdb list-keys](/cli/azure/cosmosdb#list-keys) para recuperar a chave primária.

```azurecli
az cosmosdb list-keys \
    --name <cosmosdb_name> \
    --resource-group myResourceGroup
```

A CLI do Azure emite informações semelhantes ao seguinte exemplo:

```json
{
  "primaryMasterKey": "RS4CmUwzGRASJPMoc0kiEvdnKmxyRILC9BWisAYh3Hq4zBYKr0XQiSE4pqx3UchBeO4QRCzUt1i7w0rOkitoJw==",
  "primaryReadonlyMasterKey": "HvitsjIYz8TwRmIuPEUAALRwqgKOzJUjW22wPL2U8zoMVhGvregBkBk9LdMTxqBgDETSq7obbwZtdeFY7hElTg==",
  "secondaryMasterKey": "Lu9aeZTiXU4PjuuyGBbvS1N9IRG3oegIrIh95U6VOstf9bJiiIpw3IfwSUgQWSEYM3VeEyrhHJ4rn3Ci0vuFqA==",
  "secondaryReadonlyMasterKey": "LpsCicpVZqHRy7qbMgrzbRKjbYCwCKPQRl0QpgReAOxMcggTvxJFA94fTi0oQ7xtxpftTJcXkjTirQ0pT7QFrQ=="
}
```

Copie o valor de `primaryMasterKey` para um editor de texto. Essas informações serão necessárias na próxima etapa.

<a name="devconfig"></a>
### <a name="configure-the-connection-string-in-your-nodejs-application"></a>Configurar a cadeia de conexão em seu aplicativo Node.js

Em seu repositório do MEAN.js, abra _config/env/production.js_.

No objeto `db`, substitua o valor de `uri` conforme mostrado no exemplo a seguir. Certifique-se também de substituir os dois espaços reservados _&lt;cosmosdb_name>_ por seu nome do BD Cosmos e o espaço reservado _&lt;primary_master_key>_ pela chave copiada na etapa anterior.

```javascript
db: {
  uri: 'mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false',
  ...
},
```

> [!NOTE] 
> A opção `ssl=true` é importante porque o [BD Cosmos requer SSL](../documentdb/documentdb-connect-mongodb-account.md#connection-string-requirements). 
>
>

Salve suas alterações.

### <a name="test-the-application-in-production-mode"></a>Testar o aplicativo em modo de produção 

Assim como acontece com outras estruturas Web Node.js, MEAN.js usa `gulp` para reduzir e agrupar scripts para o ambiente de produção. Isso gera os arquivos necessários para o ambiente de produção. 

Executar `gulp prod` agora.

```bash
gulp prod
```

Em seguida, execute o seguinte comando para usar a cadeia de conexão configurada em _config/env/production.js_.

```bash
NODE_ENV=production node server.js
```

`NODE_ENV=production` define a variável de ambiente que informa ao Node.js para ser executado no ambiente de produção e `node server.js` inicia o servidor Node.js com `server.js` na sua raiz de repositório. É dessa forma que o aplicativo Node.js é carregado no Azure. 

Quando o aplicativo for carregado, verifique se ele está sendo executado em produção:

```
--
MEAN.JS

Environment:     production
Server:          http://0.0.0.0:8443
Database:        mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false
App version:     0.5.0
MEAN.JS version: 0.5.0
```

Navegue até `http://localhost:8443` em um navegador. Clique em **Assinar** no menu superior e tente criar um usuário fictício exatamente como antes. Se você tiver êxito, seu aplicativo estará gravando dados no BD Cosmos no Azure. 

No terminal, pare Node.js digitando `Ctrl`+`C`. 

## <a name="deploy-app-to-azure"></a>Implantar o aplicativo no Azure
Nessa etapa, você implanta seu aplicativo Node.js conectado ao MongoDB no Serviço de Aplicativo do Azure.

### <a name="create-an-app-service-plan"></a>Criar um plano de Serviço de Aplicativo

Criar um plano do Serviço de Aplicativo com o comando [az appservice plan create](/cli/azure/appservice/plan#create). 

> [!NOTE] 
> Um plano do Serviço de Aplicativo representa a coleção de recursos físicos usados para hospedar seus aplicativos. Todos os aplicativos atribuídos a um plano do Serviço de Aplicativo compartilham os recursos definidos por ele, permitindo que você economize ao hospedar vários aplicativos. 
> 
> Os Planos do Serviço de Aplicativo definem: 
> 
> * Região (Europa Setentrional, Leste dos EUA, Sudeste Asiático) 
> * Tamanha da Instância (Pequena, Média, Grande) 
> * Contagem da Escala (uma, duas ou três instâncias etc.) 
> * SKU (Gratuito, Compartilhado, Básico, Standard, Premium) 
> 

O exemplo a seguir cria um Plano do Serviço de Aplicativo chamado _myAppServicePlan_ usando o tipo de preço **GRATUITO**:

```azurecli
az appservice plan create \
    --name myAppServicePlan \
    --resource-group myResourceGroup \
    --sku FREE
```

Quando o Plano do Serviço de Aplicativo é criado, a CLI do Azure mostra informações semelhantes ao exemplo a seguir:

```json 
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "North Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan", 
  "kind": "app",
  "location": "North Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  ...
  < Output has been truncated for readability >
} 
```

### <a name="create-a-web-app"></a>Criar um aplicativo Web

Agora que um Plano do Serviço de Aplicativo foi criado, crie um aplicativo Web dentro do Plano do Serviço de Aplicativo _myAppServicePlan_. O aplicativo Web fornece um espaço de hospedagem para implantar seu código e fornecer uma URL para exibir o aplicativo implantado. Use o comando [az appservice web create](/cli/azure/appservice/web#create) para criar o aplicativo Web. 

No seguinte comando, substitua o espaço reservado _&lt;app_name>_ pelo seu próprio nome de aplicativo exclusivo. Esse nome exclusivo é usado como a parte do nome de domínio padrão para o aplicativo Web, portanto, o nome precisa ser exclusivo entre todos os aplicativos no Azure. Posteriormente, você poderá mapear qualquer entrada DNS personalizada para o aplicativo Web antes de expor para seus usuários. 

```azurecli
az appservice web create \
    --name <app_name> \
    --resource-group myResourceGroup \
    --plan myAppServicePlan
```

Quando o aplicativo Web tiver sido criado, a CLI do Azure mostrará informações semelhantes ao exemplo a seguir: 

```json 
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  ...
  < Output has been truncated for readability >
}
```

### <a name="configure-an-environment-variable"></a>Configurar um variável de ambiente

Anteriormente no tutorial, você codificou a cadeia de conexão de banco de dados em _config/env/production.js_. De acordo com as melhores práticas de segurança, você deseja manter esses dados confidenciais fora do seu repositório Git. Para seu aplicativo em execução no Azure, você usará uma variável de ambiente.

No Serviço de Aplicativo, são definidas as variáveis de ambiente como _configurações do aplicativo_ usando o comando [az appservice web config appsettings update](/cli/azure/appservice/web/config/appsettings#update). 

O exemplo a seguir permite configurar uma configuração de aplicativo `MONGODB_URI` no seu aplicativo Web do Azure. Certifique-se de substituir os espaços reservados como antes.

```azurecli
az appservice web config appsettings update \
    --name <app_name> \
    --resource-group myResourceGroup \
    --settings MONGODB_URI="mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true"
```

No código Node.js, você acessa essa configuração de aplicativo com `process.env.MONGODB_URI`, assim como você teria acesso a qualquer variável de ambiente. 

Agora, desfaça as alterações em _config/env/production.js_ com o seguinte comando:

```bash
git checkout -- .
```

Abra _config/env/production.js_ novamente. Observe que o aplicativo MEAN.js padrão já está configurado para usar a variável de ambiente `MONGODB_URI` que você acabou de criar.

```javascript
db: {
  uri: ... || process.env.MONGODB_URI || ...,
  ...
},
```

### <a name="configure-local-git-deployment"></a>Configurar a implantação do git local 

É possível implantar seu aplicativo no Serviço de Aplicativo do Azure de várias maneiras, incluindo FTP, Git local, GitHub, Visual Studio Team Services e BitBucket. Para o FTP e o Git local, é necessário ter um usuário de implantação configurado no servidor para autenticar sua implantação. 

Use o comando [az appservice web deployment user set](/cli/azure/appservice/web/deployment/user#set) para criar suas credenciais no nível da conta. 

> [!NOTE] 
> Um usuário de implantação é necessário para a implantação do FTP e do Git Local em um Serviço de Aplicativo. Esse usuário de implantação é de nível de conta. Sendo assim, é diferente da sua conta de assinatura do Azure. É necessário configurar esse usuário de implantação somente uma vez.

```azurecli
az appservice web deployment user set \
    --user-name <specify-a-username> \
    --password <minimum-8-char-capital-lowercase-number>
```

Use o comando [az appservice web source-control config-local-git](/cli/azure/appservice/web/source-control#config-local-git) para configurar o acesso do Git local para o aplicativo Web do Azure. 

```azurecli
az appservice web source-control config-local-git \
    --name <app_name> \
    --resource-group myResourceGroup
```

Quando o usuário de implantação é configurado, a CLI do Azure mostra a URL de implantação ao seu aplicativo Web Azure no seguinte formato:

```bash 
https://<username>@<app_name>.scm.azurewebsites.net:443/<app_name>.git 
``` 

Copie a saída do terminal, pois ela será usada na próxima etapa. 

### <a name="push-to-azure-from-git"></a>Enviar do Git para o Azure

Adicione um Azure remoto ao seu repositório Git local. 

```bash
git remote add azure <paste_copied_url_here> 
```

Envie para o Azure remoto para implantar seu aplicativo Node.js. Será solicitada a senha que você forneceu anteriormente como parte da criação do usuário de implantação. 

```bash
git push azure master
```

Durante a implantação, o Serviço de Aplicativo do Azure comunica seu andamento com o Git.

```bash
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6c7c716eee'.
remote: Running custom deployment command...
remote: Running deployment command...
remote: Handling node.js deployment.
.
.
.
remote: Deployment successful.
To https://<app_name>.scm.azurewebsites.net/<app_name>.git
 * [new branch]      master -> master
``` 

> [!NOTE]
> É possível notar que o processo de implantação executa [Gulp](http://gulpjs.com/) após `npm install`. O Serviço de Aplicativo não executa tarefas Gulp ou Grunt durante a implantação, portanto, esse repositório de exemplo possui dois arquivos adicionais em seu diretório raiz para habilitá-lo: 
>
> - _.deployment_ – este arquivo instrui o Serviço de Aplicativo a executar `bash deploy.sh` como o script de implantação personalizado.
> - _deploy.sh_ – o script de implantação personalizado. Se você revisar o arquivo, verá que ele executa `gulp prod` após `npm install` e `bower install`. 
>
> É possível usar essa abordagem para adicionar qualquer etapa à implantação com base em Git.
>
> Se você reiniciar seu aplicativo Web do Azure em qualquer ponto, o Serviço de Aplicativo não executará novamente essas tarefas de automação.
>
>

### <a name="browse-to-the-azure-web-app"></a>Navegar até o aplicativo Web do Azure 
Navegue até o aplicativo Web implantado usando o navegador da Web. 

```bash 
http://<app_name>.azurewebsites.net 
``` 

Clique em **Assinar** no menu superior e tente criar um usuário fictício. 

Se você tiver êxito e o aplicativo entrar automaticamente no usuário criado, isso significa que seu aplicativo MEAN.js no Azure tem conectividade com o banco de dados do MongoDB (BD Cosmos). 

![Aplicativo MEAN.js em execução no Serviço de Aplicativo do Azure](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Tente clicar em **Admin** > **Gerenciar Artigos** para adicionar alguns artigos. 

**Parabéns!** Você está executando um aplicativo Node.js controlado por dados no Serviço de Aplicativo do Azure.

## <a name="update-data-model-and-redeploy"></a>Atualizar o modelo de dados e reimplantar

Nessa etapa, você faz algumas alterações no modelo de dados `article` e publica as alterações no Azure.

### <a name="update-the-data-model"></a>Atualizar o modelo de dados

Abra _modules/articles/server/models/article.server.model.js_.

Em `ArticleSchema`, adicione um tipo `String` chamado `comment`. Quando terminar, seu código de esquema deverá ter essa aparência:

```javascript
var ArticleSchema = new Schema({
  ...,
  user: {
    type: Schema.ObjectId,
    ref: 'User'
  },
  comment: {
    type: String,
    default: '',
    trim: true
  }
});
```

Você concluiu as alterações do modelo. 

### <a name="update-the-articles-code"></a>Atualizar o código dos artigos

Em seguida, atualize o resto do código `articles` para usar `comment`.

No geral, há cinco (5) arquivos que você precisa modificar, o controlador de servidor e as quatro exibições do cliente. 

Primeiro, abra _modules/articles/server/controllers/articles.server.controller.js_.

Na função `update` adicione uma atribuição para `article.comment`. Quando terminar, a função `update` deverá ter esta aparência:

```javascript
exports.update = function (req, res) {
  var article = req.article;

  article.title = req.body.title;
  article.content = req.body.content;
  article.comment = req.body.comment;

  ...
};
```

Em seguida, abra _modules/articles/client/views/view-article.client.view.html_.

Logo acima da marca de fechamento `</section>`, adicione a seguinte linha para exibir `comment` juntamente com o resto dos dados do artigo:

```HTML
<p class="lead" ng-bind="vm.article.comment"></p>
```

Depois, abra _modules/articles/client/views/list-articles.client.view.html_.

Logo acima da marca de fechamento `</a>`, adicione a seguinte linha para exibir `comment` juntamente com o resto dos dados do artigo:

```HTML
<p class="list-group-item-text" ng-bind="article.comment"></p>
```

Em seguida, abra _modules/articles/client/views/admin/list-articles.client.view.html_.

Dentro da marca `<div class="list-group">` e logo acima da marca de fechamento `</a>`, adicione a seguinte linha para exibir `comment` juntamente com o resto dos dados do artigo:

```HTML
<p class="list-group-item-text" data-ng-bind="article.comment"></p>
```

Por fim, abra _modules/articles/client/views/admin/form-article.client.view.html_.

Localize a marca `<div class="form-group">` que contém o botão enviar, que se parece com isto:

```HTML
<div class="form-group">
  <button type="submit" class="btn btn-default">{{vm.article._id ? 'Update' : 'Create'}}</button>
</div>
```

Logo acima dessa marca, adicione outra marca `<div class="form-group">` que permite editar o campo `comment`. Sua nova marca deve ter esta aparência:

```HTML
<div class="form-group">
  <label class="control-label" for="comment">Comment</label>
  <textarea name="comment" data-ng-model="vm.article.comment" id="comment" class="form-control" cols="30" rows="10" placeholder="Comment"></textarea>
</div>
```

### <a name="test-your-changes-locally"></a>Testar suas alterações localmente

Salve todas as alterações.

Teste suas alterações em modo de produção novamente.

```bash
gulp prod
NODE_ENV=production node server.js
```

> [!NOTE]
> Lembre-se de que _config/env/production.js_ foi revertido e a variável de ambiente `MONGODB_URI` está definida apenas no seu aplicativo Web do Azure e não no computador local. Se analisar o arquivo de configuração você verá que a configuração de produção é definida por padrão para usar um banco de dados MongoDB local. Isso garante seus dados de produção não sejam tocados ao testar suas alterações de código localmente.
>
>

Navegue até `http://localhost:8443` em um navegador e certifique-se de que está conectado.

Clique em **Admin** > **Gerenciar Artigos** e, em seguida, adicione um artigo clicando no botão **+**.

Agora, você deverá ver a nova caixa de texto `Comment`.

![Campo de comentário adicionado para Artigos](./media/app-service-web-tutorial-nodejs-mongodb-app/added-comment-field.png)

No terminal, pare Node.js digitando `Ctrl`+`C`. 

### <a name="publish-changes-to-azure"></a>Publicar alterações no Azure

Confirme suas alterações no git, em seguida, envie as alterações do código para o Azure.

```bash
git commit -am "added article comment"
git push azure master
```

Quando `git push` estiver completo, navegue até seu aplicativo Web do Azure e tente a nova funcionalidade novamente.

![Alterações de banco de dados e modelos publicadas no Azure](media/app-service-web-tutorial-nodejs-mongodb-app/added-comment-field-published.png)

> [!NOTE]
> Se você adicionou artigos anteriores, ainda será possível vê-los. Dados existentes no BD Cosmos não são perdidos. Também, suas atualizações para o esquema de dados e deixa seus dados existentes intactos.
>
>

## <a name="stream-diagnostic-logs"></a>Logs de diagnóstico de fluxo 

Enquanto o aplicativo Node.js executa no Serviço de Aplicativo do Azure, você pode obter logs do console transferidos diretamente para o seu terminal. Dessa forma, é possível obter as mesmas mensagens de diagnóstico para ajudá-lo a depurar erros de aplicativo.

Para iniciar o streaming de log, use o comando [az appservice web log tail](/cli/azure/appservice/web/log#tail).

```azurecli 
az appservice web log tail \
    --name <app_name> \
    --resource-group myResourceGroup 
``` 

Uma vez iniciado o streaming de log, atualize seu aplicativo Web do Azure no navegador para obter algum tráfego da Web. Agora, deve ser possível ver os logs do console transferidos para seu terminal.

Para interromper o streaming de log a qualquer momento, digite `Ctrl`+`C`. 

## <a name="manage-your-azure-web-app"></a>Gerenciar seu aplicativo Web do Azure

Vá para o portal do Azure para ver o aplicativo Web que você criou.

Para fazer isso, entre em [https://portal.azure.com](https://portal.azure.com).

No menu à esquerda, clique em **Serviço de Aplicativo**, em seguida, clique no nome do seu aplicativo Web do Azure.

![Navegação do portal para o aplicativo Web do Azure](./media/app-service-web-tutorial-nodejs-mongodb-app/access-portal.png)

Você foi para a _folha_ de seu aplicativo Web (uma página do portal que abre horizontalmente).

Por padrão, a folha de seu aplicativo Web mostra a página **Visão Geral**. Esta página fornece uma visão de como está seu aplicativo. Aqui, você também pode executar tarefas básicas de gerenciamento como procurar, parar, iniciar, reiniciar e excluir. As guias no lado esquerdo da folha mostram as páginas de configuração diferentes que você pode abrir.

![Folha Serviço de Aplicativo no portal do Azure](./media/app-service-web-tutorial-nodejs-mongodb-app/web-app-blade.png)

Essas guias na folha mostram muitos recursos excelentes que você pode adicionar ao seu aplicativo Web. A lista a seguir fornece algumas possibilidades:

* mapear um nome DNS personalizado
* associar um certificado SSL personalizado
* configurar uma implantação contínua
* Escalar vertical e horizontalmente
* adicionar a autenticação do usuário

## <a name="clean-up-resources"></a>Limpar recursos
 
Se não precisar desses recursos para outro tutorial (consulte [Próximas etapas](#next)), você poderá excluí-los executando o seguinte comando: 
  
```azurecli 
az group delete --name myResourceGroup 
``` 

<a name="next"></a>

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você aprendeu a:

> [!div class="checklist"]
> * Criar um banco de dados MongoDB no Azure
> * Conectar um aplicativo Node.js ao MongoDB
> * Implantar o aplicativo no Azure
> * Atualizar o modelo de dados e reimplantar o aplicativo
> * Transmitir logs do Azure para seu terminal
> * Gerenciar o aplicativo no portal do Azure

Vá para o próximo tutorial para aprender a mapear um nome DNS personalizado para ele.

> [!div class="nextstepaction"] 
> [Mapear um nome DNS personalizado existente para aplicativos Web do Azure](app-service-web-tutorial-custom-domain.md)

