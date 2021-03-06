---
title: "Guia de início rápido do cluster Kubernetes no Azure | Microsoft Docs"
description: "Implantar e começar a usar um cluster Kubernetes no Serviço de Contêiner do Azure"
services: container-service
documentationcenter: 
author: anhowe
manager: timlt
editor: 
tags: acs, azure-container-service, kubernetes
keywords: 
ms.assetid: 8da267e8-2aeb-4c24-9a7a-65bdca3a82d6
ms.service: container-service
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/05/2017
ms.author: anhowe
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 538f282b28e5f43f43bf6ef28af20a4d8daea369
ms.openlocfilehash: 5c529ae41b42d276d37e6103305e33ed04694e18
ms.lasthandoff: 04/07/2017

---

# <a name="get-started-with-a-kubernetes-cluster-in-container-service"></a>Introdução ao cluster Kubernetes no Serviço de Contêiner


Este passo a passo mostra como usar os comandos da CLI do Azure 2.0 para criar um cluster Kubernetes no Serviço de Contêiner do Azure. Em seguida, você pode usar a ferramenta de linha de comando `kubectl` para começar a trabalhar com contêineres no cluster.

A imagem a seguir mostra a arquitetura de um cluster do Serviço de Contêiner com um mestre e dois agentes. O mestre serve a API REST do Kubernetes. Os nós de agente estão agrupados em um conjunto de disponibilidade do Azure e executam os seus contêineres. Todas as VMs estão na mesma rede virtual privada e são totalmente acessíveis entre si.

![Imagem do cluster Kubernetes no Azure](media/container-service-kubernetes-walkthrough/kubernetes.png)

## <a name="prerequisites"></a>Pré-requisitos
Este passo a passo presume que você instalou e configurou a [CLI do Azure 2.0](/cli/azure/install-az-cli2). 

Os exemplos de comando pressupõem que você executa a CLI do Azure em um shell bash, comum em Linux e macOS. Se você executar a CLI do Azure em um cliente Windows, algumas sintaxes de scripts e arquivos poderão diferir, dependendo do shell de comando. 

## <a name="create-your-kubernetes-cluster"></a>Criar o cluster Kubernetes

Veja comandos breves do shell que usam a CLI do Azure 2.0 para criar o cluster. 

### <a name="create-a-resource-group"></a>Criar um grupo de recursos
Para criar o cluster, você precisa primeiro criar um grupo de recursos em um local específico. Execute comandos semelhantes ao seguinte:

```azurecli
RESOURCE_GROUP=my-resource-group
LOCATION=westus
az group create --name=$RESOURCE_GROUP --location=$LOCATION
```

### <a name="create-a-cluster"></a>Criar um cluster
Quando você tiver um grupo de recursos, é possível criar um cluster nesse grupo. O exemplo a seguir usa a opção `--generate-ssh-keys`, que gera os arquivos de chave pública e privada SSH necessários para a implantação, se já não existirem no diretório `~/.ssh/` padrão. 

Esse comando também gera a [entidade de serviço do Active Directory do Azure](container-service-kubernetes-service-principal.md) a usada por um cluster Kubernetes no Azure automaticamente.

```azurecli
DNS_PREFIX=some-unique-value
CLUSTER_NAME=any-acs-cluster-name
az acs create --orchestrator-type=kubernetes --resource-group $RESOURCE_GROUP --name=$CLUSTER_NAME --dns-prefix=$DNS_PREFIX --generate-ssh-keys
```


Após alguns minutos, o comando é concluído e você deve ter um cluster Kubernetes em funcionamento.

### <a name="connect-to-the-cluster"></a>Conectar-se ao cluster

