---
title: "Atualizar solução de gerenciamento no OMS | Microsoft Docs"
description: "Este artigo destina-se a ajudá-lo a entender como usar essa solução para gerenciar atualizações para os computadores Windows e Linux."
services: operations-management-suite
documentationcenter: 
author: MGoedtel
manager: carmonm
editor: 
ms.assetid: e33ce6f9-d9b0-4a03-b94e-8ddedcc595d2
ms.service: operations-management-suite
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 05/02/2017
ms.author: magoedte
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: 03a6c1f20632691c08f5de4afe74eacc6f79608e
ms.contentlocale: pt-br
ms.lasthandoff: 05/03/2017


---
# <a name="update-management-solution-in-oms"></a>Solução Gerenciamento de Atualizações no OMS
A solução Gerenciamento de Atualizações no OMS permite que você gerencie atualizações para os computadores Windows e Linux.  Você pode avaliar o status de atualizações disponíveis em todos os computadores de agente e iniciar rapidamente o processo de instalação das atualizações necessárias para os servidores. 

## <a name="solution-components"></a>Componentes da solução

Computadores gerenciados pelo OMS usam o seguinte para executar implantações de avaliação e atualização: 

* Agente do OMS para Windows ou Linux
* DSC (PowerShell Desired State Configuration) para Linux 
* Hybrid Runbook Worker de Automação 
* Microsoft Update ou Windows Server Update Services para computadores Windows

Os diagramas a seguir mostram uma exibição conceitual do comportamento e do fluxo de dados, indicando como a solução avalia e aplica atualizações a todos os computadores Linux e servidores Windows conectados em um espaço de trabalho.    

#### <a name="windows-server"></a>Windows Server
![Fluxo de processo de gerenciamento de atualização do Windows Server](media/oms-solution-update-management/update-mgmt-windows-updateworkflow.png)

#### <a name="linux"></a>Linux
![Fluxo do processo de gerenciamento de atualização do Linux](media/oms-solution-update-management/update-mgmt-linux-updateworkflow.png)

Depois que o computador executa uma verificação de conformidade da atualização, o agente do OMS encaminha as informações em massa ao OMS. Em um computador Windows, a verificação de conformidade é executada a cada 12 horas por padrão.  Além do agendamento da verificação, a verificação de conformidade de atualização será iniciada em 15 minutos se o MMA (Agente de Monitoramento da Microsoft) for reiniciado antes da instalação da atualização e após a instalação da atualização.  Com um computador Linux, a verificação de conformidade é executada a cada três horas por padrão, e uma verificação de conformidade é iniciada em 15 minutos, se o agente MMA é reiniciado.  

As informações de conformidade são processadas e resumidas nos painéis incluídos na solução ou são pesquisáveis usando consultas definidas pelo usuário ou pré-definidas.  A solução relata o grau de atualização do computador com base na fonte com a qual você está configurado para realizar a sincronização.  Se o computador do Windows estiver configurado para relatar para o WSUS, dependendo de quando o WSUS foi sincronizada pela última vez com o Microsoft Update, os resultados poderão diferir do que é mostrado pelo Microsoft Updates.  O mesmo se aplica a computadores Linux que estão configurados para relatar para um repositório local versus um repositório público.   

Você pode implantar e instalar atualizações de software em computadores que precisam de atualizações, criando uma implantação agendada.  As atualizações classificadas como *Opcional* não são incluídas no escopo de implantação para computadores Windows, somente as atualizações necessárias.  A implantação agendada define quais computadores de destino receberão as atualizações aplicáveis, explicitamente especificando computadores ou selecionando um [grupo de computadores](../log-analytics/log-analytics-computer-groups.md) que se baseia em pesquisas de log de determinado conjunto de computadores.  Você também pode especificar uma agenda para aprovar e designar um período de tempo quando é permitido que as atualizações sejam instaladas.  As atualizações são instaladas por runbooks na Automação do Azure.  Você não consegue exibir esses runbooks e eles não exigem nenhuma configuração.  Quando uma Implantação de Atualizações é criada, ela cria uma agenda que inicia um runbook de atualização mestre no momento especificado para os computadores incluídos.  Esse runbook mestre inicia um runbook filho em cada agente que executa a instalação de atualizações necessárias.       

