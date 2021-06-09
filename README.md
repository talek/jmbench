# JMBench

JMBench aims to provide a common scaffold for benchmarking queries performance
on various database backends. It uses [Apache JMeter](https://jmeter.apache.org/)
engine behind. JMBench takes the SQL queries from a CSV file and it executes
them on the target system using a JDBC connection. The queries are run one by
one and the execution time is captured in an output/results XML file.

Running queries using JDBC and not through some fancy scripts is the
recommended way because JDBC is transparent we can reuse the same execution
logic. If we were to choose the shell script executor approach, then we would
be required to embbed a different logic in each and every script, taking into
account the peculiarities of every database client application (e.q. `sqlplus`
for Oracle or `psql` for Postgresql).

Special care is needed when the connection pool is configured so that to have
all the database sessions already opened. Otherwise, the connection time will
be added to the first query executed from the CSV file. That is why the
connection pool has the "Preinit Pool" property set to `True`.

## Project Layout

The main JMeter specification is defined in `jmbench.jmx` file. For every
database backend there is a corresponding folder (e.g. `oracle`, `postgresql`
etc.). Each database specific folder contains a `config.properties` file which
can be used to configure the benchmark.

## The CSV file

The input CSV file must have two columns separated by a `::` separator. The
first column is the query identifier and the second one is the query text. No
header line should be present. You may find an example in each database backend
folder.

## The Configuration File

The most important properties are:

  * `DATABASE_URL`: The database connection string in JDBC format.
  * `DATABASE_USER`: The database user to connect to the target database.
  * `DATABASE_USER_PWD`: The user's password.
  * `QUERY_CSV_FILE`: The CSV file with the queries to be executed.
  * `QUERY_TIMEOUT_SEC`: The execution timeout for each query (in seconds).
  * `QUERY_MAX_ROWS`: How many records to fetch from the resultset.

## Run Benchmark

To run the benchmark you may use the following command:

    jmeter -n -t jmbench.jmx -p <db_engine>/config.properties -l results.xml -f

For example, to run a benchmark for Oracle the command is:

    jmeter -n -t jmbench.jmx -p oracle/config.properties -l results.xml -f

With the following CSV:

```
Q1::select count(*) from all_source
Q2::SELECT * FROM   (SELECT * FROM   dba_tables ORDER BY DBMS_RANDOM.RANDOM) WHERE  rownum < 10
Q3::select sysdate from dual
```

We get the following results in `results.xml` file:

```
<?xml version="1.0" encoding="UTF-8"?>
<testResults version="1.2">
<sample t="342" it="0" lt="341" ct="0" ts="1623264961541" s="true" lb="Q1" rc="200" rm="OK" tn="Thread 1-1" by="16" sby="0" ng="1" na="1" resultset___.0023="1"/>
<sample t="96" it="0" lt="92" ct="0" ts="1623264961641" s="true" lb="Q2" rc="200" rm="OK" tn="Thread 1-1" by="3349" sby="0" ng="1" na="1" resultset___.0023="9"/>
<sample t="1" it="0" lt="1" ct="0" ts="1623264961643" s="true" lb="Q3" rc="200" rm="OK" tn="Thread 1-1" by="30" sby="0" ng="1" na="1" resultset___.0023="1"/>
</testResults>
```

The meaning of the most important XML attributes above is:

  * `t`: the response time in ms
  * `ts`: the timestamp in UNIX epoch format
  * `lb`: the label of the query as defined in the CSV file
  * `rm`: the result message; OK or the encounter error in case of a failure
  * `resultset___*`: how many records have been processed from the query resultset
