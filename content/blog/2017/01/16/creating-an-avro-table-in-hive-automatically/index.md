---
tags: ["Hive","Sqoop"]
title: Creating an Avro table in Hive automatically
author: Kit
categories: ["Hadoop"]
date: 2017-01-16T17:53:59-06:00
---

My goal was to create a process for importing data into Hive using Sqoop 1.4.6. It needs to be simple (or easily automated) and use a robust file format.

Importing data from a Relational Database into Hive should be easy. It's not. When you're first getting started there are a lot of snags along the way including configuration issues, missing jar files, formatting problems, schema issues, data type conversion, and ... the list goes on. This post shines some light on a way to use command line tools to import data as Avro files and create Hive schemas plus solutions for some of the problems along the way.  

## Schemas: source vs target

A big problem is schemas. When importing from a relational database, the Hive schema *should match the source as closely as possible without data loss*. 

Creating Hive schemas by hand is tedious, error prone, and unmanageable when you have hundreds/thousands of tables in your source database.

Sqoop does have a *create-hive-table* tool which can create a Hive schema.. but it only works with delimited file formats. A delimited file format might work well for demos but for real use cases they stink. They take up a lot of space. They aren't portable - with no schema defined in the file parsing them consistently is a nightmare. Querying them in Hive is slow.

Luckily, Hadoop has a variety of data formats designed specifially for these problems. Apache's [Avro](https://avro.apache.org/), [ORC](https://orc.apache.org/), or [Parquet](https://parquet.apache.org/) all have compression built in and include the schema IN the file.

While researching Hive's support for Avro, I stumbled across a Hive feature which, given an Avro binary and schema file, you can create a Hive table just by linking to an Avro schema file:

```sql
CREATE EXTERNAL TABLE my_table
STORED AS AVRO
LOCATION '/user/lambda/my_table_avro/'
TBLPROPERTIES ('avro.schema.url'='hdfs:///user/lambda/schemas/my_table.avsc');
```

