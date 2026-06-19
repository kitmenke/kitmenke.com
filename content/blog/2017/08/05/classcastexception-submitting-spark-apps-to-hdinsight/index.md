---
tags: ["Hadoop","Spark", "Azure"]
title: ClassCastException submitting Spark apps to HDInsight
author: Kit
categories: ["Hadoop"]
date: 2017-08-05T11:20:10-05:00
---

I recently ran into an issue submitting Spark applications to a HDInsight cluster. The job would run fine until it attempted to use files in blob storage and then blow up with an exception: `java.lang.ClassCastException: org.apache.xerces.parsers.XIncludeAwareParserConfiguration cannot be cast to org.apache.xerces.xni.parser.XMLParserConfiguration`.  

## The Problem

We also noticed that some spark apps worked and others didn't. Everything compiled, submitted to the cluster, and started fine. In our case the files were being accessed through Hive and Spark SQL so we thought at first it was a Hive issue overwriting tables.

Most of the stack trace here, notice the last exception:
```
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: Getting globStatus wasb://<our blob container>@<our blob account>.blob.core.windows.net/hive/warehouse/<our database>.db/<our table>/.hive-staging_hive_2017-06-29_21-15-33_536_3216662945768617306-1/-ext-10000
        at org.apache.hadoop.hive.ql.metadata.Hive.replaceFiles(Hive.java:2834)
        at org.apache.hadoop.hive.ql.metadata.Hive.loadTable(Hive.java:1640)
        ... 54 more
Caused by: org.apache.hadoop.fs.azure.AzureException: java.util.NoSuchElementException: An error occurred while enumerating the result, check the original exception for details.
        at org.apache.hadoop.fs.azure.AzureNativeFileSystemStore.retrieveMetadata(AzureNativeFileSystemStore.java:2024)
        at org.apache.hadoop.fs.azure.NativeAzureFileSystem.getFileStatus(NativeAzureFileSystem.java:2050)
        at org.apache.hadoop.fs.FileSystem.exists(FileSystem.java:1443)
        at org.apache.hadoop.fs.azure.NativeAzureFileSystem.conditionalRedoFolderRename(NativeAzureFileSystem.java:2106)
        at org.apache.hadoop.fs.azure.NativeAzureFileSystem.getFileStatus(NativeAzureFileSystem.java:2073)
        at org.apache.hadoop.fs.Globber.getFileStatus(Globber.java:57)
        at org.apache.hadoop.fs.Globber.glob(Globber.java:252)
        at org.apache.hadoop.fs.FileSystem.globStatus(FileSystem.java:1674)
        at org.apache.hadoop.hive.ql.metadata.Hive.replaceFiles(Hive.java:2832)
        ... 55 more
Caused by: java.util.NoSuchElementException: An error occurred while enumerating the result, check the original exception for details.
        at com.microsoft.azure.storage.core.LazySegmentedIterator.hasNext(LazySegmentedIterator.java:113)
        at org.apache.hadoop.fs.azure.StorageInterfaceImpl$WrappingIterator.hasNext(StorageInterfaceImpl.java:130)
        at org.apache.hadoop.fs.azure.AzureNativeFileSystemStore.retrieveMetadata(AzureNativeFileSystemStore.java:2003)
        ... 63 more
Caused by: com.microsoft.azure.storage.StorageException: The server encountered an unknown failure: OK
        at com.microsoft.azure.storage.StorageException.translateException(StorageException.java:101)
        at com.microsoft.azure.storage.core.ExecutionEngine.executeWithRetry(ExecutionEngine.java:199)
        at com.microsoft.azure.storage.core.LazySegmentedIterator.hasNext(LazySegmentedIterator.java:109)
        ... 65 more
Caused by: java.lang.ClassCastException: org.apache.xerces.parsers.XIncludeAwareParserConfiguration cannot be cast to org.apache.xerces.xni.parser.XMLParserConfiguration
        at org.apache.xerces.parsers.SAXParser.<init>(Unknown Source)
        at org.apache.xerces.parsers.SAXParser.<init>(Unknown Source)
        at org.apache.xerces.jaxp.SAXParserImpl$JAXPSAXParser.<init>(Unknown Source)
        at org.apache.xerces.jaxp.SAXParserImpl.<init>(Unknown Source)
        at org.apache.xerces.jaxp.SAXParserFactoryImpl.newSAXParser(Unknown Source)
        at com.microsoft.azure.storage.core.Utility.getSAXParser(Utility.java:668)
        at com.microsoft.azure.storage.blob.BlobListHandler.getBlobList(BlobListHandler.java:72)
        at com.microsoft.azure.storage.blob.CloudBlobContainer$6.postProcessResponse(CloudBlobContainer.java:1284)
        at com.microsoft.azure.storage.blob.CloudBlobContainer$6.postProcessResponse(CloudBlobContainer.java:1248)
        at com.microsoft.azure.storage.core.ExecutionEngine.executeWithRetry(ExecutionEngine.java:146)
        ... 66 more
```

## The Solution

After a lot of trial and error including opening a ticket with Microsoft (who were very helpful even over the July 4th holiday) we discovered the root cause of the issue was that we were using a custom spark submit script and was missing a flag which defined the SAXParserFactory.

```
-Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl
```

The default as specified in Ambari: ![Ambari Spark Options](/uploads/2017/08/ambari-spark-options.png "Ambari Spark Options")

Full text:

```
-Dhdp.version={{hdp_full_version}} -Detwlogger.component=sparkdriver -DlogFilter.filename=SparkLogFilters.xml -DpatternGroup.filename=SparkPatternGroups.xml -Dlog4jspark.root.logger=INFO,console,RFA,ETW,Anonymizer -Dlog4jspark.log.dir=/var/log/sparkapp/${user.name} -Dlog4jspark.log.file=sparkdriver.log -Dlog4j.configuration=file:/usr/hdp/current/spark-client/conf/log4j.properties -Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl
```

So... when overriding `--driver-java-options` / `spark.driver.extraJavaOptions` or `spark.executor.extraJavaOptions` with HDInsight don't forget to include `-Djavax.xml.parsers.SAXParserFactory`!