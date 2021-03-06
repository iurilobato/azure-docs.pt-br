---
title: "Visão geral do serviço de banco de dados relacional Banco de Dados do Azure para PostgreSQL | Microsoft Docs"
description: "Fornece uma visão geral do serviço de banco de dados relacional Banco de Dados do Azure para PostgreSQL."
services: postgresql
author: kamathsun
ms.author: sukamat
manager: jhubbard
editor: jasonh
ms.assetid: 
ms.service: postgresql-database
ms.tgt_pltfrm: portal
ms.topic: overview
ms.date: 05/10/2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 8aae9994f8fc6bb8e4e5368a1495dd257be0417f
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017

---
# <a name="what-is-azure-database-for-postgresql"></a>O que é o Banco de Dados do Azure para PostgreSQL?

O Banco de Dados do Azure para PostgreSQL é um serviço de banco de dados relacional no Microsoft Cloud, projetado para desenvolvedores com base na versão de comunidade do mecanismo de banco de dados [PostgreSQL](https://www.postgresql.org/) de software livre. Esse serviço está na fase de visualização pública. O Banco de Dados do Azure para PostgreSQL fornece:
- Desempenho previsível em vários níveis de serviço
- Escalabilidade dinâmica sem tempo de inatividade do aplicativo
- Alta disponibilidade interna
- Proteção de dados

Todos esses recursos não precisam de quase nenhuma administração e todos são fornecidos sem nenhum custo adicional. Esses recursos permitem que você se concentre no método RAD e acelere seu tempo de colocação no mercado, em vez de alocar tempo e recursos preciosos para gerenciamento de máquinas virtuais e infraestrutura. Além disso, você pode continuar desenvolvendo seu aplicativo com a plataforma e as ferramentas de software livre de sua escolha e pode fornecê-lo com a velocidade e a eficiência que sua empresa exige, sem precisar aprender novas habilidades. 

Este artigo é uma introdução aos principais conceitos e recursos do Banco de Dados do Azure para PostgreSQL relacionados a desempenho, escalabilidade e gerenciabilidade. Consulte estes inícios rápidos para começar:

- [Criar um servidor de Banco de Dados do Azure para PostgreSQL usando o portal do Azure](quickstart-create-server-database-portal.md)
- [Criar um servidor de Banco de Dados do Azure para PostgreSQL usando a CLI Azure](quickstart-create-server-database-azure-cli.md)

Para ver vários exemplos da CLI do Azure e do PowerShell, consulte:

- [Exemplos da CLI do Azure para o Banco de Dados do Azure para PostgreSQL](./sample-scripts-azure-cli.md)

## <a name="adjust-performance-and-scale-without-downtime"></a>Ajuste de desempenho e dimensionamento sem tempo de inatividade

O serviço de Banco de Dados do Azure para PostgreSQL atualmente oferece duas camadas de serviço: Básico e Standard. Cada camada de serviço oferece [diferentes níveis de desempenho, garantias de IOPS e recursos](concepts-service-tiers.md) para dar suporte a cargas de trabalho leves e pesadas de banco de dados. Você pode criar seu primeiro aplicativo em um servidor pequeno por alguns dólares por mês e, em seguida, [alterar o nível de desempenho](scripts/sample-scale-server-up-or-down.md) na camada de serviço manualmente ou por meio de programação a qualquer momento para atender às necessidades de sua solução. Você pode fazer isso sem tempo de inatividade para seu aplicativo ou seus clientes. A escalabilidade dinâmica permite que o banco de dados responda de forma transparente às mudanças rápidas de requisitos de recursos e que você pague apenas pelos recursos de que precisa, quando precisar deles.

## <a name="monitoring-and-alerting"></a>Monitoramento e alertas

Como você decide quando aumentar e reduzir? Use os recursos internos de alerta e monitoramento de desempenho, em conjunto com as classificações de desempenho baseadas na Unidade de computação. Usando essas ferramentas, você pode avaliar rapidamente o impacto da expansão ou redução das Unidades de computação com base nas suas necessidades de desempenho atuais ou previstas. Para obter detalhes, consulte [Opções e desempenho do Banco de Dados do Azure para PostgreSQL: compreender o que está disponível em cada camada de serviço](./concepts-service-tiers.md).

## <a name="keep-your-app-and-business-running"></a>Mantenha seus aplicativos e a continuidade dos negócios

O SLA (Contrato de Nível de Serviço) de disponibilidade de 99,99% do Azure (indisponível na versão prévia), que é líder do setor e é alimentado por uma rede global de datacenters gerenciados pela Microsoft, ajuda a manter seu aplicativo em execução de forma ininterrupta. Com cada servidor do Banco de Dados do Azure para PostgreSQL, você tira proveito dos recursos internos de segurança, tolerância a falhas e proteção de dados que, em outras situações, seria necessário comprar ou projetar, criar e gerenciar. Com o Banco de Dados do Azure para PostgreSQL, cada camada de serviço oferece um conjunto abrangente de opções e recursos de continuidade de negócios que você pode usar para voltar à execução e permanecer assim. Você pode usar a [restauração para um ponto específico](howto-restore-server-portal.md) para retornar um banco de dados para um estado anterior, até 35 dias. Além disso, se o datacenter que hospeda seus bancos de dados sofrer uma interrupção, você poderá restaurar os bancos de dados de cópias de backups recentes com redundância geográfica.

## <a name="secure-your-data"></a>Proteja seus dados

Os serviços de banco de dados do Azure têm uma tradição de segurança de dados que o Banco de Dados do Azure para PostgreSQL mantém, com recursos que limitam o acesso, protegem dados em repouso e em movimento e ajudam a monitorar atividades. Visite a [Central de Confiabilidade do Azure](https://www.microsoft.com/TrustCenter/Security/default.aspx) para obter informações sobre a segurança da plataforma do Azure.

## <a name="next-steps"></a>Próximas etapas
- Consultar a [página de preços](https://azure.microsoft.com/pricing/details/postgresql/) para ver comparações de custo e calculadoras.
- Comece pela [criação do seu primeiro Banco de Dados do Azure para PostgreSQL](./quickstart-create-server-database-portal.md).
- Crie seu primeiro aplicativo no Python, PHP, Ruby, C\#, Java, Node.js: [Bibliotecas de conexão](./concepts-connection-libraries.md)

