---
title: Writing a Spark DataFrame to ORC files
author: Kit
categories: ["Hadoop"]
date: 2016-12-12T20:10:54-06:00
guid: http://kitmenke.com/blog/?p=840
id: 840
tags: ["Spark"]
---

Spark includes the ability to write multiple different file formats to HDFS. One of those is [ORC](https://orc.apache.org/) which is columnar file format featuring great compression and improved query performance through Hive. 

You'll need to create a HiveContext in order to write using the ORC data source in Spark. First, create some properties in your pom.xml:
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <scala.core>2.10</scala.core>
  <spark.version>1.6.1</spark.version>
</properties>
```

Include spark-hive in addition to your other project dependencies:
```xml
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-core_${scala.core}</artifactId>
  <version>${spark.version}</version>
</dependency>
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-sql_${scala.core}</artifactId>
  <version>${spark.version}</version>
</dependency>
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-hive_${scala.core}</artifactId>
  <version>${spark.version}</version>
</dependency>
```

Then in your code:
```scala
// create a new hive context from the spark context
val hiveContext = new org.apache.spark.sql.hive.HiveContext(sparkContext)
// create the data frame and write it to orc
// output will be a directory of orc files
val df = hiveContext.createDataFrame(rdd)
df.write.mode(SaveMode.Overwrite).format("orc")
 .save("/tmp/myapp.orc/")
 ```

If you want your table to be accessible via Hive the directory could be the location of the internal hive table like `/apps/hive/warehouse/some.db/some_table/` or somewhere that a Hive external table points to.

Alternatively, if you want to handle the table creation entirely within Spark with the data stored as ORC, just register a Spark SQL temp table and run some HQL to create the table.

```scala
df.registerTempTable("my_temp_table")
hiveContext.sql("CREATE TABLE new_table_name STORED AS ORC  AS SELECT * from my_temp_table")
```

Sources:

 - [Good example of how to write ORC files from Spark](https://github.com/rajkrrsingh/SparkORCWriter)
 - [How do I create an ORC Hive table from Spark?](https://community.hortonworks.com/questions/4292/how-do-i-create-an-orc-hive-table-from-spark.html)
 - [How to save a dataframe as ORC file ?](https://community.hortonworks.com/questions/70622/how-to-save-a-dataframe-as-orc-file.html)