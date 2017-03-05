chap17-hive
===

Start dfs and yarn beforing using hive

**CLI (Command Line Interface)**

This is non-interactive mode

Make a local files

> echo "X" > ~/dummy.txt

Copy to hdfs

> hdfs dfs -put ~/dummy.txt /input

Build a table from local file using hive

> hive -e "CREATE TABLE dummy (value STRING); \
    LOAD DATA LOCAL INPATH '/home/perra/dummy.txt' \
    OVERWRITE INTO TABLE dummy"

`LOAD DATA LOCAL INPATH` load data from local, `LOAD DATA INPATH` load data from HDFS

The files of database are stored at `/user/hive/warehouse/` on **HDFS**, we can change it by adding `hive.metastore.warehouse.dir` on `hive-site.xml`

### Running Hive

If you plan to have more than one Hive user sharing a Hadoop cluster, you need to make the directories that Hive uses writable by all users. The following commands will create the directories and set their permissions appropriately

    hdfs dfs -mkdir /tmp
    hdfs dfs -chmod a+w /tmp
    hdfs dfs -mkdir -p /usr/hive/warehouse
    hdfs dfs -chmod a+w /usr/hive/warehouse

To see all configuration of both Hadoop and Hive

> hive -v

### Comparison with Traditional Database

#### Schema on Read versus on Schema on Write

* In a traditional database, a table's schema is enforced at data load time. This design is called **schema on write** and making query time performance faster because the database can index columns and perform compression on the data.
* **schema on read** verifies the data when a query is issued. This makes for a very fast initial load, since the data does not have to be read, parsed, and serialized to disk in the database's internal format.

### HiveQL

#### Data Type

* Primitive: BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, FLOAT, DOUBLE, DECIMAL(BigDecimal in Java, DECIMAL(5,2)), STRING(unbounded, max is 2GB), VARCHAR(variable length), CHAR(fixed length), BINARY, TIMESTAMP, DATE
* Complex: ARRAY, MAP, STRUCT, UNION

    CREATE TABLE complex (
            c1 ARRAY<INT>,
            c2 MAP<STRING, INT>,
            c3 STRUCT<a:STRING, b:INT, c:DOUBLE>
            c4 UNIONTYPE<STRING, INT>
    );

    SELECT c1[0], c2['b'], c3.c, c4 FROM complex;

#### Consersions

It's almost the same as Java, implicit type conversions from low to higher, and cast from higher to lower

#### External Table

    CREATE EXTERNAL TABLE external_table (dummy STRING)
        LOCATION '/user/tom/external_table';
    LOAD DATA INPATH '/usr/tom/data.txt' INTO TABLE external_table;

With the `EXTERNAL` keyword, Hive knows that it is not managing the data, so it doesn't move it to its warehouse directory. This is a useful feature because it means you can create the data **lazily** after creating the table. When you drop an external table, Hive will leave the data untouched and only delete the metadata.

As a rule of thumb, **if you are doing all your processing with Hive, then use managed tables, but if you wish to use Hive and other tools on the same dataset, then use external tables. A common pattern is to use an external table to access an initial dataset stored in HDFS, then use a Hive transform to move the data into a managed Hive table.**

Or, **An external table can be used to export data from Hive for other applications to use**

#### Storage Formats

There are two dimensions that govern table storage in Hive: the **row format** and the **file format**

* row format: Serializer-Deserializer

**default storage format: delimited text**

* The default row delimiter is not a tab character, but a Ctrl-A, ^A; the default collection item delimiter is Ctrl-B, used to delimit items in an ARRAY or STRUCT, or in key-value pairs in a MAP; the default map key delimiter is a Ctrl-C, used to delimit the key and value in a MAP

    CREATE TABLE ...
    ROW FORMAT DELIMITED
        FIELD TERMINATED BY '\001'
        COLLECTION ITEMS TERMINATED BY '\002'
        MAP KEYS TERMINATED BY '\003'
        LINES TERNIMATED BY '\n'
    STORED AS TEXTFILE;

**Binary storage formats: Sequence files; Avro datafiles; (row-oriented)| Parquet files; RCFiles; and ORCFiles (column-oriented)**

row-oriented formats and column-oriented formats

    CREATE TABLE user_parquent STORED AS PARQUET
    AS
    SELECT * FROM users;

**Drop table**

* `DROP` deletes the data and metadata for a table. In the case of external tables, only the metadata is deleted; the data is left untouched.
* `TRUNCATE`　deletes all the data in a table but keep the table definition. This doesn't work for external tables; use dfs -rmr to remove the external table directory directly.

### Querying Data

#### Sorting and Aggregating

* `ORDER BY` performs a parrallel total sort of the input.
* `SORT BY` is used when a globally sorting is not required

    FROM records
    SELECT year, temperature
    DISTRIBUTE BY year
    SORT BY year ASC, temperature DESC;

#### MapReduce Scripts

Using an approach like Hadoop Streaming, the **`TRANSFORM`, `MAP`, `REDUCE`** clauses make it possible to invoke an external script or program from Hive.

    ADD FILE /localfile/is_good_quality.py
    FROM records
    SELECT TRANSFORM(year, temperature, quality)
    USING 'is_good_quality.py'
    AS year, temperature
    
