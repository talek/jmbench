# JMBench

JMBench aims to provide a common scaffold for benchmarking queries performance
on various database backends. It uses [Apache JMeter](https://jmeter.apache.org/)
engine behind. JMBench takes the SQL queries from a CSV file and it executes
them on the target system using a JDBC connection. The queries are run one by
one and the execution time is captured in an output/results XML file.

## Project Layout

The main JMeter specification is defined in `jmbench.jmx` file. For every
database backend there is a corresponding folder (e.g. `oracle`, `postgresql`
etc.). Each database specific folder contains a `config.properties` file which
can be use to configure the benchmark.

## The CSV file

The input CSV file must have two columns separated by a `::` separator. The
first column is the query identifier and the second one is the query text.

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

For example, for running the benchmark for Oracle the command is:

    jmeter -n -t jmbench.jmx -p oracle/config.properties -l results.xml -f

