---
title: Espelhar o cluster Apache Kafka no HDInsight | Microsoft Docs
description: "Saiba como usar o recurso de espelhamento do Kafka para manter uma réplica de um cluster Kafka no HDInsight espelhando tópicos para um cluster secundário."
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: 015d276e-f678-4f2b-9572-75553c56625b
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 02/13/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 8c4e33a63f39d22c336efd9d77def098bd4fa0df
ms.openlocfilehash: c7517f61944b9fdb02a3589d7c9cd83355dae6d8
ms.lasthandoff: 04/20/2017

---
# <a name="use-mirrormaker-to-create-a-replica-of-a-kafka-on-hdinsight-cluster-preview"></a>Usar o MirrorMaker para criar uma réplica de um cluster Kafka no HDInsight (visualização)

O Apache Kafka inclui um recurso de espelhamento, que permite replicar tópicos de um cluster Kafka para outro. Por exemplo, replicar registros entre clusters Kafka em diferentes regiões do Azure.

O espelhamento pode ser executado como um processo contínuo ou usado de forma intermitente como um método de migração de dados de um cluster para outro.

> [!WARNING]
> O espelhamento não deve ser considerado um meio de obter tolerância a falhas. O deslocamento para itens em um tópico é diferente entre os clusters de origem e de destino. Assim, os clientes não podem usar os dois intercambiavelmente.
> 
> Se estiver preocupado com a tolerância a falhas, você deverá definir a replicação para os tópicos no cluster. Para obter mais informações, confira [Introdução ao Kafka no HDInsight](hdinsight-apache-kafka-get-started.md).

## <a name="prerequisites"></a>Pré-requisitos

* Uma Rede Virtual do Azure: os clusters Kafka de origem e destino devem ser capazes de se comunicar diretamente entre si. O HDInsight não expõem APIs Kafka publicamente na Internet. Portanto, os clusters de origem e de destino devem existir na mesma Rede Virtual do Azure.

* Dois clusters Kafka: este documento usa um modelo do Azure Resource Manager para criar dois clusters Kafka no HDInsight em uma Rede Virtual do Azure.

## <a name="how-does-mirroring-work"></a>Como o espelhamento funciona?

O espelhamento funciona usando a ferramenta MirrorMaker (parte do Apache Kafka) para consumir registros de tópicos no cluster de origem e criar uma cópia local no cluster de destino. O MirrorMaker usa um (ou mais) *consumidores* que leem do cluster de origem e um *produtor* que grava no cluster local (destino).

O seguinte diagrama ilustra o processo de espelhamento:

![Diagrama do processo de espelhamento](./media/hdinsight-apache-kafka-mirroring/kafka-mirroring.png)

Os clusters de origem e de destino podem ser diferentes no número de nós e partições, e os deslocamentos nos tópicos também são diferentes. O espelhamento mantém o valor de chave que é usado para particionamento. Assim, a ordem de registros é preservada por chave.

### <a name="mirroring-between-networks"></a>Espelhamento entre redes

Se você precisa de espelhamento entre clusters Kafka em redes diferentes, há as seguintes considerações adicionais:

* **Gateways**: as redes devem ser capazes de se comunicar no nível de TCP/IP.

* **Resolução de nomes**: os clusters Kafka em cada rede devem ser capazes de se conectar entre si usando nomes de host. Isso pode exigir um servidor DNS (Sistema de Nomes de Domínio) em cada rede configurado para encaminhar solicitações para outras redes. 
  
    Ao criar uma Rede Virtual do Azure, em vez de usar o DNS automático fornecido com a rede, você deve especificar um servidor DNS personalizado e o endereço IP do servidor. Depois que a Rede Virtual for criada, você deverá criar uma Máquina Virtual do Azure que use esse endereço IP e instalar e configurar o software DNS nela.
  
    > [!WARNING]
    > Crie e configure o servidor DNS personalizado antes de instalar o HDInsight na Rede Virtual. Não é necessária configuração adicional para que o HDInsight use o servidor DNS configurado para a Rede Virtual.

