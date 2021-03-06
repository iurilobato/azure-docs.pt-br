---
title: "Azure Active Directory B2C: solucionar problemas de políticas personalizadas | Microsoft Docs"
description: "Um tópico sobre como solucionar problemas com políticas personalizadas do Azure Active Directory B2C"
services: active-directory-b2c
documentationcenter: 
author: saeeda
manager: krassk
editor: parakhj
ms.assetid: 658c597e-3787-465e-b377-26aebc94e46d
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.devlang: na
ms.date: 04/04/2017
ms.author: saeeda
ms.translationtype: Human Translation
ms.sourcegitcommit: a3ca1527eee068e952f81f6629d7160803b3f45a
ms.openlocfilehash: 2fa67038f2a214c1569fc65fd9f1beba394cb790
ms.contentlocale: pt-br
ms.lasthandoff: 04/27/2017

---

# <a name="azure-active-directory-b2c-collecting-logs"></a>Azure Active Directory B2C: Coleta de logs

Este artigo fornece etapas para coletar logs do Azure AD B2C para que você possa diagnosticar problemas com suas políticas personalizadas.

## <a name="use-application-insights"></a>Usar o Application insights

O Azure AD B2C oferece suporte a um recurso para envio de dados ao Application Insights.  O Application Insights fornece uma maneira de diagnosticar exceções e visualizar os problemas de desempenho do aplicativo.

### <a name="setup-application-insights"></a>Configurar o Application Insights

1. Vá para o [Portal do Azure](https://portal.azure.com). Certifique-se de que você esteja no locatário com sua assinatura do Azure (não no locatário do Azure AD B2C).
1. Clique em **+ Novo** no menu de navegação esquerdo.
1. Pesquise e selecione **Application Insights** e clique em **Criar**.
1. Preencha o formulário e clique em **Criar**. Selecione **Geral** para o **Tipo de Aplicativo**.
1. Após a criação do recurso, abra o recurso Application Insights.
1. Localize **Propriedades** no menu à esquerda e clique nele.
1. Copie a **Chave de Instrumentação** e salve-a na próxima seção.

### <a name="set-up-the-custom-policy"></a>Configurar a política personalizada

1. Abra o arquivo RP (por exemplo, SignUpOrSignin.xml).
1. Adicione os atributos a seguir ao elemento `<TrustFrameworkPolicy>`:

  ```XML
  UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
  ```

1. Se ele ainda não existir, adicione um nó filho `<UserJourneyBehaviors>` ao nó `<RelyingParty>`.
2. Adicione o seguinte nó como um filho do elemento `<UserJourneyBehaviors>`. Substitua `{Your Application Insights Key}` pela **Chave de Instrumentação** que você obteve na seção anterior.

  ```XML
  <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
  ```

  * `DeveloperMode="true"` informa ao ApplicationInsights para agilizar a telemetria por meio do pipeline de processamento, bom para o desenvolvimento, mas com restrição em grandes volumes.
  * `ClientEnabled="true"` envia o script do lado do cliente do ApplicationInsights para rastrear erros de exibição de página e do lado do cliente (não é necessário).
  * `ServerEnabled="true"` envia o JSON UserJourneyRecorder existente como um evento personalizado para o Application Insights.
  O XML final terá a aparência a seguir:

  ```XML
  <TrustFrameworkPolicy
    ...
    TenantId="fabrikamb2c.onmicrosoft.com"
    PolicyId="SignUpOrSignInWithAAD"
    UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
  >
    ...
    <RelyingParty>
      <UserJourneyBehaviors>
        <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      </UserJourneyBehaviors>
      ...
  </TrustFrameworkPolicy>
  ```

3. Carregue a política.

### <a name="see-the-logs"></a>Consulte os logs

>[!NOTE]
> Há um breve atraso (menos de cinco minutos) antes que você possa ver novos logs no Application Insights.

1. Abra o recurso do Application Insights que você criou no [Portal do Azure](https://portal.azure.com).
1. No menu **Visão geral**, clique em **Análise**.
1. Abra uma nova guia no Application Insights.
1. Aqui está uma lista de consultas que você pode usar para ver os logs

| Consultar | Descrição |
|---------------------|--------------------|
traces | Veja todos os logs gerados pelo Azure AD B2C |
traces \| em que timestamp > ago(1d) | Veja todos os logs gerados pelo Azure AD B2C para o último dia

Saiba mais sobre essa ferramentas de análise [aqui](https://docs.microsoft.com/azure/application-insights/app-insights-analytics).





