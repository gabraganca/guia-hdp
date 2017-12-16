# Guia para Certificação HDPCD da Hortonworks

O material abaixo foi inspirado na guia de objetivos fornecido pela Hortonworks
em seu [site][site], e ainda está em fase de elaboração.


[site]: https://br.hortonworks.com/services/training/certification/exam-objectives/#hdpcd

## Objetivos

1. [Ingestão de Dados](#ingestão-de-dados)
   * [Importação de tabela de um RDBMS para o HDFS usando o Sqoop](#importe-dados-de-uma-tabela-em-uma-base-de-dados-relacional-para-o-hdfs)
   * [Importe os resultados de uma *query* a um banco de dados para o HDFS](#importe-os-resultados-de-uma-query-a-um-banco-de-dados-para-o-hdfs)
   * [Importe uma tabela de um banco de dados relacional para uma tabela do Hive](#importe-uma-tabela-de-um-banco-de-dados-relacional-para-uma-tabela-do-hive)
   * [Insira ou atualize dados do HDFS para um tabela de uma banco de dados relacional](#insira-ou-atualize-dados-do-hdfs-para-um-tabela-de-uma-banco-de-dados-relacional)
   * [Inicie um agente do Flume a partir de um arquivo de configuração](#inicie-um-agente-do-flume-a-partir-de-um-arquivo-de-configura%C3%A7%C3%A3o)
   * [Configure um `channel` de memória com um tamanho específico](#configure-um-channel-de-mem%C3%B3ria-com-um-tamanho-espec%C3%ADfico)
2. Transformação de Dados
  * Escreva execute um script do Pig
3. Análise de Dados

### Ingestão de Dados

#### Importe dados de uma tabela em uma base de dados relacional para o HDFS

Para isso, nós devemos usar o [`sqoop import`][SQOOP-IMPORT] e sempre devemos
usar o argumento `--connect` que vai permitir que o `sqoop` se conecte ao banco
de dados. Por exemplo:
```
$ sqoop import \
      --connect jdbc:mysql://database.example.com/employees
```

No exemplo acima, o `sqoop` se conectará a uma base de dados MySQL nomeada
`employees` na máquina `host` `database.example.com`.

Podemos listar o nome de todas as base de dados disponíveis com `sqoop
list-databases` e as tabelas na base de dados usando o `sqoop list-tables`.

Na importação, o nome da tabela deve ser informada usando o argumento
`--table`.  Caso deseje, é possível importar todas as tabelas do banco de dados
usando `sqoop import--all-tables`.

É provável que seja necessário fornecer um nome de usuário (`--username`) e
senha para o `sqoop`. Para o nome de usuários, nós usamos `--username` e para a
senha, nós usamos diretamente via `--password` ou usando um arquivo com o
`--password-file` (mais seguro).

O `sqoop` nos permite salvar a tabela no HDFS nos formatos
* Texto (padrão): `--as-textfile`
* Avro: `--as-avrodatafile`
* SequenceFiles: `--as-sequencefile`
* Parquet: `--as-parquetfile`

Vamos ver um exemplo de como importar uma tabela chamada `vendas` de um banco
de dados MySQL chamado `ecommerce.db` para dentro do HDFS not formato texto:

```
$ sqoop import \
      --connect jdbc:mysql://database.example.com/ecommerce.db \
      --username fulano \
      --password 123456 \
      --table vendas \
      --as-textfile
```

Note que o último argumento, `--as-textfile`, não é obrigatório já que o
`sqoop` importa para formato texto por padrão. Uma pasta chamada `vendas` será
criada na pasta do usuário no HDFS.

Para maiores informações, recomendo a leitura da [documentação][SQOOP-IMPORT].

[SQOOP-IMPORT]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_literal_sqoop_import_literal

#### Importe os resultados de uma *query* a um banco de dados para o HDFS

Podemos realizar uma *query* em um banco de dados relacional e importar o
resultado direto para o HDFS. Para isso, nós precisaremos usar dois argumentos:

* `--query`, em que passamos a `query` para o `sqoop`; e
* `--target-dir`, em que passamos o diretório no HDFS em que os dados serão
  gravados.

Por exemplo:

```
$ sqoop import \
      --connect jdbc:mysql://database.example.com/ecommerce.db \
      --username fulano \
      --password 123456 \
      --query 'SELECT vendas.*, fornecedores.* FROM vendas JOIN fornecedores on (vendas.forn_id == fornecedores.id) WHERE vendas.preco > 5000' \
      --target-dir /user/fulano/meus-resultados
```

A [documentação][FREE-FORM QUERY IMPORTS] mostra alguns argumentos extras para
casos especiais.

[FREE-FORM QUERY IMPORTS]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_free_form_query_imports


#### Importe uma tabela de um banco de dados relacional para uma tabela do Hive

Para importar uma tabela direto para o Hive, nós podemos usar os seguintes
argumentos:

* `--hive-import`: este parâmetro é obrigatório.
* `--create-hive-table`: opcional. Cria a tabela caso esta não exista. Se
  existir, retorna um erro.
* `--hive-overwrite`: opcional. Sobrescreve uma tabela caso esta já exista.
* `--hive-table <db_name>.<table_name>`: opcional. Informa em qual banco de
  dados e em qual tabela salvar.

Vamos ver um exemplo:

```
$ sqoop import \
      --connect jdbc:mysql://database.example.com/ecommerce.db \
      --username fulano \
      --password 123456 \
      --table vendas \
      --hive-import \
      --create-hive-table \
      --hive-table ecommerce.vendas
```

Para maiores detalhes, recomendo a [documentação][IMPORTING DATA INTO HIVE] e
[este tutorial][sqoop-hive].

[IMPORTING DATA INTO HIVE]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_importing_data_into_hive
[sqoop-hive]: https://dzone.com/articles/sqoop-import-data-from-mysql-to-hive


#### Insira ou atualize dados do HDFS para um tabela de uma banco de dados relacional

Além de inserir tabelas de um banco de dados relaciona para o HDFS, o `sqoop`
também nos permite fazer o processo contrário. vamos ver agora como exportar
uma tabela do HDFS para um banco de dados.

Ao invés do `import` nós devemos suar o `export`, e alguns dos argumentos são os mesmos, tais quais
`--connect <jdbc-uri>`, `--username` e `--password`. Vamos ver um exemplo:

```
$ sqoop export \
      --connect jdbc:mysql://database.example.com/ecommerce.db \
      --username fulano \
      --password 123456 \
      --table vendas \
      --export-dir /user/fulano/vendas
```

O comando acima vai exportar uma tabela localizada no diretório
`/user/fulano/vendas` no HDFS para uma banco de dados relacional MySQL chamado
`ecommerce.db`.

O comando `export` aceita outros argumentos e recomendo a
[documentação][SQOOP-EXPORT] para uma melhor explicaçação.

[SQOOP-EXPORT]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_literal_sqoop_export_literal


#### Inicie um agente do Flume a partir de um arquivo de configuração

Nós iniciamos um agente do Flume através do terminal. Para isso

```
$ flume-ng agent --name nome_do_agente --conf /etc/flume/conf --conf-file flume.conf
```

O comando `agent` inicia o agente coletor do Flume. Vamos ver agora os
argumentos passados para o Flume:

* `--name` (`-n`): nome do agente (obrigatório):
* `--conf` (`-c`): aponta o diretório onde Flume encontrará seus principais
  configurações.
* `conf-file` (`-f`): caminho para o arquivo de configuração do Flume onde o
  agente está definido.

Para maiores detalhes, recomendo a [documentação oficial][FLUME AGENT] e [este
guia][horton_flume_guia] da Hortonworks que mostra como começar o agente no
HDP.

[FLUME AGENT]: (https://flume.apache.org/FlumeUserGuide.html#starting-an-agent
[horton_flume_guia]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_command-line-installation/content/ch_installing_flume_chapter.html


#### Configure um `channel` de memória com um tamanho específico

Um `channel` é um repositório onde os eventos capturados pelo `course` são
armazenados até que o `sink` os removam. Um `channel` de removam é o tipo mais
básico de `channel` e, abaixo, mostramos uma configuração típica, considerando
que um `source` e um `sink` já foram configurados.

```
a1.channels = c1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 10000
a1.channels.c1.byteCapacity = 800000
a1.channels.c1.byteCapacityBufferPercentage = 20
```

Vamos ver esta configuração linha a linha:
1. Um `channel` de nome `a1` é definido para o agente `a1`.
2. O tipo é definido, `memory`.
3. Definimos a capacidade do `channel` que, neste exemplo, é de 10000 eventos.
4. Nesta linha, definimos a capacidade máxima de eventos que o `channel` vai
   passar do `source` para o `sink`. Neste caso, são 10000 eventos por
   transação.
5. Quantidade total de memória, em bytes, que é a soma de todos os eventos
   armazenados. Vale notar que, da forma que o `channel` de memória foi
   implementado, o `byteCapacity` corresponde apenas aos dados do `body` do
   evento.
6. Nesta última linha, definimos a a quantidade de memória para o cabeçalho dos
   eventos. O valor é a porcentagem do *buffer* entre o `byteCapacity` e a soma
   estimada de todos os eventos que, neste caso, foi definida como 20%.

O exemplo acima foi retirado da [documentação][MEMORY CHANNEL] que também
exemplifica os outros tipos de `channels`.

[MEMORY CHANNEL]: (https://flume.apache.org/FlumeUserGuide.html#memory-channel


### Transformação de Dados

#### Escreva a execute um script do Pig

O Pig é uma ferramenta de alto nível para processamento de dados. Ele nos
permite processar dados de forma mais prática do que se fôssemos usar
diretamente o *MapReduce*.

Nós podemos processar dados através um um ambiente REPL
(*Read-Eval-Print-Loop*) ichamado *Grunt* ou através de execução de scripts
criados no nosso editor de preferência. Vamos ver aqui um exemplo simples de
script e como executá-lo, mas saiba que é possível rodar o script linha a linha
no ambiente REPL.

Vamos ver um script simples obtido da [documentação][pig_run]:

```
/* meu_script.pig
Meu script é simples.
É composto apenas de três declarações
*/

A = LOAD 'vendas' USING PigStorage() AS (item:chararray, preco:float, qtde:int); -- carregando os dados
B = FOREACH A GENERATE item;  -- transformando os dados
DUMP B;  -- obtendo os resultados
```

As quatro primeiras linhas são um comentário. Na sexta linha, nós carregamos
uma tabela chamada `vendas` em que três colunas são definidas: `item` definida
como uma *string*, `preco` definida como um número do tipo *float* e `qtde`
definida como um número inteiro. Na sétima linha, executamos um *loop* na
tabela carregada em `A` e executamos uma operação que, neste caso, é só
retornar o valor de `item`. Na última linha, nós retornamos o resultado da
relação `B`.

É importante salientar, que no Pig, `A` e `B` não são chamados de variáveis,
mas de relações.

Para executar este script, nós devemos executar o seguinte commando:

```
$ pig meu_script.pig
```

[pig_run]: https://pig.apache.org/docs/r0.15.0/start.html#run


#### Load data into a Pig relation without a schema

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#load)


#### Load data into a Pig relation with a schema

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#load)


#### Load data from a Hive table into a Pig relation

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore)


#### Use Pig to transform data into a specified format

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#foreach)


#### Transform data to match a given Hive schema

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#foreach)


#### Group the data of one or more Pig relations

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#group)


#### Use Pig to remove records with null values from a relation

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#filter)


#### Store the data from a Pig relation into a folder in HDFS

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#store)


#### Store the data from a Pig relation into a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore)


#### Sort the output of a Pig relation

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#order-by)


#### Remove the duplicate tuples of a Pig relation

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#distinct)


#### Specify the number of reduce tasks for a Pig MapReduce job

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/perf.html#parallel)


#### Join two datasets using Pig

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#join-outer)


#### Perform a replicated join using Pig

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/perf.html#replicated-joins)


#### Run a Pig job using Tez

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/perf.html#tez-mode)


#### Within a Pig script, register a JAR file of User Defined Functions

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/udf.html#piggybank)


#### Within a Pig script, define an alias for a User Defined Function

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#define-udfs)


#### Within a Pig script, invoke a User Defined Function

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#register)


### Análise de Dados

#### Write and execute a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/Tutorial)


#### Define a Hive-managed table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-Create/Drop/TruncateTable)


#### Define a Hive external table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ExternalTables)


#### Define a partitioned Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables)


#### Define a bucketed Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-BucketedSortedTables)


#### Define a Hive table from a select query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTableAsSelect(CTAS))


#### Define a Hive table that uses the ORCFile format

  [LEARN MORE](https://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/)


#### Create a new ORCFile table from the data in an existing non-ORCFile Hive table

  [LEARN MORE](https://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/)


#### Specify the storage format of a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RowFormat,StorageFormat,andSerDe)


#### Specify the delimiter of a Hive table

  [LEARN MORE](https://hortonworks.com/hadoop-tutorial/using-hive-data-analysis/)


#### Load data into a Hive table from a local directory

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Loadingfilesintotables)


#### Load data into a Hive table from an HDFS directory

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Loadingfilesintotables)


#### Load data into a Hive table as the result of a query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingdataintoHiveTablesfromqueries)


#### Load a compressed data file into a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage)


#### Update a row in a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Update)


#### Delete a row from a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Delete)


#### Insert a new row into a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingvaluesintotablesfromSQL)


#### Join two Hive tables

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Joins)


#### Run a Hive query using Tez

  [LEARN MORE](https://hortonworks.com/hadoop-tutorial/supercharging-interactive-queries-hive-tez/)


#### Run a Hive query using vectorization

  [LEARN MORE](https://hortonworks.com/hadoop-tutorial/supercharging-interactive-queries-hive-tez/)


#### Output the execution plan for a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Explain)


#### Use a subquery within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries)


#### Output data from a Hive query that is totally ordered across multiple reducers

  [LEARN MORE](https://issues.apache.org/jira/browse/HIVE-1402)


#### Set a Hadoop or Hive configuration property from within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Configuration#AdminManualConfiguration-ConfiguringHive)


#### Output the execution plan for a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Explain)


#### Use a subquery within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries)


#### Output data from a Hive query that is totally ordered across multiple reducers

  [LEARN MORE](https://issues.apache.org/jira/browse/HIVE-1402)


#### Set a Hadoop or Hive configuration property from within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Configuration#AdminManualConfiguration-ConfiguringHive)
