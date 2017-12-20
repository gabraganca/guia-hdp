# Lista de Exercícios

Os exercícios a seguir foram retirados do simulado da Hortonworks.

* [Exercício 1](#exercicio-1)
* [TASK 2](#task-02)
* [TASK 3](#task-03)
* [TASK 4](#task-04)
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

## TASK 02

Cleanse Data using Pig

1. Notice the comma-separated values of the flightdelays files in HDFS contain
   historical data for airline flight delays. The columns in the files match
the following schema:
2. Year, Month, DayofMonth, DayOfWeek, DepTime, CRSDepTime, ArrTime,
   CRSArrTime,
3. UniqueCarrier, FlightNum, TailNum, ActualElapsedTime, CRSElapsedTime,
   AirTime,
4. ArrDelay, DepDelay, Origin, Dest, Distance, TaxiIn, TaxiOut, Cancelled,
5. CancellationCode, Diverted, CarrierDelay, WeatherDelay, NASDelay,
   SecurityDelay,
6. LateAircraftDelay
7. Write a Pig script that satisfies all of the following criteria:
   * Load all of the data in /user/horton/flightdelays
   * Remove all rows in the flightdelays data where the DepTime column equals
     the string "NA".
   * The output should only contain
     the Year, Month, DayofMonth, DepTime, UniqueCarrier, FlightNum, ArrDelay, Origin and Dest

Store the result as comma-separated records in a new directory in HDFS
named /user/horton/flightdelays_clean Save the script in a file
named flightdelays_clean.pig and save it in
the /home/horton/solutions directory on the local filesystem of the client
machine.

## TASK 03

Analyze Data using Pig

1. Write a Pig script saved on the client machine
   as /home/horton/solutions/cleaned_total.pig that calculates the number of
rows in the /user/horton/flightdelays_cleanfiles in HDFS. Store the output of
your script in a new directory in HDFS named cleaned_total.
2. The Dest column is the destination airport code where the flight ends. Write
   a Pig script saved on the client machine
as /home/horton/solutions/denver_total.pig that calculates the number of rows
in the /user/horton/flightdelays_clean data where the Dest field equals the
Denver airport code "DEN". Store the output of your script in a new directory
in HDFS named denver_total.
3. The ArrDelay column is the number of minutes that a flight arrived late.
   Write a Pig script saved on the client machine
as /home/horton/solutions/denver_late.pig that counts the number of flights
whose Dest is the "DEN" airport that arrived 60 or more mintues late. Store the
output of your script in a new directory in HDFS named denver_late

## TASK 04

Define a Hive Table

1. Define a Hive table named flightdelays that matches the data stored in
   your /user/horton/flightdelays_clean directory in HDFS. The Hive table
should satisfy all of the following criteria: An external table with the
location set to /user/horton/flightdelays_clean The schema matches the
columns Year, Month, DayOfMonth, DepTime, UniqueCarrier, FlightNum, ArrDelay, Origin and Dest
The UniqueCarrier, Origin and Dest columns are string types; the other columns
are all integers

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