Para se conectar ao cluster Kubernetes do computador cliente, você deve usar [ `kubectl` ](https://kubernetes.io/docs/user-guide/kubectl/), o cliente de linha de comando do Kubernetes. 

Se você ainda não tem `kubectl` instalado, instale-o com:

```azurecli
sudo az acs kubernetes install-cli
```
> [!TIP]
> Por padrão, esse comando instala o binário `kubectl` em `/usr/local/bin/kubectl` em um sistema Linux ou macOS, ou em `C:\Program Files (x86)\kubectl.exe` no Windows. Para especificar um caminho de instalação diferente, use o parâmetro `--install-location`.
>

Depois que `kubectl` for instalado, verifique se seu diretório está no caminho do sistema ou adicione-o ao caminho. 


Em seguida, execute o comando a seguir para baixar a configuração do cluster Kubernetes mestre para o arquivo `~/.kube/config`:

```azurecli
az acs kubernetes get-credentials --resource-group=$RESOURCE_GROUP --name=$CLUSTER_NAME
```

Para obter mais opções instalar e configurar `kubectl`, consulte [Conectar-se a um cluster do Serviço de Contêiner do Azure](container-service-connect.md).

Agora você deve estar pronto para acessar o cluster em seu computador. Experimente executar:

```bash
kubectl get nodes
```

Verifique se é possível ver uma lista dos computadores no cluster.

## <a name="create-your-first-kubernetes-service"></a>Criar seu primeiro serviço Kubernetes

Depois de concluir este passo a passo, você saberá como:
* implantar um aplicativo Docker e expor ao mundo
* usar `kubectl exec` para executar comandos em um contêiner 
* acessar o painel do Kubernetes

### <a name="start-a-simple-container"></a>Iniciar um contêiner simples
Você pode executar um contêiner simples (nesse caso, o servidor Web Nginx), executando:

```bash
kubectl run nginx --image nginx
```

Esse comando inicia o contêiner do Docker Nginx em um pod em um dos nós.

Para ver o contêiner em execução, execute:

```bash
kubectl get pods
```

### <a name="expose-the-service-to-the-world"></a>Expor o serviço para o mundo
Para expor o serviço para o mundo, crie um `Service` Kubernetes do tipo `LoadBalancer`:

```bash
kubectl expose deployments nginx --port=80 --type=LoadBalancer
```

Esse comando faz com que o Kubernetes crie uma regra de Azure Load Balancer com um endereço IP público. A alteração demora alguns minutos para se propagar ao balanceador de carga. Para obter mais informações, consulte [Balancear a carga de contêineres em um cluster Kubernetes no Serviço de Contêiner do Azure](container-service-kubernetes-load-balancing.md).

Execute o comando a seguir para observar o serviço mudar de `pending` para a exibição de um endereço IP externo:

```bash
watch 'kubectl get svc'
```

  ![Imagem da observação da transição de pendente para endereço IP externo](media/container-service-kubernetes-walkthrough/kubernetes-nginx3.png)

Ao visualizar o endereço IP externo, você poderá navegar até ele no seu navegador:

  ![Imagem de navegação até o Nginx](media/container-service-kubernetes-walkthrough/kubernetes-nginx4.png)  


### <a name="browse-the-kubernetes-ui"></a>Navegar na interface do usuário do Kubernetes
Para ver a interface da Web do Kubernetes, você pode usar:

```bash
kubectl proxy
```
Esse comando executa um proxy autenticado simples no localhost, que pode ser usado para exibir a interface do usuário da Web do Kubernetes em [http://localhost:8001/ui](http://localhost:8001/ui). Para obter mais informações, consulte [Usando a interface do usuário da Web do Kubernetes com o Serviço de Contêiner do Azure](container-service-kubernetes-ui.md).

![Imagem do painel de Kubernetes](media/container-service-kubernetes-walkthrough/kubernetes-dashboard.png)

### <a name="remote-sessions-inside-your-containers"></a>Sessões remotas em seus contêineres
O Kubernetes permite executar comandos em um contêiner remoto do Docker que está execução no seu cluster.

```bash
# Get the name of your nginx pods
kubectl get pods
```

Usando seu nome pod, é possível executar um comando remoto no seu pod.  Por exemplo:

```bash
kubectl exec <pod name> date
```

Você também pode obter uma sessão totalmente interativa usando os sinalizadores `-it`:

```bash
kubectl exec <pod name> -it bash
```

![Sessão remota dentro de um contêiner](media/container-service-kubernetes-walkthrough/kubernetes-remote.png)



## <a name="next-steps"></a>Próximas etapas

Faça mais com seu cluster Kubernetes, consulte os seguintes recursos:

* [Kubernetes Bootcamp](https://katacoda.com/embed/kubernetes-bootcamp/1/) – Mostra como implantar, dimensionar, atualizar e depurar aplicativos em contêineres.
* [Guia do Usuário do Kubernetes](http://kubernetes.io/docs/user-guide/) – fornece informações sobre como executar programas em um cluster Kubernetes existente.
* [Exemplos de Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/examples) – fornece exemplos de como executar aplicativos reais com Kubernetes.

