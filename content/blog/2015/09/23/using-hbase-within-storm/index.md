---
author: Kit
categories:
- Hadoop
date: 2015-09-23T09:03:12Z
guid: http://kitmenke.com/blog/?p=812
id: 812
tags:
- HBase
- Java
- Storm
title: Using HBase within Storm
---

There is a lot of documentation around [Apache Storm](https://storm.apache.org/) and [Apache HBase](http://hbase.apache.org/) but not so much about how to use the hbase-client inside of storm. In this post, I'll outline:

  1. Information about my dev environment
  2. Setting up your Storm project to use the HBase client
  3. Managing connections to HBase in Storm
  4. Reading one row (Get)
  5. Reading many rows (Scan)
  6. Writing one row (Put)
  7. Writing many rows in a batch of Puts

Please note, this post assumes you already are comfortable with Storm and HBase terminology. If you are just starting out with Storm, take a look at my example project on GitHub: <a href="https://github.com/kitmenke/storm-stlhug-demo" data-pjax="#js-repo-pjax-container">storm-stlhug-demo</a>.

Also, an option to consider when writing to HBase from storm is <a href="https://github.com/ptgoetz/storm-hbase" rel="nofollow">storm-hbase</a> and it is a great way to start streaming data into hbase. However, if you need to write to multiple tables or get into more advanced scenarios you will need to understand how to write your own HBase bolts.

### My Environment

Some quick notes about my environment:

  * Eclipse Mars (comes with the maven and git plugins)
  * Storm 0.9.3
  * HBase 0.98.4
  * Maven

## Setting up your Project

Assuming you're using maven and the maven-shade plugin...

To get started you'll need to edit your pom.xml to reference the hbase-client. The version of the hbase-client you're using should match the version of HBase running on your cluster.

```xml
<dependency>
   <groupId>org.apache.hbase</groupId>
   <artifactId>hbase-client</artifactId>
   <version>${hbase.version}</version>
</dependency>
```

Also make sure to package hbase-site.xml in your topology jar. You can download this file from your cluster and just put it in src/main/resources. I also have one for testing in dev named hbase-site.dev.xml. Then just use the shade plugin to move it to the root of the jar.

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-shade-plugin</artifactId>
   <version>2.4</version>
   <configuration>
      <createDependencyReducedPom>true</createDependencyReducedPom>
      <artifactSet>
         <excludes>
            <exclude>classworlds:classworlds</exclude>
            <exclude>junit:junit</exclude>
            <exclude>jmock:*</exclude>
            <exclude>*:xml-apis</exclude>
            <exclude>org.apache.maven:lib:tests</exclude>
            <exclude>log4j:log4j:jar:</exclude>
            <exclude>org.testng:testng</exclude>
         </excludes>
      </artifactSet>
   </configuration>
   <executions>
      <execution>
         <phase>package</phase>
         <goals>
            <goal>shade</goal>
         </goals>
         <configuration>
            <transformers>
               <transformer implementation="org.apache.maven.plugins.shade.resource.IncludeResourceTransformer">
                       <resource>core-site.xml</resource>
                       <file>src/main/resources/core-site.xml</file>
                   </transformer>
                   <transformer implementation="org.apache.maven.plugins.shade.resource.IncludeResourceTransformer">
                       <resource>hbase-site.xml</resource>
                       <file>src/main/resources/hbase-site.xml</file>
                   </transformer>
                   <transformer implementation="org.apache.maven.plugins.shade.resource.IncludeResourceTransformer">
                       <resource>hdfs-site.xml</resource>
                       <file>src/main/resources/hdfs-site.xml</file>
                   </transformer>
               <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
               <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                  <mainClass></mainClass>
               </transformer>
            </transformers>
            <filters>
               <filter>
                  <artifact>*:*</artifact>
                  <excludes>
                     <exclude>META-INF/*.SF</exclude>
                     <exclude>META-INF/*.DSA</exclude>
                     <exclude>META-INF/*.RSA</exclude>
                     <exclude>junit/*</exclude>
                     <exclude>webapps/</exclude>
                     <exclude>testng*</exclude>
                     <exclude>*.js</exclude>
                     <exclude>*.png</exclude>
                     <exclude>*.css</exclude>
                     <exclude>*.json</exclude>
                     <exclude>*.csv</exclude>
                  </excludes>
               </filter>
            </filters>
         </configuration>
      </execution>
   </executions>
</plugin>
```

Note: I have lines in there for the other configs I use so remove them if you don't need them. As an aside, I don't really like packaging the configs like this BUT... it makes setting up the HBase connection much easier and solves a bunch of weird connection errors.

## Managing HBase Connections in Storm

The most important thing is to create **one HConnection for each instance of your bolt** in the prepare method and then **re-use that connection** for the entire lifetime of the bolt!

```java
private HConnection connection;

@SuppressWarnings("rawtypes")
@Override
public void prepare(java.util.Map stormConf, backtype.storm.task.TopologyContext context) {
   Configuration config = HBaseConfiguration.create();
   connection = HConnectionManager.createConnection(config);
}

@Override
public void execute(Tuple tuple, BasicOutputCollector collector) {
   try {
      // use the connection
   } catch (Exception e) {
      LOG.error("bolt error", e);
      collector.reportError(e);
   }
}

@Override
public void cleanup() {
  LOG.info("cleanup called");
  try {
     connection.close();
     LOG.info("hbase closed");
  } catch (Exception e) {
     LOG.error("cleanup error", e);
  }
}
```

First, on line 7, the connection is created in the **prepare** method.

Second, on line 13, the connection is used repeatedly in the **execute** method.

Finally, on line 24, it is probably a good idea to try to close your HBase connection in **cleanup**. Just be aware it might not be called before your worker is killed.

### Example Data

For the next couple examples, we'll be using an example HBase table filled with different fruit. The table is named &#8220;fruit&#8221;, has one column family named &#8220;cf1&#8221;, and two columns in that column family named &#8220;COLOR&#8221; and &#8220;COUNT&#8221;.

<table>
	<thead>
		<tr>
			<th>KEY</th>
			<th>cf1:COLOR</th>
			<th>cf1:COUNT</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>APPLE_FUJI</td>
			<td>red</td>
			<td>4</td>
		</tr>
		<tr>
			<td>APPLE_GRANNY_SMITH</td>
			<td>green</td>
			<td>7</td>
		</tr>
		<tr>
			<td>APPLE_RED_DELICIOUS</td>
			<td>red</td>
			<td>3</td>
		</tr>
		<tr>
			<td>ORANGE_VALENCIA</td>
			<td>orange</td>
			<td>11</td>
		</tr>
	</tbody>
</table>

### Constants.java

```java
static final String TABLE_FRUIT = "fruit";
static final byte[] COLUMN_FAMILY_FRUIT = "cf1".getBytes();
static final byte[] COLUMN_COLOR = "COLOR".getBytes();
static final byte[] COLUMN_COUNT = "COUNT".getBytes();
```

### Read one row from HBase (Get)

To return one row from HBase, use a Get. To get the row for &#8220;APPLE_FUJI&#8221;:

```java
String key = "APPLE_FUJI";
HTableInterface table = connection.getTable(Constants.TABLE_FRUIT);
try {
   Get g = new Get(key.getBytes());
   Result result = table.get(g);
   String color = Bytes.toString(result.getValue(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COLOR));
   // color = red
   int count = Bytes.toInt(result.getValue(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COUNT));
   // count = 4
} finally {
   table.close();
}
```

### Read many rows from HBase (Scan)

To retrieve multiple sequential rows from HBase, use a Scan. For example, to get all &#8220;APPLE&#8221; rows:

```java
String prefix = "APPLE";
HTableInterface table = connection.getTable(Constants.TABLE_FRUIT);
try {
   Scan scan = new Scan(Bytes.toBytes(prefix));
   
   // only return rows which begin with APPLE
   Filter prefixFilter = new PrefixFilter(Bytes.toBytes(prefix));
   scan.setFilter(prefixFilter);

   ResultScanner resultScanner = table.getScanner(scan);
   
   // iterate over each result
   // APPLE_FUJI	 red 	4
   // APPLE_GRANNY_SMITH	 green	 7
   // APPLE_RED_DELICIOUS	 red	 3
   for (Result result : resultScanner) {
      String key = Bytes.toString(result.getRow());
      String color = Bytes.toString(result.getValue(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COLOR));
      int count = Bytes.toInt(result.getValue(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COUNT));
      // do something with key, color, or count
   }
} finally {
   table.close();
}
```

### Write one row into HBase (Put)

To write one row at a time to HBase, use a Put and then immediately close the table.

```java
String key = "APPLE_FUJI";
String color = "red";
int count = 4;

HTableInterface table = connection.getTable(Constants.TABLE_FRUIT);
try {
   Put p = new Put(key.getBytes());
   p.add(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COLOR, Bytes.toBytes(color));
   p.add(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COUNT, Bytes.toBytes(count));
   table.put(p);
} finally {
   table.close();
}
```

Eventually this won't scale due to the number of requests you're trying to send across the network and you'll need to start batching multiple PUTs together.

### Write many rows into HBase (Put with setAutoFlush(false, ...))

In order to batch PUTs, you'll need to open a table using the HConnection, set autoFlush to false by calling table.setAutoFlush, and then keep the HTable open.

> When performing a lot of Puts, make sure that setAutoFlush is set to false on your [Table](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Table.html) instance. Otherwise, the Puts will be sent one at a time to the RegionServer. Puts added via table.add(Put) and table.add( <List> Put) wind up in the same write buffer. If autoFlush = false, these messages are not sent until the write-buffer is filled. To explicitly flush the messages, call flushCommits. Calling close on theTable instance will invoke flushCommits.
  
> [From the HBase book 89.4](http://hbase.apache.org/book.html#perf.hbase.client.autoflush)

Setting autoFlush to false means the table will automatically buffer requests until it reaches the &#8220;hbase.client.write.buffer&#8221; size (default is 2097152).

```java
private HConnection connection;
private static boolean AUTO_FLUSH = false;
private static boolean CLEAR_BUFFER_ON_FAIL = false;
private HTableInterface table;

@SuppressWarnings("rawtypes")
@Override
public void prepare(java.util.Map stormConf, backtype.storm.task.TopologyContext context) {
   Configuration config = HBaseConfiguration.create();
   connection = HConnectionManager.createConnection(config);
   table = connection.getTable(Constants.TABLE_FRUIT);
   table.setAutoFlush(AUTO_FLUSH, CLEAR_BUFFER_ON_FAIL);
}

@Override
public void execute(Tuple tuple, BasicOutputCollector collector) {
   try {
      // read data from the tuple, hard coded as an example
      String key = "APPLE_FUJI";
      String color = "red";
      int count = 4;

      // put the data into hbase
      Put p = new Put(key.getBytes());
      p.add(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COLOR, Bytes.toBytes(color));
      p.add(Constants.COLUMN_FAMILY_FRUIT, Constants.COLUMN_COUNT, Bytes.toBytes(count));
      table.put(p);
      // not closing the table here anymore!
   } catch (Exception e) {
      LOG.error("bolt error", e);
      collector.reportError(e);
   }
}

@Override
public void cleanup() {
  LOG.info("cleanup called");
  try {
     table.close();
     connection.close();
     LOG.info("hbase closed");
  } catch (Exception e) {
     LOG.error("cleanup error", e);
  }
}
```

**Note:** This blog post was adapted from my answer on [Approach to insert and delete values in HBase from Apache Storm bolt](http://stackoverflow.com/q/32128158/98933).