---
title: Usar o Docker Machine para criar hosts Linux no Azure | Microsoft Docs
description: Descreve o uso do computador Docker para criar hosts do Docker no Azure.
services: virtual-machines-linux
documentationcenter: 
author: squillace
manager: timlt
editor: tysonn
ms.assetid: 164b47de-6b17-4e29-8b7d-4996fa65bea4
ms.service: virtual-machines-linux
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/22/2016
ms.author: rasquill
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: b303510970aca957a4da5f2ed51a9125302d419a
ms.lasthandoff: 04/03/2017


---
# <a name="use-docker-machine-with-the-azure-driver"></a>Usar o computador Docker com o driver do Azure
O [Docker](https://www.docker.com/) fornece virtualização usando contêineres Linux em vez de VMs para isolar dados do aplicativo e computação em um recurso compartilhado. Este tópico descreve como e quando usar o [Computador Docker](https://docs.docker.com/machine/). O comando `docker-machine` cria uma nova VM Linux no Azure habilitada como um host de docker para contêineres do Linux.

## <a name="create-vms-with-docker-machine"></a>Criar VMs com o computador Docker
Crie VMs de host do Docker no Azure com o comando `docker-machine create` usando o argumento de driver `azure` para a opção de driver (`-d`) e para todos os outros argumentos. 

O exemplo a seguir conta com os valores padrão, mas abre a porta 80 na VM com a Internet para testar com um contêiner nginx, cria o usuário de logon `ops` para o SSH e chama a nova VM `machine`. 

Digite `docker-machine create --driver azure` para ver as opções e seus valores padrão; você também pode ler a [documentação do Docker Azure Driver](https://docs.docker.com/machine/drivers/azure/). (Observe que, se a autenticação de dois fatores estiver habilitada, você deverá autenticar usando o segundo fator).

```bash
docker-machine create -d azure \
  --azure-ssh-user ops \
  --azure-subscription-id <Your AZURE_SUBSCRIPTION_ID> \
  --azure-open-port 80 \
  machine
```

Dependendo de a autenticação de dois fatores estar configurada em sua conta, a saída deverá se parecer com o seguinte.

```bash
Creating CA: /Users/user/.docker/machine/certs/ca.pem
Creating client certificate: /Users/user/.docker/machine/certs/cert.pem
Running pre-create checks...
(machine) Microsoft Azure: To sign in, use a web browser to open the page https://aka.ms/devicelogin. Enter the code <code> to authenticate.
(machine) Completed machine pre-create checks.
Creating machine...
(machine) Querying existing resource group.  name="machine"
(machine) Creating resource group.  name="machine" location="eastus"
(machine) Configuring availability set.  name="docker-machine"
(machine) Configuring network security group.  name="machine-firewall" location="eastus"
(machine) Querying if virtual network already exists.  name="docker-machine-vnet" location="eastus"
(machine) Configuring subnet.  name="docker-machine" vnet="docker-machine-vnet" cidr="192.168.0.0/16"
(machine) Creating public IP address.  name="machine-ip" static=false
(machine) Creating network interface.  name="machine-nic"
(machine) Creating storage account.  name="vhdsolksdjalkjlmgyg6" location="eastus"
(machine) Creating virtual machine.  name="machine" location="eastus" size="Standard_A2" username="ops" osImage="canonical:UbuntuServer:15.10:latest"
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with ubuntu(systemd)...
Installing Docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env machine
```

## <a name="configure-your-docker-shell"></a>Configurar o shell do Docker
Agora, digite `docker-machine env <VM name>` para ver o que você precisa fazer para configurar o shell. 

```bash
docker-machine env machine
```

Esse comando imprime as informações de ambiente do seu contêiner, a saída deve ser parecida com a imagem a seguir. Observe que o endereço IP, que você precisará para testar a VM foi atribuído.

```bash
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://191.237.46.90:2376"
export DOCKER_CERT_PATH="/Users/rasquill/.docker/machine/machines/machine"
export DOCKER_MACHINE_NAME="machine"
# Run this command to configure your shell:
# eval $(docker-machine env machine)
```

Você pode executar o comando de configuração sugerido ou definir as variáveis de ambiente por si. 

## <a name="run-a-container"></a>Executar um contêiner
Agora você pode executar um servidor Web simples para testar se tudo está funcionando corretamente. Aqui usamos uma imagem padrão do nginx, especificamos que ela deve escutar na porta 80 e, que se a VM for reiniciada, o contêiner deverá ser reiniciado também (`--restart=always`). 

```bash
docker run -d -p 80:80 --restart=always nginx
```

A saída deve ter uma aparência semelhante ao seguinte:

```bash
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
efd26ecc9548: Pull complete
a3ed95caeb02: Pull complete
83f52fbfa5f8: Pull complete
fa664caa1402: Pull complete
Digest: sha256:12127e07a75bda1022fbd4ea231f5527a1899aad4679e3940482db3b57383b1d
Status: Downloaded newer image for nginx:latest
25942c35d86fe43c688d0c03ad478f14cc9c16913b0e1c2971cb32eb4d0ab721
```

## <a name="test-the-container"></a>Testar o contêiner
Examine os contêineres em execução usando `docker ps`:

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
d5b78f27b335        nginx               "nginx -g 'daemon off"   5 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp, 443/tcp   goofy_mahavira
```

E verifique o contêiner em execução, digite `docker-machine ip <VM name>` para localizar o endereço IP (se você se esqueceu dele do comando `env`):

![Contêiner ngnix em execução](./media/docker-machine/nginxsuccess.png)

## <a name="next-steps"></a>Próximas etapas
Se estiver interessado, você poderá experimentar a [Extensão de VM do Docker](dockerextension.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) do Azure para fazer a mesma operação usando a CLI do Azure ou os modelos do Azure Resource Manager. 

Para obter mais exemplos de como trabalhar com o Docker, consulte [Trabalhando com o Docker](https://github.com/Microsoft/HealthClinic.biz/wiki/Working-with-Docker) na [demonstração](https://blogs.msdn.microsoft.com/visualstudio/2015/12/08/connectdemos-2015-healthclinic-biz/) do [HealthClinic.biz](https://github.com/Microsoft/HealthClinic.biz) 2015 Connect. Para ver mais guias de início rápido da demonstração do HealthClinic.biz, consulte [Azure Developer Tools Quickstarts (Guias de início rápido das ferramentas de desenvolvedor do Azure)](https://github.com/Microsoft/HealthClinic.biz/wiki/Azure-Developer-Tools-Quickstarts).


