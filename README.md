# Guia para Certificação HDPCD da Hortonworks

O material abaixo foi inspirado na guia de objetivos fornecido pela Hortonworks
em seu [site][site].

Este é um trabalho em elaboração. As seguintes etapas precisam ser feitas:

* Os itens em inglês precisam ser descritos.
* Revisão completa do texto.


[site]: https://br.hortonworks.com/services/training/certification/exam-objectives/#hdpcd

## Sumário

1. [Ingestão de Dados](#ingestão-de-dados)
   * [Importação de tabela de um RDBMS para o HDFS usando o Sqoop](#importe-dados-de-uma-tabela-em-uma-base-de-dados-relacional-para-o-hdfs)
   * [Importe os resultados de uma *query* a um banco de dados para o HDFS](#importe-os-resultados-de-uma-query-a-um-banco-de-dados-para-o-hdfs)
   * [Importe uma tabela de um banco de dados relacional para uma tabela do Hive](#importe-uma-tabela-de-um-banco-de-dados-relacional-para-uma-tabela-do-hive)
   * [Insira ou atualize dados do HDFS para um tabela de uma banco de dados relacional](#insira-ou-atualize-dados-do-hdfs-para-um-tabela-de-uma-banco-de-dados-relacional)
   * [Inicie um agente do Flume a partir de um arquivo de configuração](#inicie-um-agente-do-flume-a-partir-de-um-arquivo-de-configura%C3%A7%C3%A3o)
   * [Configure um `channel` de memória com um tamanho específico](#configure-um-channel-de-mem%C3%B3ria-com-um-tamanho-espec%C3%ADfico)
2. [Transformação de Dados](#transforma%C3%A7%C3%A3o-de-dados)
   * [Escreva execute um script do Pig](#escreva-a-execute-um-script-do-pig)
   * [Carregue dados para uma relação do Pig sem definir um esquema](#carregue-dados-para-uma-rela%C3%A7%C3%A3o-do-pig-sem-definir-um-esquema)
   * [Carregue dados para uma relação do Pig definindo um esquema](#carregue-dados-para-uma-rela%C3%A7%C3%A3o-do-pig-definindo-um-esquema)
   * [Carregue dados de uma tabela do Hive para uma relação do Pig](#carregue-dados-de-uma-tabela-do-hive-para-uma-rela%C3%A7%C3%A3o-do-pig)
   * [Use o Pig para transformar dados para um formato específico](#use-o-pig-para-transformar-dados-para-um-formato-espec%C3%ADfico)
   * [Transforme os dados para um esquema pŕe-definido do Hive](#transforme-os-dados-para-um-esquema-p%C5%95e-definido-do-hive)
   * [Agrupe os dados em uma ou mais relações do Pig](#agrupe-os-dados-em-uma-ou-mais-rela%C3%A7%C3%B5es-do-pig)
   * [Use o Pig para remover valores ausentes em uma relação](#use-o-pig-para-remover-valores-ausentes-em-uma-rela%C3%A7%C3%A3o)
   * [Armazene os dados de uma relação no Pig em uma pasta no HDFS](#armazene-os-dados-de-uma-rela%C3%A7%C3%A3o-no-pig-em-uma-pasta-no-hdfs)
   * [Armazene os dados de uma relação no Pig em uma tabela do Hive](#armazene-os-dados-de-uma-rela%C3%A7%C3%A3o-no-pig-em-uma-tabela-do-hive)
   * Sort the output of a Pig relation
   * Remove the duplicate tuples of a Pig relation
   * Specify the number of reduce tasks for a Pig MapReduce job
   * Join two datasets using Pig
   * Perform a replicated join using Pig
   * Run a Pig job using Tez
   * Within a Pig script, register a JAR file of User Defined Functions
   * Within a Pig script, define an alias for a User Defined Function
   * Within a Pig script, invoke a User Defined Function
3. [Análise de Dados](#an%C3%A1lise-de-dados)
   * Write and execute a Hive query
   * Define a Hive-managed table
   * Define a Hive external table
   * Define a partitioned Hive table
   * Define a bucketed Hive table
   * Define a Hive table from a select query
   * Define a Hive table that uses the ORCFile format
   * Create a new ORCFile table from the data in an existing non-ORCFile Hive table
   * Specify the storage format of a Hive table
   * Specify the delimiter of a Hive table
   * Load data into a Hive table from a local directory
   * Load data into a Hive table from an HDFS directory
   * Load data into a Hive table as the result of a query
   * Load a compressed data file into a Hive table
   * Update a row in a Hive table
   * Delete a row from a Hive table
   * Insert a new row into a Hive table
   * Join two Hive tables
   * Run a Hive query using Tez
   * Run a Hive query using vectorization
   * Output the execution plan for a Hive query
   * Use a subquery within a Hive query
   * Output data from a Hive query that is totally ordered across multiple reducers
   * Set a Hadoop or Hive configuration property from within a Hive query
   * Output the execution plan for a Hive query
   * Use a subquery within a Hive query
   * Output data from a Hive query that is totally ordered across multiple reducers
   * Set a Hadoop or Hive configuration property from within a Hive query

## Ingestão de Dados

### Importe dados de uma tabela em uma base de dados relacional para o HDFS

Para isso, nós devemos usar o [`sqoop import`][SQOOP-IMPORT] e sempre devemos
usar o argumento `--connect` que vai permitir que o `sqoop` se conecte ao banco
de dados. Por exemplo:
```
$ sqoop import \
      --connect jdbc:mysql://database.example.com:3306/employees
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

### Importe os resultados de uma *query* a um banco de dados para o HDFS

Podemos realizar uma *query* em um banco de dados relacional e importar o
resultado direto para o HDFS. Para isso, nós precisaremos usar dois argumentos:

* `--query`, em que passamos a `query` para o `sqoop`; e
* `--target-dir`, em que passamos o diretório no HDFS em que os dados serão
  gravados.

Por exemplo:

```
$ sqoop import \
      --connect jdbc:mysql://database.example.com:3306/ecommerce.db \
      --username fulano \
      --password 123456 \
      --query 'SELECT vendas.*, fornecedores.* FROM vendas JOIN fornecedores on (vendas.forn_id == fornecedores.id) WHERE vendas.preco > 5000' \
      --target-dir /user/fulano/meus-resultados
```

A [documentação][FREE-FORM QUERY IMPORTS] mostra alguns argumentos extras para
casos especiais.

[FREE-FORM QUERY IMPORTS]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_free_form_query_imports


### Importe uma tabela de um banco de dados relacional para uma tabela do Hive

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
      --connect jdbc:mysql://database.example.com:3306/ecommerce.db \
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


### Insira ou atualize dados do HDFS para uma tabela de uma banco de dados relacional

Além de inserir tabelas de um banco de dados relacional para o HDFS, o `sqoop`
também nos permite fazer o processo contrário. Vamos ver agora como exportar
uma tabela do HDFS para um banco de dados.

Ao invés do `import` nós devemos usar o `export`, e alguns dos argumentos são os mesmos, tais quais
`--connect <jdbc-uri>`, `--username` e `--password`. Vamos ver um exemplo:

```
$ sqoop export \
      --connect jdbc:mysql://database.example.com:3306/ecommerce.db \
      --username fulano \
      --password 123456 \
      --table vendas \
      --export-dir /user/fulano/vendas
```

O comando acima vai exportar uma tabela localizada no diretório
`/user/fulano/vendas` no HDFS para uma banco de dados relacional MySQL chamado
`ecommerce.db`.

O comando `export` aceita outros argumentos e recomendo a
[documentação][SQOOP-EXPORT] para uma melhor explicação.

[SQOOP-EXPORT]: http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_literal_sqoop_export_literal


### Inicie um agente do Flume a partir de um arquivo de configuração

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


### Configure um `channel` de memória com um tamanho específico

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


## Transformação de Dados

### Escreva a execute um script do Pig

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


### Carregue dados para uma relação do Pig sem definir um esquema

Nós utilizamos a declaração `LOAD` para carregar um conjunto de dados para uma
relação do Pig. A forma mais básica de carregar um conjunto de dados é:

```
A = LOAD 'vendas';
```

Em que carregamos uma tabela chamada `vendas` usando o método padrão,
`PigStorage`.

É importante saber que o Pig considera tabulação como delimitador padrão. Para
definir um delimitador explictamente precisamos usar `USING
PigStorage('delimitador')`. Por exemplo, para carregar um arquivo CSV, nós
temos que fazer o seguinte:

```
A = LOAD 'vendas' USING PigStorage(',');
```

A [documentação][pig_load] possui mais detalhes.

[pig_load]: https://pig.apache.org/docs/r0.15.0/basic.html#load


### Carregue dados para uma relação do Pig definindo um esquema


Agora, veremos como definir um esquema ao carregar uma tabela de dados no Pig.

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
```

Nós carregamos a mesma tabela, só que desta vez nós definimos o nome de cada
coluna e o tipo de dado que cada coluna armazena. Não é necessário definir o
tipo da variável e, caso não façamos, será carregada como texto.


### Carregue dados de uma tabela do Hive para uma relação do Pig

O Pig nos permite obter dados de outras fontes como de uma tabela armazenada no
Hive. Para isso, nós vamos precisar o argumento `USING` junto com o `LOAD`:

```
A = LOAD 'vendas' USING org.apache.hive.hcatalog.pig.HCatLoader();
```

Veja que utilizamos o `HCatLoader` ao invés do `PigStorage`. Para maiores detalhes, veja a [documentação][HCatalog].

[HCatalog]: https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore


### Use o Pig para transformar dados para um formato específico

Após carregar os dados usando `LOAD`, nós vamos quere executar alguma operação
neles e para isso podemos usar o comando `FOREACH` que executa um *loop* linha
a linha. Por exemplo:

```
A = LOAD 'vendas';
B = FOREACH A GENERATE $0;
```

Nós carregamos a tabela acima  na primeira linha e criamos uma relação `B` que
contém apenas a informação da primeira coluna.

Um exemplo mais interessante é calcular o valor da receita de cada venda:

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
B = FOREACH A GENERATE preco * qtde;
```

Veja a [documentação do `FOREACH`][foreach_docs] para maiores detalhes.

[foreach_docs]: https://pig.apache.org/docs/r0.15.0/basic.html#foreach


### Transforme os dados para um esquema pŕe-definido do Hive

Neste tópico, o objetivo é transformar uma tabela carregada com o Pig para um
esquema já pré-definido do Hive. Para isso, basta usarmos o `FOREACH` para
colocar as colunas da tabela na mesma ordem da tabela do Hive. Por exemplo:

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
B = FOREACH A GENERATE qtde, preco, item;
```

Em que o esquema do Hive teria o esquema `qtde`, `preco` e `item`.

### Agrupe os dados em uma ou mais relações do Pig

Em processos de análise de dados, o argupamento de dados é uma operação básica,
essencial e corriqueira. No Pig, nós utilizamos o `GROUP BY`:

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
B = GROUP A BY item;
```

Também é possível agrupar por mais de um campo:

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
B = GROUP A BY (item, preco);
```

Caso tenhamos duas relações, podemos fazer um `COGROUP`:

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
B = LOAD 'estoque' AS (item_nome:chararray, qtde_estoque:int);
C = COGROUP A BY item, B BY item_nome;
```

A [documentação][pig_group] possui mais detalhes e exemplos.

[pig_group]: https://pig.apache.org/docs/r0.15.0/basic.html#group


### Use o Pig para remover valores ausentes em uma relação

É comum termos valores ausentes em nosso dados e, em muita das vezes,
precisamos excluir estas entradas. Para fazer isto com o Pig, nós usamos o
comando `FILTER`:

```
A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
B = FILTER A BY item != '';
```

A [documentação do `FILTER`][pig_filter] possui mais detalhes.

[pig_filter]: https://pig.apache.org/docs/r0.15.0/basic.html#filter


### Armazene os dados de uma relação no Pig em uma pasta no HDFS

Após realizarmos nossas operações nos dados, é bem provável que iremos querer
salvar o conjunto de dados trasnformado para futuras consultas. Para isso, nós
utilizamos o método `STORE`. No exemplo abaixo, exemplifico como armazenar uma
relação `A`:

```
STORE A INTO 'meus_dados' USING PigStorage(',');
```

Estamos salvando uma relação `A` em uma pasta chamada `meus_dados`. Esta pasta
contém arquivos típicos de uma saída de um processo *MapReduce*: um arquivo
chamado `_SUCCESS` apontando o sucesso da operação, e um ou mais arquivos com
os dados.

Aqui, nós estamos forçando que os elementos de cada linha sejam separadas por
vírgula (`USING PigStorage(',')`). Se nós não usarmos este argumento, o Pig
salvará usando tabulação que é seu delimitador padrão.

No nosso exemplo acima, é possível que o Pig salve os dados na máquina local;
vai depender de como o pig foi iniciado (e.g., `pig -x local`). para nos
certificarmos que a relação `A` será salva no HDFS, nós temos que passar o
endereço correto do HDFS.

```
STORE A INTO 'hdfs://sandbox-hdp.hortonworks.com:8020/user/root/meus_dados';
```

[A documentação do `STORE`][pig_store] possui mais exemplos.

[pig_store]: https://pig.apache.org/docs/r0.15.0/basic.html#store


### Armazene os dados de uma relação no Pig em uma tabela do Hive

Além de salvar o arquivo em um diretório do HDFS, nós também podemos salvar
diretamente para um banco de dados do Hive. Para isso, nós precisaremos primeiro
criar a tabela no Hive. Você pode fazer isso na interface gráfica do Ambari ou na
interface do Hive no terminal:

```
hive> CREATE TABLE vendas (
    > item string,
    > preco float,
    > qtde int
    > );
```

Com isso, teremos uma tabela criada no Hive. Por padrão, o Hive criara no banco
de dados `default`. Agora, para podermos exportar o dado para o Hive, nós
precisaremos usar a *flag* `-useHCatalog`, tanto se estivermos rodando um
script quanto para abrir o *grunt*:

```
$ pig  -useHCatalog
...
grunt> A = LOAD 'vendas' AS (item:chararray, preco:float, qtde:int);
grunt> STORE A INTO 'nome_da_db.vendas' using org.apache.hive.hcatalog.pig.HCatStorer();
```

Outro requisito é que os dados tenham um esquema definido e que seja o mesmo da
tabela criada no Hive.

Para maioes detalhes, veja a [documentação][pig_hive].

[pig_hive]: https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore


### Sort the output of a Pig relation

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#order-by)


### Remove the duplicate tuples of a Pig relation

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#distinct)


### Specify the number of reduce tasks for a Pig MapReduce job

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/perf.html#parallel)


### Join two datasets using Pig

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#join-outer)


### Perform a replicated join using Pig

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/perf.html#replicated-joins)


### Run a Pig job using Tez

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/perf.html#tez-mode)


### Within a Pig script, register a JAR file of User Defined Functions

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/udf.html#piggybank)


### Within a Pig script, define an alias for a User Defined Function

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#define-udfs)


### Within a Pig script, invoke a User Defined Function

  [LEARN MORE](https://pig.apache.org/docs/r0.15.0/basic.html#register)


## Análise de Dados

### Write and execute a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/Tutorial)


### Define a Hive-managed table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-Create/Drop/TruncateTable)


### Define a Hive external table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ExternalTables)


### Define a partitioned Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables)


### Define a bucketed Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-BucketedSortedTables)


### Define a Hive table from a select query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTableAsSelect(CTAS))


### Define a Hive table that uses the ORCFile format

  [LEARN MORE](https://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/)


### Create a new ORCFile table from the data in an existing non-ORCFile Hive table

  [LEARN MORE](https://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/)


### Specify the storage format of a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RowFormat,StorageFormat,andSerDe)


### Specify the delimiter of a Hive table

  [LEARN MORE](https://hortonworks.com/hadoop-tutorial/using-hive-data-analysis/)


### Load data into a Hive table from a local directory

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Loadingfilesintotables)


### Load data into a Hive table from an HDFS directory

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Loadingfilesintotables)


### Load data into a Hive table as the result of a query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingdataintoHiveTablesfromqueries)


### Load a compressed data file into a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage)


### Update a row in a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Update)


### Delete a row from a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-Delete)


### Insert a new row into a Hive table

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingvaluesintotablesfromSQL)


### Join two Hive tables

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Joins)


### Run a Hive query using Tez

  [LEARN MORE](https://hortonworks.com/hadoop-tutorial/supercharging-interactive-queries-hive-tez/)


### Run a Hive query using vectorization

  [LEARN MORE](https://hortonworks.com/hadoop-tutorial/supercharging-interactive-queries-hive-tez/)


### Output the execution plan for a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Explain)


### Use a subquery within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries)


### Output data from a Hive query that is totally ordered across multiple reducers

  [LEARN MORE](https://issues.apache.org/jira/browse/HIVE-1402)


### Set a Hadoop or Hive configuration property from within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Configuration#AdminManualConfiguration-ConfiguringHive)


### Output the execution plan for a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Explain)


### Use a subquery within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries)


### Output data from a Hive query that is totally ordered across multiple reducers

  [LEARN MORE](https://issues.apache.org/jira/browse/HIVE-1402)


### Set a Hadoop or Hive configuration property from within a Hive query

  [LEARN MORE](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Configuration#AdminManualConfiguration-ConfiguringHive)
