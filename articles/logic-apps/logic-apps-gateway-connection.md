---
title: "Acesso a dados locais - Aplicativo Lógico do Azure no local | Microsoft Docs"
description: "Como seus aplicativos lógicos podem acessar dados locais conectando-se a um gateway de dados local."
services: logic-apps
documentationcenter: .net,nodejs,java
author: jeffhollan
manager: anneta
editor: 
ms.assetid: 6cb4449d-e6b8-4c35-9862-15110ae73e6a
ms.service: logic-apps
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 07/05/2016
ms.author: jehollan
translationtype: Human Translation
ms.sourcegitcommit: c300ba45cd530e5a606786aa7b2b254c2ed32fcd
ms.openlocfilehash: 3e9b95e6e9e84f8c2b615f43783d9fec5a2c09b6
ms.lasthandoff: 04/14/2017


---
# <a name="connect-to-on-premises-data-from-logic-apps"></a>Conectar-se a dados locais de aplicativos lógicos

Para acessar dados locais, você pode configurar uma conexão com um gateway de dados locais para conectores do Aplicativo Lógico do Azure com suporte. As etapas a seguir explicam como instalar e configurar o gateway de dados local para trabalhar com seus aplicativos lógicos. O gateway de dados local dá suporte a estas conexões:

*   BizTalk Server
*   DB2  
*   Sistema de Arquivos
*   Informix
*   MQ
*   MySQL
*   Banco de dados Oracle 
*   Servidor de aplicativos SAP 
*   Servidor de mensagens SAP
*   SharePoint somente para HTTP e não para HTTPS
*   SQL Server
*   Teradata

Para obter mais informações sobre essas conexões, confira [Conectores para Aplicativos Lógicos do Azure](https://docs.microsoft.com/azure/connectors/apis-list).

## <a name="requirements"></a>Requisitos

* Você tem que usar um endereço de email corporativo ou de estudante no Azure para associar o gateway de dados local com sua conta (conta do Azure Active Directory).

* Se estiver usando uma Conta da Microsoft (por exemplo, @outlook.com), você poderá usar sua conta do Azure para [criar um endereço de email corporativo ou de estudante](../virtual-machines/windows/create-aad-work-id.md#locate-your-default-directory-in-the-azure-classic-portal).

* Você tem que ter o [gateway de dados local instalado em um computador local](logic-apps-gateway-install.md).

* Você pode associar a instalação a apenas um recurso de gateway. O gateway não pode ser exibido por outro gateway de dados local do Azure. A declaração ocorre na ([criação durante a etapa 2 deste tópico](#2-create-an-azure-on-premises-data-gateway-resource)).

## <a name="install-and-configure-the-connection"></a>Instalar e configurar a conexão

### <a name="1-install-the-on-premises-data-gateway"></a>1. Instale o gateway de dados local

Se você ainda não fez isso, siga estas etapas para [instalar o gateway de dados local](logic-apps-gateway-install.md). Antes de continuar com as outras etapas, verifique se você instalou o gateway de dados em uma máquina local.

### <a name="2-create-an-azure-on-premises-data-gateway-resource"></a>2. Crie um recurso de gateway de dados local do Azure

Depois de instalar o gateway, você deverá associar sua assinatura do Azure a ele.

> [!IMPORTANT] 
> Verifique se o recurso do gateway foi criado na mesma região do Azure que seu aplicativo lógico. Se você não implantar o recurso do gateway na mesma região, seu aplicativo lógico não poderá acessar o gateway. 
> 

1. Entre no Azure usando o mesmo endereço de email corporativo ou escolar usado durante a instalação do gateway.
2. Escolha **Novo**.
3. Encontre e selecione o **gateway de dados local**.
4. Para associar o gateway à sua conta, preencha as informações e selecione o **Nome de Instalação**.
   
    ![Conexão ao Gateway de Dados Local][1]

5. Para criar o recurso, escolha **Criar**.

### <a name="3-create-a-logic-app-connection-in-logic-app-designer"></a>3. Criar uma conexão com o Designer de Aplicativos Lógicos

Agora que sua assinatura do Azure está associada a uma instância do gateway de dados local, você pode criar uma conexão com o gateway do aplicativo lógico.

1. Abra um aplicativo lógico e escolha um conector que ofereça suporte à conectividade local, como o SQL Server.
2. Selecione **Conectar por meio do gateway de dados local**.
   
    ![Criação de Gateway do Designer de Aplicativo Lógico][2]

3. Selecione o **Gateway** que você deseja conectar e preencha outras informações de conexão necessárias.
4. Para criar a conexão, escolha **Criar**.

Sua conexão agora está configurada para ser usada pelo aplicativo lógico.

## <a name="edit-your-data-gateway-connection-settings"></a>Editar as configurações de conexão do seu gateway de dados

Depois de adicionar a conexão do gateway de dados ao seu aplicativo lógico, talvez seja necessário fazer alterações para que você possa ajustar as configurações específicas para essa conexão. É possível encontrar a conexão em um destes dois locais:

* Na folha do aplicativo lógico, em **Ferramentas de Desenvolvimento**, selecione **Conexões de API**. Esta lista mostra todas as Conexões de API associadas ao seu aplicativo lógico, incluindo sua conexão de gateway de dados. Para exibir e modificar as configurações dessa conexão, selecione essa conexão.

* Na folha principal de Conexões de API, você pode encontrar todas as Conexões de API associadas à sua assinatura do Azure, incluindo sua conexão de gateway de dados. Para exibir e modificar as configurações da conexão, selecione essa conexão.

## <a name="next-steps"></a>Próximas etapas

* [Exemplos comuns e cenários de aplicativos lógicos](../logic-apps/logic-apps-examples-and-scenarios.md)
* [Recursos de integração corporativa](../logic-apps/logic-apps-enterprise-integration-overview.md)

<!-- Image references -->
[1]: ./media/logic-apps-gateway-connection/createblade.png
[2]: ./media/logic-apps-gateway-connection/blankconnection.png
[3]: ./media/logic-apps-logic-gateway-connection/checkbox.png