According to the [Hive documentation for AvroSerDe](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe#AvroSerDe-SpecifyingtheAvroschemaforatable), the `avro.schema.url` property allows you to specify a path to an avsc (avro json schema file) on a web server or in HDFS (safer option for large clusters).

Awesome! No need to specify another list of columns and types! I wish this feature was available for ORC and Parquet too. (Note to self, future open source contribution?) This means if we can Sqoop some data over as Avro, then we don't need to manually create & maintain a Hive schema.

## Sqoop import data as Avro

To bring the data into Hadoop we are using Sqoop 1.4.6. In my case, the data source was an Oracle database and to facilitate testing I created a small wrapper script `sqoop.sh`: 
```bash
#!/bin/bash
export HADOOP_CLASSPATH=/home/lambda/sqoop/ojdbc6.jar
if [ -z "$1" ]; then
   echo "Missing options file. Usage: ./sqoop.sh options-file.txt"
   exit 1
fi
echo Options file = $1
sqoop --options-file $1
```

Side note: Oracle's jdbc thin driver, ojdbc6.jar, normally is installed in Sqoop's lib directory. For this testing, I didn't have access to install the jar. I tried unsuccessfully to use Sqoop's -libjars argument to include the driver. Luckily I found [this stackoverflow answer](http://stackoverflow.com/a/26994076/98933) which includes the oracle thin driver (ojdbc6.jar) in the HADOOP_CLASSPATH.

The majority of my Sqoop job is contained in sqoop_my_table.txt, an options file for Sqoop which you would then call using `./sqoop.sh sqoop_my_table.txt`:
```
# Used by sqoop.sh
# which invokes sqoop --options-file sqoop_my_table.txt
# comments staring with # are allowed
# blank lines are allowed
# spaces at the end of a line are NOT allowed, especially following a command!

# load from oracle to hdfs
import

# Ran into the following error when attempting to import as Avro due to conflicting versions
# Error: org.apache.avro.reflect.ReflectData.addLogicalTypeConversion(Lorg/apache/avro/Conversion;)V
# https://community.hortonworks.com/questions/60890/sqoop-import-to-avro-failing-which-jars-to-be-used.html
-Dmapreduce.job.user.classpath.first=true

# connect using oracle jdbc
--connect
jdbc:oracle:thin:@my-db-server:1521:my-db-sid

# oracle username
--username
MY_ORACLE_USER

# prompt for password
-P

# the oracle table to load
# THIS IS CASE SENSITIVE
--table
MY_TABLE

# import data to Avro Data Files
--as-avrodatafile

# where the data should be stored in HDFS
--target-dir
/user/lambda/my_table_avro

# delete the target directory if it exists already (overwrite)
--delete-target-dir

# Use one mapper (in my case I'm loading a small table with no primary key)
-m
1
```

Then, using an Avro's avro-tools-1.8.1.jar JAR we can extract the schema from the Avro binary files. Even better, as of [AVRO-867](https://issues.apache.org/jira/browse/AVRO-867) it can read files stored in HDFS!

```bash
# export the schema from one of the avro binary files to my_table.avsc
java -jar ../avro-tools-1.8.1.jar getschema hdfs://sandbox.hortonworks.com:8020/user/lambda/my_table_avro/part-m-00000.avro > my_table.avsc
# create a new directory in HDFS to store our schemas
hdfs dfs -mkdir schemas
# upload our schema file into our HDFS schemas directory
hdfs dfs -put my_table.avsc schemas
```

Now, we have our Avro binary files and the associated schema in HDFS. Run the `hive` CLI and create our Hive table:

```sql
CREATE EXTERNAL TABLE my_table
STORED AS AVRO
LOCATION '/user/lambda/my_table_avro/'
TBLPROPERTIES ('avro.schema.url'='hdfs:///user/lambda/schemas/my_table.avsc');
```

Done! So the workflow to automate looks something like:

 1. Run sqoop to export table to HDFS
 1. Use Avro tools to generate schemas for exported data
 1. Create Hive tables using Hive CLI

Now just need to automate it...

## Appendix A: Solutions for various errors

Trying to get the schema from a binary Avro file using Avro tools.. ERROR! Turns out 1.7.7 was too old for Hadoop 2.7... 1.8.1 worked fine.
```
[lambdaops@sandbox sqoop]$ java -jar ../avro-tools-1.7.7.jar getschema hdfs://sandbox.hortonworks.com:8020/user/lambda/my_table_avro/part-m-00000.avro
Exception in thread "main" org.apache.hadoop.ipc.RemoteException: Server IPC version 9 cannot communicate with client version 4
        at org.apache.hadoop.ipc.Client.call(Client.java:1066)
        at org.apache.hadoop.ipc.RPC$Invoker.invoke(RPC.java:225)
        at com.sun.proxy.$Proxy1.getProtocolVersion(Unknown Source)
        at org.apache.hadoop.ipc.RPC.getProxy(RPC.java:396)
        at org.apache.hadoop.ipc.RPC.getProxy(RPC.java:379)
        at org.apache.hadoop.hdfs.DFSClient.createRPCNamenode(DFSClient.java:118)
        at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:222)
        at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:187)
        at org.apache.hadoop.hdfs.DistributedFileSystem.initialize(DistributedFileSystem.java:89)
        at org.apache.hadoop.fs.FileSystem.createFileSystem(FileSystem.java:1328)
        at org.apache.hadoop.fs.FileSystem.access$200(FileSystem.java:65)
        at org.apache.hadoop.fs.FileSystem$Cache.get(FileSystem.java:1346)
        at org.apache.hadoop.fs.FileSystem.get(FileSystem.java:244)
        at org.apache.hadoop.fs.Path.getFileSystem(Path.java:187)
        at org.apache.avro.mapred.FsInput.<init>(FsInput.java:37)
        at org.apache.avro.tool.Util.openSeekableFromFS(Util.java:110)
        at org.apache.avro.tool.DataFileGetSchemaTool.run(DataFileGetSchemaTool.java:47)
        at org.apache.avro.tool.Main.run(Main.java:84)
        at org.apache.avro.tool.Main.main(Main.java:73)
```

Running Sqoop's `create-hive-table` with `--as-avrodatafile`... ERROR! Sqoop only supposed delimited file formats with the `create-hive-table` tool.
```
17/01/16 23:43:38 INFO tool.BaseSqoopTool: Using Hive-specific delimiters for output. You can override
17/01/16 23:43:38 INFO tool.BaseSqoopTool: delimiters with --fields-terminated-by, etc.
Hive import is not compatible with importing into AVRO format.
```

Running Sqoop with a `--split-by`... ERROR! These properties are CASE SENSITIVE!
```
java.sql.SQLSyntaxErrorException: ORA-00904: "CaSe_SeNsItIvE_iD": invalid identifier
```

Sqoop options file problems:

 1. Be very careful about whitespace because Sqoop seems very picky.
 1. Comments staring with # are allowed.
 1. Blank lines are allowed. 
 1. Spaces at the end of a line are NOT allowed, especially following a command!
 1. Table/column names are case sensitive (at least with oracle). 

Running a Sqoop import... ERROR! Oracle's jdbc thin driver (ojdbc6.jar) wasn't in Sqoop's classpath. Add to Sqoop's lib directory or to HADOOP_CLASSPATH. Note: unable to get Sqoop's -libjars argument to work in this case.
```
17/01/17 02:36:00 ERROR sqoop.Sqoop: Got exception running Sqoop: java.lang.RuntimeException: Could not load db driver class: oracle.jdbc.OracleDriver
java.lang.RuntimeException: Could not load db driver class: oracle.jdbc.OracleDriver
        at org.apache.sqoop.manager.OracleManager.makeConnection(OracleManager.java:287)
        at org.apache.sqoop.manager.GenericJdbcManager.getConnection(GenericJdbcManager.java:52)
        at org.apache.sqoop.manager.SqlManager.execute(SqlManager.java:763)
        at org.apache.sqoop.manager.SqlManager.execute(SqlManager.java:786)
        at org.apache.sqoop.manager.SqlManager.getColumnInfoForRawQuery(SqlManager.java:289)
        at org.apache.sqoop.manager.SqlManager.getColumnTypesForRawQuery(SqlManager.java:260)
        at org.apache.sqoop.manager.SqlManager.getColumnTypes(SqlManager.java:246)
        at org.apache.sqoop.manager.ConnManager.getColumnTypes(ConnManager.java:328)
        at org.apache.sqoop.orm.ClassWriter.getColumnTypes(ClassWriter.java:1853)
        at org.apache.sqoop.orm.ClassWriter.generate(ClassWriter.java:1653)
        at org.apache.sqoop.tool.CodeGenTool.generateORM(CodeGenTool.java:107)
        at org.apache.sqoop.tool.ImportTool.importTable(ImportTool.java:488)
        at org.apache.sqoop.tool.ImportTool.run(ImportTool.java:615)
        at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
        at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
        at org.apache.sqoop.Sqoop.runTool(Sqoop.java:225)
        at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
        at org.apache.sqoop.Sqoop.main(Sqoop.java:243)
```

## Appendix B: Links to Resources

For more info on [Reading and Writing Avro Files From the Command Line](http://www.michael-noll.com/blog/2013/03/17/reading-and-writing-avro-files-from-the-command-line/), Michael Noll's blog is an awesome resource.

The Avro tools jar can be downloaded from the [Avro releases page](https://avro.apache.org/releases.html).

Oracle resources:

 - Oracle thin drive connection string examples: http://www.orafaq.com/wiki/JDBC
 - Basic Oracle SQL commands for listing schemas, tables, and columns: http://www.anattatechnologies.com/q/2012/01/sqlplus-exploring-schemas-and-data/

Versions used in the Hortonworks HDP 2.5 sandbox:

 - Sqoop 1.4.6 ([User guide](https://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html))
 - Hadoop 2.7.3
 - Avro tools 1.8.1 (avro-tools-1.8.1.jar)
