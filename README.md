# Guia para Certificação HDPCD da Hortonworks

O material abaixo foi inspirado na guia de objetivos fornecido pela Hortonworks
em seu [site][site].


[site]: https://br.hortonworks.com/services/training/certification/exam-objectives/#hdpcd

## Objetivos

1. [Ingestão de Dados](#ingestão-de-dados)
   * [Importação de tabela de um RDBMS para o HDFS usando o Sqoop](#importe-dados-de-uma-tabela-em-uma-base-de-dados-relacional-para-o-hdfs)
   * [Importe os resultados de uma *query* a um banco de dados para o HDFS](#importe-os-resultados-de-uma-query-a-um-banco-de-dados-para-o-hdfs)
   * [Importe uma tabela de um banco de dados relacional para uma tabela do Hive](#importe-uma-tabela-de-um-banco-de-dados-relacional-para-uma-tabela-do-hive)
2. [Data Transformation](#data-transformation)
3. [Data Analysis](#data-analysis)

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
      --hive-import
      --create-hive-table
      --hive-table ecommerce.vendas
```

Para maiores detalhes, recomendo a [documentação][IMPORTING DATA INTO HIVE] e
[este tutorial][sqoop-hive].

[IMPORTING DATA INTO HIVE]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_importing_data_into_hive
[sqoop-hive]: https://dzone.com/articles/sqoop-import-data-from-mysql-to-hive


#### Insert or update data from HDFS into a table in a relational database

  [SQOOP-EXPORT](http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_literal_sqoop_export_literal)


#### Given a Flume configuration file, start a Flume agent

  [FLUME AGENT](https://flume.apache.org/FlumeUserGuide.html#starting-an-agent)


#### Given a configured sink and source, configure a Flume memory channel with a specified capacity

  [MEMORY CHANNEL](https://flume.apache.org/FlumeUserGuide.html#memory-channel)


### Data Transformation

#### Write and execute a Pig script

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/start.html#run)


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


### Data Analysis

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
