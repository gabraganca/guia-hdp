# Lista de Exercícios

Os exercícios a seguir foram retirados do simulado da Hortonworks. Clique em
**Solução** para mostrar a resposta.

Alguns exercícios apontam para conjuntos de dados presentes na máquina virtual
na AWS da Hortonworks que foi criada para praticar para o exame. Para sua
praticidade, este conjuntos de dados foram replicados neste diretório e estão
no diretório `datasets`.

* [Exercício 1](#exerc%C3%ADcio-1)
* [Exercício 2](#exerc%C3%ADcio-2)
* [Exercício 3](#exerc%C3%ADcio-3)
* [Exercício 4](#exerc%C3%ADcio-4)
* [TASK 5](#task-05)
* [TASK 6](#task-06)
* [TASK 7](#task-07)
* [TASK 8](#task-08)
* [TASK 9](#task-09)
* [TASK 10](#task-10)


## Exercício 1

**Importe um arquivo para o HDFS**

1. Crie um novo diretório no HDFS nomeado `/user/horton/flightdelays`.
2. Coloque os três arquivos presentes no diretório
   `/home/horton/datasets/flightdelays` da máquina local para o diretório
   `/user/horton/flightdelays` no HDFS

<details>
<summary><b>Solução</b></summary>

1. Para criar um diretório em um sistema de arquivos, nós usamos o comando
   `mkdir`. No HDFS, usamos este mesmo comando, mas como uma `flag` para o
   comando `hadoop fs`:
   ```
   # hadoop fs -mkdir -p /user/horton/flightdelays
   ```
   Aqui, o `#` é o prompt de comando. Há casos que este prompt é o `>` ou o `$`.

   Também usamos a flag `-p` que é passada para o `mkdir` para crirar diretórios
   parentes caso não existam, e que, neste caso, são os diretórios `user` e
   `horton`.

2. Os arquivos estão presentes na pasta `datasets` nese repositório. Para
   enviar para o HDFS, nós precisamos fazer:
   ```
   # hadoop fs -put flightdelays/* /user/horton/flightdelays/
   ```

</details>

## Exercício 2

**Limpe os dados usando o Pig**

Repare ue os arquivos CSV `flightdelays` que importamos para o HDFS contem
dados histórios de atrasos de voos. As columas possuem o seguinte esquema:

> `Year, Month, DayofMonth, DayOfWeek, DepTime, CRSDepTime, ArrTime, CRSArrTime,
UniqueCarrier, FlightNum, TailNum, ActualElapsedTime, CRSElapsedTime, AirTime,
ArrDelay, DepDelay, Origin, Dest, Distance, TaxiIn, TaxiOut, Cancelled,
CancellationCode, Diverted, CarrierDelay, WeatherDelay, NASDelay,
SecurityDelay, LateAircraftDelay`

Escreva um script no Pig que satisfaça os seguintes critérios:

* Carregue todos os dados presentes em `/user/horton/flightdelays`.
* Remova todas as linhas do conjunto de dados em que a coluna DepTime é igual a `NA`.
* A saída deve conter apenas as seguintes colunas:
  `Year, Month, DayofMonth, DepTime, UniqueCarrier, FlightNum, ArrDelay, Origin, Dest`

Em seguida, aramazene o resultado como uma rquivo separado por vírgulas em um
novo directório no HDFS chamado `/user/horton/flightdelays_clean`. Salve o
script em um arquivo chamado `flightdelays_clean.pig` e salve no diretório
`/home/horton/solutions` no sistema local na máquina cliente.

<details>
<summary><b>Solução</b></summary>

O código abaixo exemplifica uma solução.

```
/* flightdelays_clean.pig
Limpa o conjunto de dados de atarso de voos
*/

-- carregando os dados
flightdelays = LOAD '/user/horton/flightdelays/flight*'
               USING PigStorage(',')
               AS (Year, Month, DayofMonth, DayOfWeek, DepTime, CRSDepTime,
                   ArrTime, CRSArrTime, UniqueCarrier, FlightNum, TailNum,
                   ActualElapsedTime, CRSElapsedTime, AirTime, ArrDelay,
                   DepDelay, Origin, Dest, Distance, TaxiIn, TaxiOut,
                   Cancelled, CancellationCode, Diverted, CarrierDelay,
                   WeatherDelay,NASDelay, SecurityDelay, LateAircraftDelay);
-- Remove as linhas em que DepTime = NA
no_missing =  FILTER flightdelays BY (chararray)DepTime != 'NA';
-- Mantem apenas as colunas desejadas
subset = FOREACH no_missing GENERATE Year, Month, DayofMonth, DepTime, UniqueCarrier, FlightNum, ArrDelay, Origin, Dest;
-- Salva os arquivos
STORE subset INTO '/user/horton/flightdelays_clean' USING PigStorage(',');
```

Devemos copiar o código e salvar em um arquivo chamado `flightdelays_clean.csv`
em `/home/horton/solutions`. Para rodá-lo, executamos o seguinte comando no
terminal:

```
# pig -x tez /home/horton/flightdelays_clean.pig`
```

</details>

## Exercício 3

**Analise dados usando o Pig**

1. Escreva um *script* em Pig e salve na máquia cliente em
   `/home/horton/solutions/` com o nome `cleaned_total.pig`. Este *script* deve
   calcular o número de linhas no conjunto de dados
   `/user/horton/flightdelays_cleanfiles` presentes no HDFS. Armazene o resultado
   do *script* em um novo diretório no HDFS nomeado `cleaned_total`.
2. A coluna `Dest` é código do aeroporto de destino do voo. Escreva um *script*
   em Pig que calcule o número de linhas no conjunto de dados
   `/user/horton/flightdelays_clean` em que o campo `Dest` é igual ao aeroporto
   de Denver ( código `DEN`). Salve o *script* na máquina cliente em
   `/home/horton/solutions/denver_total.pig` e armazene o resultado do *script*
   em um novo diretório no HDFS nomeado `denver_total`.
3. A coluna `ArrDelay` é o número em minutos em que um voo chegou atrasado.
   Escreva um *script* em Pig que conta o número de voos atrasados que `Dest` é
   igual ao aeroporto `DEN` e que teve 60 minutos ou mais de atraso. Salve o
   *script*  na máquina local em `/home/horton/solutions/denver_late.pig` e
   armazene o resultado do *script* no HDFS em um diretório chamado
   `denver_late`.

<details>
<summary><b>Solução</b></summary>

1. Salve o script abaixo em `/home/horton/solutions/cleaned_total.pig`:
   ```
   /* cleaned_total.pig
   Conta o número de entrdas no arquivo cleaned_total
   */
   -- Carrega o dataset
   dataset = LOAD '/user/horton/flightdelays_clean' USING PigStorage(',')
             AS (Year, Month, DayofMonth, DepTime, UniqueCarrier, FlightNum,
                 ArrDelay, Origin, Dest);
   -- Conta o numero de entradas
   total = FOREACH (GROUP dataset ALL) GENERATE COUNT_STAR(dataset);
   -- Armazena o valor
   STORE total INTO '/user/horton/cleaned_total';
   ```
2. Salve o script abaixo em `/home/horton/solutions/denver_total.pig`:
   ```
   /* denver_total.pig
   Conta o número de voos cujo destino e Denver.
   */
   -- Carrega o dataset
   dataset = LOAD '/user/horton/flightdelays_clean' USING PigStorage(',')
             AS (Year, Month, DayofMonth, DepTime, UniqueCarrier, FlightNum,
                 ArrDelay, Origin, Dest);
   -- Filtra Denver
   only_denver = FILTER dataset BY (chararray)Dest == 'DEN';
   -- Conta o numero de entradas
   total = FOREACH (GROUP only_denver ALL) GENERATE COUNT_STAR(only_denver);
   -- Armazena o valor
   STORE total INTO '/user/horton/denver_total';
   ```
3. Salve o script abaixo em `/home/horton/solutions/denver_late.pig`:
   ```
   /* denver_late.pig
   Conta o número de voos cujo destino e Denver e
   tenham atraso de 60 minutos ou mais
   */
   -- Carrega o dataset
   dataset = LOAD '/user/horton/flightdelays_clean' USING PigStorage(',')
             AS (Year, Month, DayofMonth, DepTime, UniqueCarrier, FlightNum,
                 ArrDelay, Origin, Dest);
   -- Filtra Denver
   late_denver = FILTER dataset BY (chararray)Dest == 'DEN' AND (int)ArrDelay>=60;
   -- Conta o numero de entradas
   total = FOREACH (GROUP late_denver ALL) GENERATE COUNT_STAR(late_denver);
   -- Armazena o valor
   STORE total INTO '/user/horton/denver_late';
   ```

</details>


## Exercício 4

**Defina uma tabela no Hive**

Defina uma tabela no Hive noemada `flightdelays` que corresponde aos dados
armazenados no HDFS no directório `/user/horton/flightdelays_clean`.  A tabela
do Hive deve satisfazer os seguintes critérios:

* Uma tabela extrena com sua localalização configurada para
  `/user/horton/flightdelays_clean`.
* O esquema corresponde às colunas
  `Year`, `Month`, `DayOfMonth`, `DepTime`, `UniqueCarrier`, `FlightNum`, 
   `ArrDelay`, `Origin ` e `Dest`.
* As colunas `UniqueCarrier`, `Origin` e `Dest` são do tipo `string` e as
  outras são do tipo inteiro.

<details>
<summary><b>Solução</b></summary>

Abra o Hive no terminal ou no Ambari e execute os seguintes comandos:

```
CREATE EXTERNAL TABLE flightdelays (
  Year INT,
  Month INT,
  DayOfMonth INT,
  DepTime INT,
  UniqueCarrier STRING,
  FlightNum INT,
  ArrDelay INT,
  Origin STRING,
  Dest STRING
  )
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/horton/flightdelays_clean';
```

</details>

## TASK 05

Use HCatalog with Pig

1. Write a Pig script saved on the client machine
   as /home/horton/solutions/flightdelays_nonzero.pig that satisfies all of the
following criteria: Run the Pig query using Tez as the execution engine Load
the data from the flightdelays table in Hive using HCatalog Remove any rows
where the arrdelay is less than or equal to 0 Order the output by
the arrdelay value descending Store the output as three comma-separated files
in a new directory in HDFS named /user/horton/flightdelays_nonzero ## TASK 06
Analyzing Data with Hive
1. Write a Hive query for each of the tasks below. Save the queries in a single
   text file named /home/horton/solutions/flightdelays.hive: Compute the
average arrdelay of flights landing in Denver (dest equals "DEN") Compute the
average arrdelay of flights where the origin is LAX and the dest is SFO
Determine which dest airport had the highest average arrdelay

## TASK 07

Define and Populate an ORCFile Table

1. Define a Hive table named sfo_weather that satisfies all of the following
   criteria: A Hive-managed table The data is stored in the ORCFile format The
table is populated with the records in
the /home/horton/datasets/flightdelays/sfo_weather.csv file on the client
machine The schema matches the columns in sfo_weather.csv - the first column is
a string named station_name, followed by integers for
the Year, Month, DayOfMonth, precipitation, temperature_max,
and temperature_min

## TASK 08

Hive Join
1. Write a Hive query in a file
   named /home/horton/solutions/flights_weather.hive that satisfies the
following criteria: Use Tez as the execution engine The result of the query is
in a new Hive-managed table named flights_weather stored as a textfile Join
the flightdelays table with the sfo_weather table where dest or origin equals
"SFO" in flightdelays, and the year, month and dayofmonth are equal in the two
tables Select all columns from the flightdelays table, and
the temperature_max and temperature_min columns from sfo_weather

## TASK 09

Hive Partitioned Tables
1. Write a Hive query in a file
   named /home/horton/solutions/weather_partitioned.hive that satisfies the
following criteria: Define a new Hive-managed table
named weather_partitioned that has the same schema as the sfo_weather table The
table is partitioned on the year and month columns The data is stored in the
ORCFile format Insert the records from January, 2008, of the sfo_weather table
into the appropriate partition of weather_partitioned

## TASK 10

Sqoop Export
1. Put the local
   file /home/hortonworks/datasets/flightdelays/sfo_weather.csv into HDFS in a
new directory named /user/hortonworks/weather/
2. Note there is a MySQL database named flightinfo on the namenode machine. It
   contains a table named weather with the following schema:
 +---------------+--------------+------+-----+---------+-------+
 | Field         | Type         | Null | Key | Default | Extra |
 +---------------+--------------+------+-----+---------+-------+
 | station       | varchar(100) | YES  |     | NULL    |       |
 | year          | int(11)      | YES  |     | NULL    |       |
 | month         | int(11)      | YES  |     | NULL    |       |
 | dayofmonth    | int(11)      | YES  |     | NULL    |       |
 | precipitation | int(11)      | YES  |     | NULL    |       |
 | maxtemp       | int(11)      | YES  |     | NULL    |       |
 | mintemp       | int(11)      | YES  |     | NULL    |       |
 +---------------+--------------+------+-----+---------+-------+

3. Use Sqoop to export the weather directory in HDFS to the weather table in
   MySQL on port 3306 on the namenode machine. The username for MySQL
is root and the password is hadoop.
