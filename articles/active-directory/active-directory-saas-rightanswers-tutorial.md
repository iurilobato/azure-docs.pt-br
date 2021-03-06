---
title: "Tutorial: integração do Azure Active Directory ao RightAnswers | Microsoft Docs"
description: "Saiba como usar o RightAnswers com o Active Directory do Azure para habilitar o logon único, provisionamento automatizado e muito mais!"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 7f09e25a-a716-41e1-8ca3-fd00e3d1b8cc
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/23/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 62aa62c41e7ac59212fb9f06a1a104984db90621
ms.lasthandoff: 04/03/2017


---
# <a name="tutorial-azure-active-directory-integration-with-rightanswers"></a>Tutorial: Integração do Active Directory do Azure ao RightAnswers
O objetivo deste tutorial é mostrar a integração do Azure com o RightAnswers. O cenário descrito neste tutorial pressupõe que você já tem os seguintes itens:

* Uma assinatura válida do Azure
* Uma assinatura do RightAnswers com SSO (logon único) habilitado

Depois de concluir este tutorial, os usuários do Azure AD atribuídos ao RightAnswers poderão fazer logon único no aplicativo usando a [Introdução ao Painel de Acesso](active-directory-saas-access-panel-introduction.md).

O cenário descrito neste tutorial consiste nos seguintes blocos de construção:

1. Habilitando a integração de aplicativos com o RightAnswers
2. Configuração do SSO (logon único)
3. Configurando o provisionamento de usuários
4. Atribuindo usuários

![Cenário](./media/active-directory-saas-rightanswers-tutorial/IC802925.png "Cenário")

## <a name="enable-the-application-integration-for-rightanswers"></a>Habilitar a integração de aplicativos com o RightAnswers
O objetivo desta seção é descrever como habilitar a integração de aplicativos com o RightAnswers.

**Para habilitar a integração de aplicativos com o RightAnswers, execute as seguintes etapas:**

1. No Portal clássico do Azure, no painel de navegação à esquerda, clique em **Active Directory**.
   
    ![Active Directory](./media/active-directory-saas-rightanswers-tutorial/IC700993.png "Active Directory")
2. Na lista **Diretório** , selecione o diretório para o qual você deseja habilitar a integração de diretórios.
3. Para abrir a visualização dos aplicativos, na exibição do diretório, clique em **Aplicativos** no menu principal.
   
    ![Aplicativos](./media/active-directory-saas-rightanswers-tutorial/IC700994.png "Aplicativos")
4. Clique em **Adicionar** na parte inferior da página.
   
    ![Adicionar aplicativo](./media/active-directory-saas-rightanswers-tutorial/IC749321.png "Adicionar aplicativo")
5. Na caixa de diálogo **O que você deseja fazer**, clique em **Adicionar um aplicativo da galeria**.
   
    ![Adicionar um aplicativo da galeria](./media/active-directory-saas-rightanswers-tutorial/IC749322.png "Adicionar um aplicativo da galeria")
6. Na **caixa de pesquisa**, digite **RightAnswers**.
   
    ![Galeria de Aplicativos](./media/active-directory-saas-rightanswers-tutorial/IC802926.png "Galeria de Aplicativos")
7. No painel de resultados, selecione **RightAnswers** e clique em **Concluir** para adicionar o aplicativo.
   
## <a name="configure-single-sign-on"></a>Configurar o logon único

O objetivo desta seção é descrever como permitir que os usuários se autentiquem no RightAnswers com sua conta do AD do Azure usando federação baseada em protocolo SAML.

**Para configurar o logon único, execute as seguintes etapas:**

1. No Portal Clássico do Azure, na página de integração de aplicativos do **RightAnswers**, clique em **Configurar logon único** para abrir a caixa de diálogo **Configurar Logon Único**.
   
    ![Configurar Logon Único](./media/active-directory-saas-rightanswers-tutorial/IC802927.png "Configurar Logon Único")
2. Na página **Como você deseja que os usuários façam logon no RightAnswers**, selecione **Logon Único do Microsoft Azure AD** e clique em **Avançar**.
   
    ![Configurar Logon Único](./media/active-directory-saas-rightanswers-tutorial/IC802928.png "Configurar Logon Único")
3. Na página **Definir Configurações do Aplicativo**, na caixa de texto **URL de Entrada**, digite a URL usada pelos usuários para fazer logon em seu aplicativo RightAnswers (por exemplo: *https://fortify.rightanswers.com/portal/ss/*) e clique em **Avançar**.
   
    ![Definir Configurações de Aplicativo](./media/active-directory-saas-rightanswers-tutorial/IC802929.png "Definir Configurações de Aplicativo")
4. Na página **Configurar logon único no RightAnswers**, para baixar os metadados, clique em **Baixar metadados** e salve o arquivo de metadados localmente no computador.
   
    ![Configurar Logon Único](./media/active-directory-saas-rightanswers-tutorial/IC802930.png "Configurar Logon Único")
5. Envie o arquivo de metadados baixado para a equipe de suporte do RightAnswers.
   
    >[!NOTE]
    >A equipe de suporte do RightAnswers precisa fazer a configuração real do SSO.
    >Você receberá uma notificação quando o SSO tiver sido habilitado para sua assinatura.
    > 
    > 

6. No portal clássico do Azure, selecione a confirmação da configuração de logon único e clique em **Concluir** para fechar a caixa de diálogo **Configurar logon único**.
   
    ![Configurar Logon Único](./media/active-directory-saas-rightanswers-tutorial/IC802931.png "Configurar Logon Único")
   
## <a name="configure-user-provisioning"></a>Configurar provisionamento do usuário

Para permitir que os usuários do AD do Azure façam logon no RightAnswers, eles devem ser provisionados no RightAnswers. No caso do RightAnswers, o provisionamento é uma tarefa automatizada, então não há nenhum item de ação para você.

Os usuários são criados automaticamente, se necessário, durante a primeira tentativa de logon único.

>[!NOTE]
>É possível usar qualquer outra ferramenta de criação da conta de usuário do RightAnswers ou as APIs fornecidas pelo RightAnswers para provisionar as contas de usuário do AAD.
> 
> 

## <a name="assign-users"></a>Atribuir usuários
Para testar sua configuração, é necessário conceder acesso ao aplicativo aos usuários do Azure AD que você deseja que usem seu aplicativo.

**Para atribuir usuários ao RightAnswers, execute as seguintes etapas:**

1. No Portal clássico do Azure, crie uma conta de teste.

2. Na página de integração de aplicativos do **RightAnswers**, clique em **Atribuir usuários**.
   
    ![Atribuir Usuários](./media/active-directory-saas-rightanswers-tutorial/IC802932.png "Atribuir Usuários")
3. Selecione seu usuário de teste, clique em **Atribuir** e, em seguida, clique em **Sim** para confirmar a atribuição.
   
    ![Sim](./media/active-directory-saas-rightanswers-tutorial/IC767830.png "Sim")

Se você quiser testar suas configurações de SSO, abra o Painel de Acesso. Para obter mais detalhes sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](active-directory-saas-access-panel-introduction.md).