Na data e hora especificadas na implantação da atualização, os computadores de destino executam a implantação em paralelo.  Uma verificação é executada primeiro para verificar se as atualizações ainda são necessárias e as instala.  É importante observar que, para computadores cliente do WSUS, se as atualizações não forem aprovadas no WSUS, a implantação de atualização falhará.  Os resultados das atualizações aplicadas são encaminhados ao OMS para serem processados e resumidos em painéis ou com a pesquisa de eventos.     

## <a name="prerequisites"></a>Pré-requisitos
* A solução dá suporte à execução de avaliações de atualização no Windows Server 2008 e superior e à atualização de implantações no Windows Server 2012 e superior.  Não há suporte para as opções de instalação Server Core e Nano Server.
* Não há suporte para sistemas operacionais clientes do Windows.  
* Os agentes do Windows devem ser configurados para se comunicar com um servidor WSUS (Windows Server Update Services) ou ter acesso ao Microsoft Update.  
  
    > [!NOTE]
    > O agente do Windows não pode ser gerenciado simultaneamente pelo System Center Configuration Manager.  
    >
* CentOS 6 (x86/x64) e 7 (x64)
* Red Hat Enterprise 6 (x86/x64) e 7 (x64)
* SUSE Linux Enterprise Server 11 (x86/x64) e 12 (x64)
* Ubuntu 12.04 LTS e posterior x86/x64  
* Os agentes do Linux devem ter acesso a um repositório de atualização.  

    > [!NOTE]
    > Um agente do OMS para Linux configurado para emitir relatórios para vários espaços de trabalho do OMS não tem suporte nessa solução.  
    > 

