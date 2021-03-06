---
title: Usar o Beeline para trabalhar com o Hive no HDInsight (Hadoop) | Microsoft Docs
description: "Aprenda a usar o SSH para se conectar a um cluster Hadoop no HDInsight e enviar interativamente consultas do Hive usando o Beeline. Beeline é um utilitário para trabalhar com HiveServer2 sobre JDBC."
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 3adfb1ba-8924-4a13-98db-10a67ab24fca
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 04/05/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 785d3a8920d48e11e80048665e9866f16c514cf7
ms.openlocfilehash: 7a1757a7ef2881b9e09389745a8e5aeaea2abf9d
ms.lasthandoff: 04/12/2017


---
# <a name="use-hive-with-hadoop-in-hdinsight-with-beeline"></a>Usar o Hive com o Hadoop no HDInsight com Beeline
[!INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

Saiba como usar o [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) para executar consultas Hive no HDInsight.

O Beeline é uma ferramenta de linha de comando que está incluída em nós de cabeçalho do cluster HDInsight. Ele usa o JDBC para se conectar ao HiveServer2, um serviço hospedado no seu cluster HDInsight. A tabela a seguir fornece as cadeias de caracteres de conexão para uso com o Beeline:

| De que local você executa o Beeline | Cadeia de conexão | Outros parâmetros |
| --- | --- | --- |
| Uma conexão SSH a um nó de cabeçalho | `jdbc:hive2://localhost:10001/;transportMode=http` | `-n admin` |
| Um nó de borda | `jdbc:hive2://headnodehost:10001/;transportMode=http` | `-n admin` |
| Fora do cluster | `jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2` | `-n admin -p password` |

> [!NOTE]
> Substitua 'admin' com a conta de logon do cluster para o cluster.
>
> Substitua 'password' pela senha da conta de logon do cluster.
>
> Substitua `clustername` pelo nome do cluster HDInsight.

## <a id="prereq"></a>Pré-requisitos

* Um cluster do Hadoop no HDInsight baseado em Linux.

  > [!IMPORTANT]
  > O Linux é o único sistema operacional usado no HDInsight versão 3.4 ou superior. Para obter mais informações, consulte [Substituição do HDInsight versão 3.2 e 3.3](hdinsight-component-versioning.md#hdi-version-33-nearing-deprecation-date).

* Um cliente SSH. Para saber mais sobre como usar SSH, confira [Usar SSH com HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="ssh"></a>Conexão com o SSH

Conecte-se ao cluster usando o SSH com o seguinte comando:

```bash
ssh sshuser@myhdinsight-ssh.azurehdinsight.net
```

Substitua `sshuser` pela conta de usuário SSH para o cluster. Substitua `myhdinsight` pelo nome do cluster HDInsight.

**Se você tiver fornecido uma senha para autenticação SSH** ao criar o cluster HDInsight, você precisará fornecer a senha quando solicitado.

**Se você tiver fornecido uma chave de certificado para autenticação de SSH** ao criar o cluster HDInsight, talvez seja necessário especificar o local da chave privada no sistema cliente:

    ssh -i ~/.ssh/mykey.key ssh@myhdinsight-ssh.azurehdinsight.net

Para saber mais sobre como usar o SSH com HDInsight, confira [Usar SSH com HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="beeline"></a>Usar o comando Beeline

1. Na sessão de SSH, use o seguinte comando para iniciar o Beeline:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin
    ```

    Esse comando inicia o cliente Beeline e conecta-se ao HiveServer2 no nó de cabeçalho do cluster. O parâmetro `-n` é usado para fornecer a conta de logon do cluster. O logon padrão é `admin`. Se durante a criação do cluster você usou um nome diferente, use-o no lugar de `admin`.

    Quando o comando for concluído, você chegará a um prompt `jdbc:hive2://localhost:10001/>`.

2. Os comandos Beeline normalmente começam com um caractere `!`, por exemplo, `!help` exibe a ajuda. No entanto, o `!` pode ser omitido para alguns comandos. Por exemplo, `help` também funciona.

    Há um `!sql`, que é usado para executar instruções HiveQL. No entanto, o HiveQL é tão usado que é possível omitir o `!sql`anterior. As duas instruções a seguir são equivalentes:

    ```hiveql
    !sql show tables;
    show tables;
    ```

    Em um novo cluster, somente uma tabela é listada: **hivesampletable**.

3. Use o comando a seguir para exibir o esquema para a hivesampletable:

    ```bash
    describe hivesampletable;
    ```

    Esse comando retorna as informações a seguir:

        +-----------------------+------------+----------+--+
        |       col_name        | data_type  | comment  |
        +-----------------------+------------+----------+--+
        | clientid              | string     |          |
        | querytime             | string     |          |
        | market                | string     |          |
        | deviceplatform        | string     |          |
        | devicemake            | string     |          |
        | devicemodel           | string     |          |
        | state                 | string     |          |
        | country               | string     |          |
        | querydwelltime        | double     |          |
        | sessionid             | bigint     |          |
        | sessionpagevieworder  | bigint     |          |
        +-----------------------+------------+----------+--+

    Essas informações descrevem as colunas na tabela. Embora possamos executar algumas consultas nestes dados, vamos criar uma nova tabela para demonstrarmos como carregar os dados no Hive e como aplicar um esquema.

4. Insira as instruções a seguir para criar uma tabela chamada **log4jLogs** usando os dados de exemplo fornecidos com o cluster HDInsight:

    ```hiveql
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
    ```

    As instruções executam as seguintes ações:

    * **DROP TABLE** – se a tabela existir, ela será excluída.

    * **CREATE EXTERNAL TABLE** – cria uma tabela **externa** em Hive. Tabelas externas só armazenam a definição da tabela no Hive. Os dados são mantidos no local original.

    * **ROW FORMAT** – o modo como os dados são formatados. Nesse caso, os campos em cada log são separados por um espaço.

    * **STORED AS TEXTFILE LOCATION** – o local em que os dados são armazenados e em qual formato de arquivo.

    * **SELECT** - Seleciona uma contagem de todas as linhas em que a coluna **t4** contém o valor **[ERROR]**. Essa consulta deve retornar um valor de **3**, já que existem três linhas que contêm esse valor.

    * **INPUT__FILE__NAME LIKE '%.log'** – o arquivo de dados de exemplo é armazenado com outros arquivos. Essa instrução limita a consulta aos dados armazenados em arquivos que terminam em .log.

  > [!NOTE]
  > As tabelas externas devem ser usadas quando você espera que os dados subjacentes sejam atualizados por uma fonte externa. Por exemplo, um processo de upload de dados automatizado ou uma operação MapReduce.
  >
  > Remover uma tabela externa **não** exclui os dados, somente a definição de tabela.

    A saída desse comando é semelhante ao texto a seguir:

        INFO  : Tez session hasn't been created yet. Opening session
        INFO  :

        INFO  : Status: Running (Executing on YARN cluster with App id application_1443698635933_0001)

        INFO  : Map 1: -/-      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
        INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
        INFO  : Map 1: 1/1      Reducer 2: 0/1
        INFO  : Map 1: 1/1      Reducer 2: 0(+1)/1
        INFO  : Map 1: 1/1      Reducer 2: 1/1
        +----------+--------+--+
        |   sev    | count  |
        +----------+--------+--+
        | [ERROR]  | 3      |
        +----------+--------+--+
        1 row selected (47.351 seconds)

5. Para sair do Beeline, use `!exit`.

## <a id="file"></a>Executar um arquivo HiveQL

Use as etapas a seguir para criar um arquivo e  executá-lo usando o Beeline.

1. Use o comando a seguir para criar um novo arquivo chamado **query.hql**:

    ```bash
    nano query.hql
    ```

2. Use o texto a seguir como conteúdo do arquivo. Essa consulta cria uma nova tabela 'interna' chamada **errorLogs**:

    ```hiveql
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    As instruções executam as seguintes ações:

    * **CREATE TABLE IF NOT EXISTS** – criará uma tabela, se ela ainda não existir. Uma vez que a palavra-chave **EXTERNAL** não é usada, essa instrução cria uma tabela interna. As tabelas internas são armazenadas no data warehouse do Hive e totalmente gerenciadas por ele.
    * **STORES AS ORC** : armazena os dados no formato ORC (Optimized Row Columnar). O formato ORC é altamente otimizado e eficiente para o armazenamento de dados do Hive.
    * **INSERT OVERWRITE ... SELECT** - seleciona linhas da tabela **log4jLogs** que contêm **[ERROR]** e insere os dados na tabela **errorLogs**.

    > [!NOTE]
    > Diferentemente de tabelas externas, o descarte de uma tabela interna excluirá também os dados subjacentes.

3. Para salvar o arquivo, use **Ctrl**+**_X**, insira **Y** e, por fim, **Enter**.

4. Use o seguinte para executar o arquivo usando Beeline. Substitua **HOSTNAME** pelo nome obtido anteriormente para o nó do cabeçalho, e **PASSWORD** pela senha da conta de administrador:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin -i query.hql
    ```

    > [!NOTE]
    > O parâmetro `-i` inicia o Beeline e executa as instruções no arquivo query.hql. Quando a consulta for concluída, você verá um prompt `jdbc:hive2://localhost:10001/>`. Você também pode executar um arquivo usando o parâmetro `-f`, que fechará o Beeline após a conclusão da consulta.

5. Para verificar se a tabela **errorLogs** foi criada, use a seguinte instrução para retornar todas as linhas de **errorLogs**:

    ```hiveql
    SELECT * from errorLogs;
    ```

    Três linhas de dados devem ser devolvidas, todas contendo **[ERROR]** na coluna t4:

        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        | errorlogs.t1  | errorlogs.t2  | errorlogs.t3  | errorlogs.t4  | errorlogs.t5  | errorlogs.t6  | errorlogs.t7  |
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        | 2012-02-03    | 18:35:34      | SampleClass0  | [ERROR]       | incorrect     | id            |               |
        | 2012-02-03    | 18:55:54      | SampleClass1  | [ERROR]       | incorrect     | id            |               |
        | 2012-02-03    | 19:25:27      | SampleClass4  | [ERROR]       | incorrect     | id            |               |
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        3 rows selected (1.538 seconds)

## <a name="edge-nodes"></a>Nós de borda

Se o cluster tem um nó de borda, é recomendável usar sempre o nó de borda em vez do nó de cabeçalho durante a conexão com o SSH. Para iniciar o Beeline de uma conexão SSH em um nó de borda, use o seguinte comando:

```bash
beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -n admin
```

## <a name="remote-clients"></a>Clientes remotos

Se você tiver Beeline instalado localmente ou são usá-lo por meio de uma imagem de Docker como [sutoiku/beeline](https://hub.docker.com/r/sutoiku/beeline/), você deverá usar os seguintes parâmetros:

* __Cadeia de conexão__: `-u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2'`

* __Nome de logon do cluster__: `-n admin`

* __Senha de logon do cluster__ `-p 'password'`

Substitua o `clustername` na cadeia de conexão pelo nome do seu cluster HDInsight.

Substitua `admin` pelo nome de logon do cluster e substitua `password` pela senha para o logon do cluster.

## <a id="summary"></a><a id="nextsteps"></a>Próximas etapas

Para obter mais informações gerais sobre como usar o Hive no HDInsight, veja o seguinte documento:

* [Usar o Hive com Hadoop no HDInsight](hdinsight-use-hive.md)

Para saber mais sobre outras maneiras pelas quais você pode trabalhar com o Hadoop no HDInsight, veja os seguintes documentos:

* [Usar o Pig com Hadoop no HDInsight](hdinsight-use-pig.md)
* [Usar o MapReduce com Hadoop no HDInsight](hdinsight-use-mapreduce.md)

Se você estiver usando o Tez com o Hive, consulte os seguintes documentos:

* [Usar a interface de usuário do Tez no HDInsight baseado em Windows](hdinsight-debug-tez-ui.md)
* [Usar a exibição de Ambari Tez no HDInsight baseado em Linux](hdinsight-debug-ambari-tez-view.md)

[hdinsight-sdk-documentation]: http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://en.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: http://azure.microsoft.com/documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md

[putty]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html


[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md
[hdinsight-upload-data]: hdinsight-upload-data.md


[powershell-here-strings]: http://technet.microsoft.com/library/ee692792.aspx