Para obter mais informações sobre como conectar duas Redes Virtuais do Azure, confira [Configurar uma conexão de VNet para VNet](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md).

## <a name="create-kafka-clusters"></a>Criar clusters Kafka

O Apache Kafka no HDInsight não fornece acesso ao serviço Kafka pela Internet pública. Qualquer item que se comunique com o Kafka deve estar na mesma rede virtual do Azure que os nós no cluster Kafka. Neste exemplo, os clusters Kafka de origem e destino estão localizados em uma rede virtual do Azure. O seguinte diagrama mostra como a comunicação flui entre os clusters:

![Diagrama de clusters Kafka de origem e destino em uma rede virtual do Azure](./media/hdinsight-apache-kafka-mirroring/spark-kafka-vnet.png)

> [!NOTE]
> Embora o Kafka em si esteja limitado à comunicação na rede virtual, outros serviços no cluster, como SSH e Ambari, podem ser acessados pela Internet. Para obter mais informações sobre as portas públicas disponíveis com o HDInsight, confira [Portas e URIs usados pelo HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Embora você possa criar uma rede virtual do Azure e clusters Kafka manualmente, é mais fácil usar um modelo do Azure Resource Manager. Use as etapas a seguir para implantar uma rede virtual do Azure e dois clusters Kafka para sua assinatura do Azure.

1. Use o botão a seguir para entrar no Azure e abra o modelo do Gerenciador de Recursos no portal do Azure.
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-kafka-mirror-cluster-in-vnet.json" target="_blank"><img src="./media/hdinsight-apache-kafka-mirroring/deploy-to-azure.png" alt="Deploy to Azure"></a>
   
    O modelo do Gerenciador de Recursos está localizado em **https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-mirror-cluster-in-vnet.json**.

2. Use as seguintes informações para preencher as entradas na folha de **Implantação personalizada** :
    
    ![Implantação personalizada do HDInsight](./media/hdinsight-apache-kafka-mirroring/parameters.png)
    
    * **Grupo de recursos**: Crie um grupo ou selecione um existente. Esse grupo contém o cluster HDInsight.

    * **Local**: escolha um local geograficamente perto de você. Esse local deve corresponder ao local na seção __CONFIGURAÇÕES__.
     
    * **Nome do Cluster de base**: esse valor é usado como o nome de base para os clusters Kafka. Por exemplo, inserir **hdi** cria clusters chamados **source-hdi** e **dest-hdi**.

    * **Nome de Usuário de Logon do Cluster**: o nome de usuário de administrador para os clusters Kafka de origem e destino.

    * **Senha de Logon do Cluster**: a senha de usuário de administrador para os clusters Kafka de origem e destino.

    * **Nome de Usuário SSH**: o usuário SSH a ser criado para os clusters Kafka de origem e destino.

    * **Senha SSH**: a senha para o usuário SSH para os clusters Kafka de origem e destino.

    * **Local**: a região na qual os clusters são criados.

3. Leia **Termos e Condições**, e depois selecione **Concordo com os termos e condições declarados acima**.

4. Por fim, marque **Fixar no painel** e selecione **Comprar**. Demora cerca de 20 minutos para criar os clusters.

Quando os recursos tiverem sido criados, você será redirecionado para uma folha do grupo de recursos que contém os clusters e o painel da Web.

![Folha do grupo de recursos para rede virtual e clusters](./media/hdinsight-apache-kafka-mirroring/groupblade.png)

> [!IMPORTANT]
> Observe que os nomes dos clusters HDInsight são **source-BASENAME** e **dest-BASENAME**, em que BASENAME é o nome fornecido para o modelo. Você usa esses nomes em etapas posteriores ao se conectar aos clusters.

## <a name="create-topics"></a>Criar tópicos

1. Conecte-se ao cluster de **origem** usando o SSH:
   
        ssh sshuser@source-BASENAME-ssh.azurehdinsight.net
   
    Substitua**sshuser** pelo nome de usuário do SSH usado ao criar o cluster. Substitua **BASENAME** pelo nome base usado ao criar o cluster.
   
    Para saber mais, confira [Usar SSH com HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Use o seguinte comando para localizar os hosts do Zookeeper, defina a variável `SOURCE_ZKHOSTS` e crie vários tópicos novos chamados `testtopic`:
   
    ```bash
    SOURCE_ZKHOSTS=`grep -R zk /etc/hadoop/conf/yarn-site.xml | grep 2181 | grep -oPm1 "(?<=<value>)[^<]+"`
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $SOURCE_ZKHOSTS
    ```

3. Use o seguinte comando para verificar se o tópico foi criado:
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $SOURCE_ZKHOSTS
    ```
   
    A resposta contém `testtopic`.

4. Use o seguinte para exibir as informações de host do Zookeeper para esse cluster (a **origem**):
   
    ```bash
    echo $SOURCE_ZKHOSTS
    ```
   
 Isso retorna informações semelhantes ao seguinte texto:
   
       zk0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:2181,zk1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:2181,zk6-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:2181
   
 Salve essas informações. Ele é usado na próxima seção.

## <a name="configure-mirroring"></a>Configurar o espelhamento

1. Conecte-se ao cluster de **destino** usando uma sessão de SSH diferente:
   
        ssh sshuser@dest-BASENAME-ssh.azurehdinsight.net
   
    Substitua**sshuser** pelo nome de usuário do SSH usado ao criar o cluster. Substitua **BASENAME** pelo nome base usado ao criar o cluster.
   
    Para saber mais, confira [Usar SSH com HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Use o seguinte comando para criar um arquivo `consumer.properties` que descreve como se comunicar com o cluster de **origem**:
   
    ```bash
    nano consumer.properties
    ```
   
    Use o seguinte texto como o conteúdo do arquivo `consumer.properties`:
   
        zookeeper.connect=SOURCE_ZKHOSTS
        group.id=mirrorgroup
   
    Substitua **SOURCE_ZKHOSTS** pelas informações de hosts de Zookeeper do cluster de **origem**.
   
    Esse arquivo descreve as informações de consumidor a serem usadas ao ler do cluster Kafka de origem. Para obter mais informações de configuração do consumidor, confira [Configurações de Consumidor](https://kafka.apache.org/documentation#consumerconfigs) em kafka.apache.org.
   
    Use **Ctrl + X**, **Y** e Enter para salvar o arquivo.

3. Antes de configurar o produtor que se comunica com o cluster de destino, você deve localizar os hosts agentes para o cluster de **destino**. Use os seguintes comandos para recuperar essas informações:
   
    ```bash
    sudo apt -y install jq
    DEST_BROKERHOSTS=`sudo bash -c 'ls /var/lib/ambari-agent/data/command-[0-9]*.json' | tail -n 1 | xargs sudo cat | jq -r '["\(.clusterHostInfo.kafka_broker_hosts[]):9092"] | join(",")'`
    echo $DEST_BROKERHOSTS
    ```
   
    Esses comandos retornarão informações semelhantes ao seguinte:
   
        wn0-dest.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn1-dest.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092

4. Use o seguinte para criar um arquivo `producer.properties` que descreve como se comunicar com o cluster de **destino**:

    ```bash
    nano producer.properties
    ```

    Use o seguinte texto como o conteúdo do arquivo `producer.properties`:
   
        bootstrap.servers=DEST_BROKERS
        compression.type=none
   
    Substitua **DEST_BROKERS** pelas informações de agente da etapa anterior. 
   
    Para obter mais informações de configuração produtor, confira [Configurações de Produtor](https://kafka.apache.org/documentation#producerconfigs) em kafka.apache.org.

## <a name="start-mirrormaker"></a>Iniciar MirrorMaker

1. Na conexão SSH ao cluster de **destino**, use o seguinte comando para iniciar o processo MirrorMaker:
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-run-class.sh kafka.tools.MirrorMaker --consumer.config consumer.properties --producer.config producer.properties --whitelist testtopic --num.streams 4
    ```
   
    Os parâmetros usados neste exemplo são:
   
    * **-consumer.config**: especifica o arquivo que contém as propriedades do consumidor. As propriedades são usadas para criar um consumidor que lê do cluster Kafka de *origem*.

    * **-producer.config**: especifica o arquivo que contém as propriedades de produtor. As propriedades são usadas para criar um produtor que grava o cluster Kafka de *destino*.

    * **–whitelist**: uma lista de tópicos que o MirrorMaker replica do cluster de origem para o destino.
    
    * **-num.streams**: o número de threads de consumidor a serem criados.
     
    Na inicialização, MirrorMaker retorna informações semelhantes ao seguinte texto:
        
    ```
    {metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-3, security.protocol=PLAINTEXT}{metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-0, security.protocol=PLAINTEXT}
    metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-kafka.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-2, security.protocol=PLAINTEXT}
    metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-1, security.protocol=PLAINTEXT}
    ```

2. Na conexão SSH para o cluster **de origem**, use o seguinte comando para iniciar um produtor e enviar mensagens para o tópico:
    
    ```bash
    sudo apt -y install jq
    SOURCE_BROKERHOSTS=`sudo bash -c 'ls /var/lib/ambari-agent/data/command-[0-9]*.json' | tail -n 1 | xargs sudo cat | jq -r '["\(.clusterHostInfo.kafka_broker_hosts[]):9092"] | join(",")'`
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $SOURCE_BROKERHOSTS --topic testtopic
    ```

 Quando você chega a uma linha em branco com um cursor, digite algumas mensagens de texto. Elas são enviadas para o tópico no cluster **fonte**. Depois de concluído, use **Ctrl + C** para finalizar o processo de produtor.

3. Na conexão SSH ao cluster de **destino**, use **Ctrl + C** para encerrar o processo MirrorMaker. Em seguida, use os comandos a seguir para verificar se o tópico `testtopic` foi criado, e esses dados no tópico foram replicados para esse espelho:
    
    ```bash
    DEST_ZKHOSTS=`grep -R zk /etc/hadoop/conf/yarn-site.xml | grep 2181 | grep -oPm1 "(?<=<value>)[^<]+"`
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $DEST_ZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --zookeeper $DEST_ZKHOSTS --topic testtopic --from-beginning
    ```
    
  A lista de tópicos agora inclui `testtopic`, que é criado quando MirrorMaster espelha o tópico do cluster de origem para o destino. As mensagens recuperadas do tópico são as mesmas inseridas no cluster de origem.

## <a name="delete-the-cluster"></a>Excluir o cluster

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Como as etapas neste documento criam ambos os clusters no mesmo grupo de recursos do Azure, você pode excluir o grupo de recursos no portal do Azure. A exclusão do grupo de recursos remove todos os recursos criados por este documento, a Rede Virtual do Azure e a conta de armazenamento usada pelos clusters.

## <a name="next-steps"></a>Próximas etapas

Neste documento, você aprendeu a usar o MirrorMaker para criar uma réplica de um cluster Kafka. Use os links a seguir para descobrir outras maneiras de trabalhar com Kafka:

* [Documentação do Apache Kafka MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) em cwiki.apache.org.
* [Introdução ao Apache Kafka no HDInsight](hdinsight-apache-kafka-get-started.md)
* [Usar o Apache Spark com o Kafka no HDInsight](hdinsight-apache-spark-with-kafka.md)
* [Usar Apache Storm com Kafka no HDInsight](hdinsight-apache-storm-with-kafka.md)
* [Conectar-se ao Kafka por meio de uma Rede Virtual do Azure](hdinsight-apache-kafka-connect-vpn-gateway.md)