Para obter informações adicionais sobre como instalar o agente do OMS para Linux e baixar a versão mais recente, confira [Operations Management Suite Agent para Linux](https://github.com/microsoft/oms-agent-for-linux).  Para obter informações sobre como instalar o Agente do OMS para Windows, confira [Operations Management Suite Agent para Windows](../log-analytics/log-analytics-windows-agents.md).  

## <a name="solution-components"></a>Componentes da solução
Essa solução consiste nos seguintes recursos que são adicionados à sua conta de Automação e a agentes conectados diretamente ou ao grupo de gerenciamento conectado do Operations Manager. 

### <a name="management-packs"></a>Pacotes de gerenciamento
Se o grupo de gerenciamento do System Center Operations Manager estiver conectado a um espaço de trabalho do OMS, os pacotes de gerenciamento a seguir serão instalados no Operations Manager.  Esses pacotes de gerenciamento também são instalados em computadores com Windows conectados diretamente após a adição dessa solução. Não há nada para configurar ou gerenciar com esses pacotes de gerenciamento. 

* Microsoft System Center Advisor Update Assessment Intelligence Pack (Microsoft.IntelligencePacks.UpdateAssessment)
* Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
* MP de Implantação de Atualizações

Para saber mais sobre como os pacotes de gerenciamento da solução são atualizados, veja [Conectar o Operations Manager ao Log Analytics](../log-analytics/log-analytics-om-agents.md).

### <a name="hybrid-worker-groups"></a>Grupos de Hybrid Worker
Depois que você habilita essa solução, qualquer computador com Windows conectado diretamente a seu espaço de trabalho do OMS é automaticamente configurado como um Hybrid Runbook Worker para dar suporte aos runbooks incluídos nessa solução.  Para cada computador Windows gerenciado pela solução, ele será listado sob a folha Hybrid Runbook Worker Groups da conta de Automação seguindo a convenção de nomenclatura *Hostname FQDN_GUID*.  Não é possível direcionar esses grupos com runbooks em sua conta, caso contrário, haverá falha. Esses grupos devem dar suporte à solução de gerenciamento.   

No entanto, você pode adicionar os computadores com Windows a um grupo de Hybrid Runbook Worker em sua conta de Automação para dar suporte a runbooks de automação enquanto você estiver usando a mesma conta para a solução e para a associação de grupo do Hybrid Runbook Worker.  Essa funcionalidade foi adicionada à versão 7.2.12024.0 do Hybrid Runbook Worker.  

## <a name="configuration"></a>Configuração
Execute as etapas a seguir para adicionar a solução de gerenciamento de atualizações a seu espaço de trabalho do OMS e confirme os agentes estão se comunicando. Os agentes do Windows já conectados ao seu espaço de trabalho são adicionados automaticamente sem nenhuma configuração adicional. 

Você pode implantar a solução usando os seguintes métodos:

* A partir do Azure Marketplace no portal do Azure, selecionando a oferta de automação e Controle ou uma solução de Gerenciamento de Atualizações
* A partir da Galeria de Soluções do OMS no espaço de trabalho do OMS

Se já tiver uma conta de Automação e o espaço de trabalho do OMS vinculados no mesmo grupo de recursos e região, selecionar Automação e Controle verificará sua configuração e apenas instalará a solução e a configurará em ambos os serviços.  Selecionar a solução de Gerenciamento de Atualizações do Azure Marketplace fornece o mesmo comportamento.  Se não tiver um dos serviços implantados em sua assinatura, siga as etapas na folha **Criar nova Solução** e confirme que deseja instalar as outras soluções pré-selecionadas recomendadas.  Opcionalmente, você pode adicionar a solução de Gerenciamento de Atualizações a seu espaço de trabalho do OMS usando as etapas descritas em [Adicionar soluções do OMS](../log-analytics/log-analytics-add-solutions.md) da Galeria de Soluções.  

### <a name="confirm-oms-agents-and-operations-manager-management-group-connected-to-oms"></a>Confirme os agentes do OMS e o grupo de gerenciamento do Operations Manager conectados ao OMS

Para confirmar se Agentes de OMS para Linux e Windows diretamente conectados estão se comunicando com o OMS, depois de alguns minutos, você pode executar a seguinte pesquisa de log:

* Linux - `Type=Heartbeat OSType=Linux | top 500000 | dedup SourceComputerId | Sort Computer | display Table`.  

* Windows - `Type=Heartbeat OSType=Windows | top 500000 | dedup SourceComputerId | Sort Computer | display Table`

Em um computador Windows, você pode examinar o seguinte para verificar a conectividade do agente com o OMS:

1.  Abra o Microsoft Monitoring Agent no Painel de Controle e, na guia **Análise de Log do Azure (OMS)**, o agente exibe uma mensagem dizendo: **O Microsoft Monitoring Agent se conectou com êxito ao serviço Microsoft Operations Management Suite**.   
2.  Abra o Log de Eventos do Windows, navegue até **Logs de Aplicativos e Serviços\Operations Manager** e procure as IDs de Evento 3000 e 5002 do Conector de Serviço de origem.  Esses eventos indicam que o computador foi registrado com o espaço de trabalho do OMS e está recebendo a configuração.  

Se o agente puder se comunicar com o serviço OMS e estiver configurado para se comunicar com a Internet através de um servidor proxy ou firewall, verifique se o servidor proxy ou firewall está configurado corretamente examinando [Definir configurações de proxy e firewall na Análise de Log](../log-analytics/log-analytics-proxy-firewall.md).
  
Agentes do Linux recém-adicionados mostrarão um status de **Atualizado** após ter sido realizada uma avaliação.  Esse processo pode levar até seis horas. 

Para confirmar se um grupo de gerenciamento do Operations Manager está se comunicando com o OMS, confira [Validar a integração do Operations Manager com o OMS](../log-analytics/log-analytics-om-agents.md#validate-operations-manager-integration-with-oms).

## <a name="data-collection"></a>Coleta de dados
### <a name="supported-agents"></a>Agentes com suporte
A tabela a seguir descreve as fontes conectadas que têm suporte dessa solução.

| Fonte Conectada | Suportado | Descrição |
| --- | --- | --- |
| Agentes do Windows |Sim |A solução coleta informações sobre atualizações do sistema de agentes do Windows e inicia a instalação de atualizações necessárias. |
| Agentes do Linux |Sim |A solução coleta informações sobre atualizações do sistema de agentes Linux e inicia a instalação das atualizações necessárias em distribuições com suporte. |
| Grupo de gerenciamento do Operations Manager |Sim |A solução coleta informações sobre atualizações do sistema de agentes em um grupo de gerenciamento conectados.<br>Uma conexão direta do agente do Operations Manager ao Log Analytics não é necessária. Os dados são encaminhados do grupo de gerenciamento para o repositório do OMS. |
| Conta de Armazenamento do Azure |Não |O armazenamento do Azure não inclui informações sobre atualizações do sistema. |

### <a name="collection-frequency"></a>Frequência de coleta
Para cada computador gerenciado do Windows, uma verificação é executada duas vezes por dia. A cada 15 minutos, a API do Windows é chamada para consultar a hora da última atualização para determinar se o status for alterado e, nesse caso, é iniciada uma verificação de conformidade.  Para cada computador Linux gerenciado, uma verificação é executada a cada três horas. 

Pode demorar de 30 minutos a seis horas para o painel exibir dados atualizados em computadores gerenciados.   

## <a name="using-the-solution"></a>Usando a solução
Ao adicionar a solução de Gerenciamento de Atualizações ao seu espaço de trabalho do OMS, o bloco **Gerenciamento de Atualizações** será adicionado ao painel do OMS. Esse bloco exibe uma contagem e representação gráfica do número de computadores em seu ambiente e sua conformidade de atualização.<br><br>
![Bloco de resumo do Gerenciamento de Atualizações](media/oms-solution-update-management/update-management-summary-tile.png)  


## <a name="viewing-update-assessments"></a>Exibição de avaliações de atualização
Clique no bloco **Gerenciamento de Atualizações** para abrir o painel **Gerenciamento de Atualizações**.<br><br> ![Painel Resumo de Gerenciamento de Atualizações](./media/oms-solution-update-management/update-management-dashboard.png)<br> 

Esse painel fornece uma análise detalhada do status de atualização categorizado por tipo de sistema operacional e a classificação da atualização: crítica, de segurança ou outros (como uma atualização de definição). Quando selecionado, o bloco **Implantações de Atualização** redireciona você para a página de implantações de atualização, onde é possível exibir agendas, implantações em execução no momento e implantações concluídas ou agendar uma nova implantação.  

Você pode executar uma pesquisa de log que retorna todos os registros clicando no bloco específico ou executando uma consulta de uma categoria específica e critérios predefinidos. Selecione um na lista disponível na coluna **Consultas Comuns de Atualização**.    

## <a name="installing-updates"></a>Instalação de atualizações
Depois de avaliar atualizações para todos os computadores com Windows e Linux em seu espaço de trabalho, você pode ter as necessárias atualizações instaladas, criando uma *Implantação de Atualizações*.  Uma Implantação de Atualizações é uma instalação agendada de atualizações necessárias para um ou mais computadores.  Você especifica a data e hora para a implantação, além de um computador ou um grupo de computadores que devem ser incluídos no escopo de uma implantação.  Para saber mais sobre grupos de computadores, confira [Grupos de computadores na Análise de Log](../log-analytics/log-analytics-computer-groups.md).  Quando você inclui grupos de computadores em sua implantação de atualização, a associação de grupo é avaliada apenas uma vez no momento da criação da agenda.  As alterações subsequentes em um grupo não são refletidas.  Para solucionar esse problema, exclua a implantação de atualização agendada e recrie-a. 

> [!NOTE]
> Máquinas virtuais do Windows implantadas no Azure Marketplace por padrão são definidas para receber atualizações automáticas do Serviço Windows Update.  Esse comportamento não é alterado depois de adicionar essa solução ou máquinas virtuais do Windows ao seu espaço de trabalho.  Se você não gerenciou atualizações ativamente com essa solução, o comportamento padrão (aplicar automaticamente as atualizações) será aplicado.  

As máquinas virtuais criadas por meio das imagens do RHEL (Red Hat Enterprise Linux) sob demanda disponíveis no Azure Marketplace são registradas para acessar a [RHUI (Infraestrutura de Atualização do Red Hat)](../virtual-machines/virtual-machines-linux-update-infrastructure-redhat.md) implantada no Azure.  Qualquer distribuição do Linux deve ser atualizada nos repositórios de distribuição de arquivo online seguindo os métodos com suporte.  

### <a name="viewing-update-deployments"></a>Exibição de implantações de atualização
Clique no bloco **Implantação de Atualização** para exibir a lista das Implantações de Atualizações existentes.  Elas são agrupadas por status – **Agendadas**, **Em Execução** e **Concluídas**.<br><br> ![Página de Agendamento de Implantações de Atualizações](./media/oms-solution-update-management/update-updatedeployment-schedule-page.png)<br>  

As propriedades exibidas para cada Implantação de Atualizações são descritas na tabela a seguir.

| Propriedade | Descrição |
| --- | --- |
| Nome |Nome da Implantação de Atualizações. |
| Agenda |Tipo de agenda.  As opções disponíveis são *Uma Vez*, *Recorrente Semanal* ou *Recorrente Mensal*. |
| Hora de Início |A data e a hora em que a Implantação de Atualizações está agendada para iniciar. |
| Duração |O número de minutos que a Implantação de Atualização pode ser executada.  Se todas as atualizações não forem instaladas nesse período, as atualizações restantes deverão aguardar até a próxima Implantação de Atualização. |
| Servidores |O número de computadores afetados pela Implantação de Atualizações.  |
| Status |O status atual da Implantação de Atualizações.<br><br>Os valores possíveis são:<br>- Não iniciado<br>- Executando<br>- Concluído |

Selecione uma Implantação de Atualização concluída para exibir a tela de detalhes que inclui as colunas na tabela a seguir.  Essas colunas não estarão preenchidas se a Implantação de Atualizações ainda não tiver sido iniciada.<br><br> ![Visão geral dos resultados da Implantação de Atualizações](./media/oms-solution-update-management/update-management-deploymentresults-dashboard.png)

| Coluna | Descrição |
| --- | --- |
| **Exibição de Computadores** | |
| Computadores com Windows |Lista o número de computadores com Windows na Implantação de Atualizações por status.  Clique em um status para executar uma pesquisa de log que retorna todos os registros de atualização com esse status para a Implantação de Atualizações. |
| Computadores Linux |Lista o número de computadores com Linux na Implantação de Atualizações por status.  Clique em um status para executar uma pesquisa de log que retorna todos os registros de atualização com esse status para a Implantação de Atualizações. |
| Status de Instalação do Computador |Lista os computadores envolvidos na Implantação de Atualizações e a porcentagem de atualizações instaladas com êxito. Clique em uma das entradas para executar uma pesquisa de log retornando todas as atualizações críticas e ausentes. |
| **Exibição de Atualizações** | |
| Atualizações do Windows |Lista as atualizações do Windows incluídas na implantação de atualização e o status de instalação por atualização.  Selecione uma atualização para executar uma pesquisa de log que retorna todos os registros de atualização para essa atualização específica ou clique no status para executar uma pesquisa de log que retorna todos os registros de atualização para a implantação. | 
| Atualizações do Linux |Listar atualizações do Linux incluídas na implantação de atualização e o status de instalação por atualização.  Selecione uma atualização para executar uma pesquisa de log que retorna todos os registros de atualização para essa atualização específica ou clique no status para executar uma pesquisa de log que retorna todos os registros de atualização para a implantação. | 

### <a name="creating-an-update-deployment"></a>Criação de uma Implantação de Atualizações
Crie uma nova Implantação de Atualizações clicando no botão **Adicionar** na parte superior da tela para abrir a página **Nova Implantação de Atualizações**.  Você deve fornecer valores para as propriedades na tabela a seguir.

| Propriedade | Descrição |
| --- | --- |
| Nome |Nome exclusivo para identificar a Implantação de Atualizações. |
| Fuso horário |Fuso horário a ser usado para a hora de início. |
| Tipo de Agenda | Tipo de agenda.  As opções disponíveis são *Uma Vez*, *Recorrente Semanal* ou *Recorrente Mensal*.  
| Hora de Início |Data e hora para iniciar a Implantação de Atualizações. **Observação:** o momento mais antecipado em que uma implantação pode ser executada é 30 minutos a contar da hora atual, se você precisa implantar imediatamente. |
| Duração |O número de minutos que a Implantação de Atualização pode ser executada.  Se todas as atualizações não forem instaladas nesse período, as atualizações restantes deverão aguardar até a próxima Implantação de Atualização. |
| Computadores |Nomes de computadores ou grupos para incluir e direcionar na Implantação de Atualizações.  Selecione uma ou mais entradas na lista suspensa. |

<br><br> ![Página Nova Implantação de Atualizações](./media/oms-solution-update-management/update-newupdaterun-page.png)

### <a name="time-range"></a>Intervalo de tempo
Por padrão, o escopo dos dados analisados na solução de Gerenciamento de Atualizações é de todos os grupos de gerenciamento conectados, gerados no último dia. 

Para alterar o intervalo de tempo dos dados, selecione **Dados baseados em** na parte superior do painel. Você pode selecionar registros criados ou atualizados dentro dos últimos 7 dias, 1 dia ou 6 horas. Outra opção é selecionar **Personalizado** e especificar um intervalo de datas personalizado.

## <a name="log-analytics-records"></a>Registros do Log Analytics
A solução de gerenciamento de atualização cria dois tipos de registros no repositório do OMS.

### <a name="update-records"></a>Registros de atualização
Um registro com um tipo **Update** é criado para cada atualização instalada ou necessária em cada computador. Os registros Update têm as propriedades descritas na tabela a seguir.

| Propriedade | Descrição |
| --- | --- |
| Tipo |*Atualização* |
| SourceSystem |A fonte que aprovou a instalação da atualização.<br>Os valores possíveis são:<br>- Microsoft Update<br>- Windows Update<br>- SCCM<br>- Servidores Linux (buscados de Gerenciadores de Pacotes) |
| Aprovado |Especifica se a atualização foi aprovada para instalação.<br> Para os servidores Linux, atualmente isso é opcional, já que a aplicação de patch não é gerenciada pelo OMS. |
| Classificação para o Windows |Classificação da Atualização.<br>Os valores possíveis são:<br>- Aplicativos<br>- Atualizações Críticas<br>- Atualizações de Definição<br>- Feature Packs<br>- Atualizações de Segurança<br>- Service Packs<br>- Pacotes Cumulativos de Atualização<br>- Atualizações |
| Classificação para Linux |Classificação da atualização.<br>Os valores possíveis são:<br>- Atualizações Críticas<br>- Atualizações de Segurança<br>- Outras Atualizações |
| Computador |Nome do computador. |
| InstallTimeAvailable |Especifica se o tempo de instalação está disponível em outros agentes que instalaram a mesma atualização. |
| InstallTimePredictionSeconds |Tempo estimado para instalação em segundos, com base em outros agentes que instalaram a mesma atualização. |
| KBID |ID do artigo da Base de Dados de Conhecimento que descreve a atualização. |
| ManagementGroupName |Nome do grupo de gerenciamento de agentes do SCOM.  Para outros agentes, é o AOI-<workspace ID>. |
| MSRCBulletinID |ID do boletim de segurança da Microsoft que descreve a atualização. |
| MSRCSeverity |Gravidade do boletim de segurança da Microsoft.<br>Os valores possíveis são:<br>- Crítico<br>- Importante<br>- Moderado |
| Opcional |Especifica se a atualização é opcional. |
| Produto |Nome do produto ao qual a atualização se destina.  Clique em **Exibir** para abrir o artigo em um navegador. |
| PackageSeverity |A gravidade da vulnerabilidade corrigida nesta atualização, conforme reportado pelos fornecedores de distribuição do Linux. |
| PublishDate |Data e hora em que a atualização foi instalada. |
| RebootBehavior |Especifica se a atualização forçará uma reinicialização.<br>Os valores possíveis são:<br>- canrequestreboot<br>- neverreboots |
| RevisionNumber |Número de revisão da atualização. |
| SourceComputerId |GUID para identificar exclusivamente o computador. |
| TimeGenerated |Data e hora em que o registro foi atualizado. |
| Title |Título da atualização. |
| UpdateID |GUID para identificar exclusivamente a atualização. |
| UpdateState |Especifica se a atualização está instalada neste computador.<br>Os valores possíveis são:<br>-Instalado - a atualização está instalada neste computador.<br>-Necessário - a atualização não está instalada e é necessário neste computador. |

Quando você executa uma pesquisa de log que retorna registros com um tipo **Update**, pode selecionar a exibição **Updates**, que exibe um conjunto de blocos resumindo as atualizações retornadas pela pesquisa. Você pode clicar nas entradas dos blocos **Atualizações ausentes e aplicadas** e **Atualizações obrigatórias e opcionais** para obter um escopo da exibição do conjunto de atualizações. Selecione a exibição **Lista** ou **Tabela** para retornar os registros individuais.<br> 

![Exibição de atualização de pesquisa de log com a atualização do tipo de registro](./media/oms-solution-update-management/update-la-view-updates.png)  

Na exibição **Tabela**, você pode clicar na **KBID** de qualquer registro para abrir um navegador com o artigo da Base de Dados de Conhecimento. Isso permite que você leia rapidamente sobre os detalhes da atualização específica.<br> 

![Exibição de tabela de pesquisa de log com atualizações do tipo de registro de blocos](./media/oms-solution-update-management/update-la-view-table.png)

Na exibição **Lista**, clique no link **Exibir** ao lado da KBID para abrir o artigo da Base de Dados de Conhecimento.<br>

![Exibição de lista de pesquisa de log com atualizações do tipo de registro de blocos](./media/oms-solution-update-management/update-la-view-list.png)

### <a name="updatesummary-records"></a>Registros de UpdateSummary
Um registro com um tipo **UpdateSummary** é criado para cada computador de agente do Windows. Esse registro é atualizado toda vez que o computador é examinado em busca de atualizações. Os registros **UpdateSummary** têm as propriedades descritas na tabela a seguir.

| Propriedade | Descrição |
| --- | --- |
| Tipo |UpdateSummary |
| SourceSystem |OpsManager |
| Computador |Nome do computador. |
| CriticalUpdatesMissing |Número de atualizações críticas ausentes no computador. |
| ManagementGroupName |Nome do grupo de gerenciamento de agentes do SCOM. Para outros agentes, é o AOI-<workspace ID>. |
| NETRuntimeVersion |Versão do tempo de execução .NET instalada no computador. |
| OldestMissingSecurityUpdateBucket |Bucket para categorizar o tempo desde a segurança ausente mais antiga atualização neste computador foi publicado.<br>Os valores possíveis são:<br>- Mais antigos<br>- Há 180 dias<br>- Há 150 dias<br>- Há 120 dias<br>- Há 90 dias<br>- Há 60 dias<br>- Há 30 dias<br>- Recente |
| OldestMissingSecurityUpdateInDays |Número de dias desde que a atualização de segurança ausente mais antiga neste computador foi publicada. |
| OsVersion |Versão do sistema operacional instalado no computador. |
| OtherUpdatesMissing |Número de outras atualizações ausentes no computador. |
| SecurityUpdatesMissing |Número de atualizações de segurança ausentes no computador. |
| SourceComputerId |GUID para identificar exclusivamente o computador. |
| TimeGenerated |Data e hora em que o registro foi atualizado. |
| TotalUpdatesMissing |Número total de atualizações ausentes no computador. |
| WindowsUpdateAgentVersion |Número de versão do agente do Windows Update no computador. |
| WindowsUpdateSetting |Configuração de como o computador instalará as atualizações importantes.<br>Os valores possíveis são:<br>- Desabilitado<br>- Notificar antes da instalação<br>- Instalação agendada |
| WSUSServer |URL do servidor WSUS se o computador estiver configurado para usar um. |

## <a name="sample-log-searches"></a>Pesquisas de log de exemplo
A tabela a seguir fornece pesquisas de log de exemplo para os registros de atualização coletados por essa solução. 

| Consultar | Descrição |
| --- | --- |
|Computadores baseados no Windows Server que precisam de atualizações |'Type:Update OSType!=Linux UpdateState=Needed Optional=false Approved!=false | measure count() by Computer' |
|Servidores Linux que precisam de atualizações | 'Type:Update OSType=Linux UpdateState!="Not needed" | measure count() by Computer' |
| Todos os computadores com atualizações ausentes |'Type=Update UpdateState=Needed Optional=false | selecione Computer,Title,KBID,Classification,UpdateSeverity,PublishedDate` |
| Atualizações ausentes para um computador específico (substitua o valor pelo nome de seu próprio computador) |'Type=Update UpdateState=Needed Optional=false Computer="COMPUTER01.contoso.com" | selecione Computer,Title,KBID,Product,UpdateSeverity,PublishedDate' |
| Todos os computadores com atualizações críticas ou de segurança ausentes |`Type=Update UpdateState=Needed Optional=false (Classification="Security Updates" OR Classification="Critical Updates"`) |
| Atualizações críticas ou de segurança necessárias para computadores em que as atualizações são aplicadas manualmente |'Type=Update UpdateState=Needed Optional=false (Classification="Security Updates" OR Classification="Critical Updates") Computer IN {Type=UpdateSummary WindowsUpdateSetting=Manual | Distinct Computer} | Distinct KBID' |
| Eventos de erro para computadores sem as atualizações críticas ou de segurança obrigatórias |'Type=Event EventLevelName=error Computer IN {Type=Update (Classification="Security Updates" OR Classification="Critical Updates") UpdateState=Needed Optional=false | Distinct Computer}' |
| Todos os computadores com pacotes cumulativos de atualização ausentes |'Type=Update Optional=false Classification="Update Rollups" UpdateState=Needed| selecione Computer,Title,KBID,Classification,UpdateSeverity,PublishedDate` |
| Atualizações ausentes distintas em todos os computadores |'Type=Update UpdateState=Needed Optional=false | Distinct Title' |
| Computador baseado no Windows Server com atualizações que falharam em uma execução de atualização | 'Type:UpdateRunProgress InstallationStatus=failed | measure count() by Computer, Title, UpdateRunName' |
| Servidor Linux com atualizações que falharam em uma execução de atualização |'Type:UpdateRunProgress InstallationStatus=failed | measure count() by Computer, Product, UpdateRunName' |
| Associação de computadores ao WSUS |'Type=UpdateSummary | measure count() by WSUSServer' |
| Configuração de atualização automática |'Type=UpdateSummary | measure count() by WindowsUpdateSetting' |
| Computadores com atualizações automáticas desabilitadas |`Type=UpdateSummary WindowsUpdateSetting=Manual` |
| Lista de todas as máquinas Linux com uma atualização de pacote disponível |'Type=Update and OSType=Linux and UpdateState!="Not needed" | measure count() by Computer' |
| Lista de todas as máquinas Linux com uma atualização de pacote disponível que trata de vulnerabilidade Crítica ou de Segurança |'Type=Update and OSType=Linux and UpdateState!="Not needed" and (Classification="Critical Updates" OR Classification="Security Updates") | measure count() by Computer' |
| Lista de todos os pacotes que tenham uma atualização disponível |Type=Update and OSType=Linux and UpdateState!="Not needed" |
| Lista de todos os pacotes que tenham uma atualização disponível para uma vulnerabilidade Crítica ou de Segurança |`Type=Update  and OSType=Linux and UpdateState!="Not needed" and (Classification="Critical Updates" OR Classification="Security Updates")` |
| Listar as implantações de atualização que modificaram computadores |'Type:UpdateRunProgress | measure Count() by UpdateRunName' |
|Computadores que foram atualizados nesta atualização executam (substitua o valor pelo nome de sua implantação de atualização |'Type:UpdateRunProgress UpdateRunName="DeploymentName" | measure Count() by Computer' |
| Lista de todas as máquinas "Ubuntu" com qualquer atualização disponível |'Type=Update and OSType=Linux and OSName = Ubuntu &| measure count() by Computer' |

## <a name="troubleshooting"></a>Solucionar problemas 

Esta seção fornece informações para ajudar a solucionar problemas da solução de Gerenciamento de Atualizações.  

### <a name="how-do-i-troubleshoot-update-deployments"></a>Como solucionar problemas de implantações de atualização?
Você pode exibir os resultados do runbook responsável por implantar as atualizações incluídas na implantação de atualização agendada na folha Trabalhos de sua conta de Automação que está vinculada ao espaço de trabalho do OMS que dá suporte a essa solução.  O runbook **MicrosoftOMSComputer Patch** é um runbook filho voltado para um computador gerenciado específico, e a análise do fluxo detalhado apresentará informações detalhadas dessa implantação.  A saída exibirá quais atualizações necessárias são aplicáveis, o status de download, o status da instalação e detalhes adicionais.<br><br> ![Atualizar status de trabalho de Implantação](media/oms-solution-update-management/update-la-patchrunbook-outputstream.png)<br>

Para obter mais informações, confira [Mensagens e saída de runbook de automação](../automation/automation-runbook-output-and-messages.md).   
  
## <a name="next-steps"></a>Próximas etapas
* Use Pesquisas de Log no [Log Analytics](../log-analytics/log-analytics-log-searches.md) para exibir dados detalhados das atualizações.
* [Crie seus próprios painéis](../log-analytics/log-analytics-dashboards.md) mostrando a conformidade da atualização dos computadores gerenciados.
* [Crie alertas](../log-analytics/log-analytics-alerts.md) quando atualizações críticas forem detectadas como ausentes de um computador ou quando um computador tiver as atualizações automáticas desabilitadas.  


