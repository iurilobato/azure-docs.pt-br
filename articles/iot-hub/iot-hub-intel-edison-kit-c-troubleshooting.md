---
title: "Conectar o Intel Edison (C) ao IoT do Azure - Solução de problemas | Microsoft Docs"
description: "Página de solução de problemas da experiência do Intel Edison C"
services: iot-hub
documentationcenter: 
author: shizn
manager: timtl
tags: 
keywords: "solução de problemas do arduino"
ROBOTS: NOINDEX
redirect_url: /azure/iot-hub/iot-hub-intel-edison-kit-c-get-started
ms.assetid: fe20f2fe-490c-4910-82e1-578ed504ae86
ms.service: iot-hub
ms.devlang: c
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 3/21/2017
ms.author: xshi
translationtype: Human Translation
ms.sourcegitcommit: 475b25f02715a60493e79ecd2170854019dfc4ac
ms.openlocfilehash: c35afd54fb20c3ecd1a70796094093ad748385d0
ms.lasthandoff: 01/25/2017


---
# <a name="troubleshooting"></a>Solucionar problemas
## <a name="hardware-issues"></a>Problemas de hardware
Para obter informações sobre como resolver problemas comuns no Intel Edison, consulte a [página oficial de solução de problemas](https://software.intel.com/en-us/node/637974).

## <a name="nodejs-package-issues"></a>Problemas do pacote de Node.js
### <a name="no-response-during-gulp-tasks"></a>Sem resposta durante as tarefas de gulp
Se você encontrar problemas ao executar tarefas de gulp, será possível adicionar a opção `--verbose` para depuração. Tente finalizar as tarefas de gulp atuais usando `Ctrl + C` e, em seguida, execute o seguinte comando na janela do console para ver mensagens de depuração. Você pode ver mensagens de erro detalhadas em sua saída do console. 

```bash
gulp --verbose
```

### <a name="npm-issues"></a>Problemas de NPM
Tente atualizar o pacote NPM com o comando a seguir:

```bash
npm install -g npm
```

Se o problema persistir, deixe seus comentários no final deste artigo ou crie um problema do GitHub em nosso [repositório de exemplo][sample-repository].

## <a name="azure-cli-issues"></a>Problemas da CLI do Azure
A interface de linha de comando do Azure (CLI do Azure) é uma compilação de visualização. Procure pela solução no [Guia de Instalação de Visualização](https://github.com/Azure/azure-cli/blob/master/doc/preview_install_guide.md) para buscar soluções. Tente atualizar a CLI do Azure para a versão mais recente quando comandos não funcionarem conforme o esperado.

Se você encontrar erros com a ferramenta, emita um [problema](https://github.com/Azure/azure-cli/issues) na seção **problemas** do repositório do GitHub.

Para obter ajuda para solucionar problemas comuns, consulte o [Leiame](https://github.com/Azure/azure-cli/blob/master/README.rst).

Se você receber a mensagem “Não foi possível localizar uma versão que atenda ao requisito”, execute o seguinte comando para atualizar o pip para a versão mais recente.

```bash
python -m pip install --upgrade pip
```

## <a name="python-installation-issues"></a>Problemas de instalação do Python
### <a name="legacy-installation-issues-macos"></a>Problemas de instalação herdada (macOS)
Ao instalar o **pip**, um erro de permissão será lançado quando houver pacotes mais antigos instalados com permissões **su**. Essa situação ocorre porque a instalação anterior do Python usando brew (macOS) não está completamente desinstalada. Alguns pacotes do **pip** de uma instalação anterior foram criados pela raiz, o que causa o erro de permissão. A solução é remover os pacotes instalados pela raiz. Use as etapas a seguir para concluir esta tarefa:

1. Acesse: /usr/local/lib/python2.7/site-packages
2. Listar pacotes criados por raiz: `ls -l | grep root`
3. Desinstalar pacotes da etapa 2: `sudo rm -rf {package name}`
4. Reinstale o Python.

## <a name="azure-iot-hub-issues"></a>Problemas do Hub IoT do Azure
Se você tiver provisionado com êxito o Hub IoT do Azure com `azure-cli` e precisar de uma ferramenta para gerenciar os dispositivos conectados ao seu Hub IoT, experimente usar as seguintes ferramentas:

### <a name="device-explorer"></a>Gerenciador de Dispositivos
O [Gerenciador de Dispositivos](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer) é executado no computador local do Windows e se conecta ao seu hub IoT no Azure. Ele se comunica com os seguintes [pontos de extremidade de Hub IoT](iot-hub-devguide.md):

- _Gerenciamento de identidade do dispositivo_ para provisionar e gerenciar dispositivos registrados com seu hub IoT.
- _Receber do dispositivo para nuvem_ para que você possa monitorar as mensagens enviadas de seu dispositivo ao seu Hub IoT.
- _Enviar do dispositivo para nuvem_ para que você possa enviar mensagens para seus dispositivos de seu Hub IoT.

Configure o `IoT hub connection string` dentro desta ferramenta para usar todos os seus recursos.

### <a name="iot-hub-explorer"></a>Gerenciador do Hub IoT
O [Gerenciador do Hub IoT](https://github.com/Azure/iothub-explorer) é uma ferramenta de CLI de várias plataformas de exemplo para gerenciar clientes de dispositivo. Use a ferramenta para gerenciar os dispositivos no registro de identidade, monitorar mensagens de dispositivo para a nuvem e enviar comandos de nuvem para o dispositivo.

Para instalar a versão mais recente (pré-lançamento) da ferramenta Gerenciador do iothub, execute o comando a seguir em seu ambiente de linha de comando:

```bash
npm install -g iothub-explorer@latest
```

Você pode usar o comando a seguir para obter ajuda adicional sobre todos os comandos do Gerenciador do iothub e seus parâmetros:

```bash
iothub-explorer help
```

### <a name="azure-portal"></a>Portal do Azure
Uma experiência completa de CLI ajuda você a criar e gerenciar todos os recursos do Azure. Você também poderá usar o [portal do Azure](../azure-portal-overview.md) para ajudar a provisionar, gerenciar e depurar seus recursos do Azure.

## <a name="azure-storage-issues"></a>Problemas de Armazenamento do Azure
[O Gerenciador de Armazenamento do Microsoft Azure (visualização)](http://storageexplorer.com) é um aplicativo autônomo da Microsoft que pode ser usado para trabalhar com os dados do [Armazenamento do Azure](https://azure.microsoft.com/en-us/services/storage/) no Windows, macOS e Linux. Com essa ferramenta você pode se conectar à sua tabela e ver os dados contidos nela. Você pode usar essa ferramenta para solucionar seus problemas de Armazenamento do Azure.

## <a name="next-steps"></a>Próximas etapas
Esta página inclui apenas os problemas mais comuns do kit Intel Edison. Você também pode deixar comentários na parte inferior para relatar problemas para solução de problemas no futuro.

Voltar para [Introdução ao Intel Edison (C)](iot-hub-intel-edison-kit-c-get-started.md)

<!-- Images and links -->

[sample-repository]: https://github.com/Azure-Samples/iot-hub-c-edison-getting-started
