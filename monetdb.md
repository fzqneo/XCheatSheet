MonetDB
=======

## monetdbd
Reference: [monetdbd](https://www.monetdb.org/Documentation/monetdbd)

Each monetdb demon is associated with a **dbfarm**. Only one instance of *monetdbd* can be running on a dbfarm.

1. Create a dbfarm: `monetdbd create ~/my-dbfarm`
2. Show properties: `monetdbd get all ~/my-dbfarm`
3. Start the demon on a dbfarm: `monetdbd start ~/my-dbfarm`
4. Shutdown: `monetdbd stop ~/my-dbfarm`

## monetdb
Reference: [monetdb man-page](https://www.monetdb.org/Documentation/monetdb-man-page)

1. Create a new database: `monetdb create tpch`
1. A newly-created DB is in *locked* state. Need to release it for use: `monetdb release tpch`
1. Show status of all databases in the dbfarm: `monetdb status`
1. Display properties of a database: `monetdb get all tpch`
2. Set database to use single-thread: `monetdb stop tpch && monetdb set nthreads=1 tpch`

## mclient
Reference: [mclient man-page](https://www.monetdb.org/Documentation/mclient-man-page)

### Start

``mclient -u monetdb -d tpch``


**Pseudo-passwordless start**: put the username and password in a file *.monetdb*:
```bash
$ cat .monetdb
user=monetdb
password=monetdb
```

Useful options:

+ **-l {sql|mal}**: Use SQL or MAL as query language
+ **-i [ms|s|m]**: Specfify time measurement precision
+ **-f {sql|csv|raw|X|tab|xml}**: Specify display format of query result
+ **-s stmt**: Provide query statement from shell command

**Running a SQL file from command line**:

``mclient -d tpch Q1.sql``


### Backslash commands


Purpose        | PostgreSql    | MonetDB
---------------|-------------|-----------:
Change DB      | \c tpch     | no way
List all DB    | \l          | no way
Describe table | \d [tpch]   | \d [tpch]
Run from file  | \i file.sql | \\\< file.sql
Write to file  |             | \\\> file.txt
Quit           | \q          | \q
Help           |             | \?


### Load a table from csv

#### Inside mclient
```sql
COPY INTO customer from '/home/neo/tpch/tpch_2_16_1/dbgen/customer.csv' USING DELIMITERS '|','\n' NULL AS '';
```

#### Running from shell command

```bash
# The file must be readable by the server. $file is the absolute path name of the file, $table is the name of the table, $db is the name of the database.
mclient -d $db -s "COPY INTO $table FROM ’$file’ USING DELIMITERS ’,’,’\\n’,’\"’"

# When the file is to be read by mclient (e.g. the server has no access to the file). $file is the (absolute or relative) path name of the file, $table is the name of the table, $db is the name of the database.
mclient -d $db -s "COPY INTO $table FROM STDIN USING DELIMITERS ’,’,’\\n’,’\"’" - < $file
```

### Export a table/query result to csv
```sql
COPY (select encode_date(l_shipdate) from "sys"."lineitem") INTO '/tmp/lineitem.l_shipdate.txt' NULL AS '0';
```
Notice: file path must be **absolute path**.

## Profiling queries

Trace profile in MonetDB gives MAL-level statistics.

+ TRACE command in SQL interface
+ Stethoscope - Proper for online monitoring
+ Tachograph - Proper for offline analysis. It generates three different formats: .trace, .csv and .json. 
    - ".trace" is same as TRACE SQL output
    - ".csv" is good for loading and parsing. 
    - ".json" contain a useful *beautystmt* attribute that gives MAL instructions more readable interpretations.
