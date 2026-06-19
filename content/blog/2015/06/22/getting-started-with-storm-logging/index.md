---
author: Kit
categories:
- Hadoop
date: 2015-06-22T17:09:10Z
guid: http://kitmenke.com/blog/?p=782
id: 782
tags:
- Java
- Storm
title: 'Getting started with Storm: Logging'
---

Logging within storm uses Simple Logging Facade for Java (SLF4J).

To get started with SLF4J, include the dependencies in the dependencies section of your maven pom.xml:

```xml
<dependency>
   <groupId>org.slf4j</groupId>
   <artifactId>slf4j-api</artifactId>
   <version>1.7.5</version>
</dependency>
<dependency>
   <groupId>org.slf4j</groupId>
   <artifactId>slf4j-simple</artifactId>
   <version>1.7.5</version>
   <scope>test</scope>
</dependency>
```

Note: the version of SLF4J here should match the version which is installed in storm's lib directory.

Next, in your spouts and bolts include a reference to the org.slf4j.Logger object.

```java
public class WordCountBolt extends BaseBasicBolt {
   private static final Logger LOG = LoggerFactory.getLogger(WordCountBolt.class);
   
   Map<String, Intege> counts = new HashMap<String, Integer>();

   @Override
   public void execute(Tuple tuple, BasicOutputCollector collector) {
      try {
         DataBean bean = (DataBean)tuple.getValue(0);
         String word = bean.getWord();
         Integer count = counts.get(word);
         if (count == null)
            count = 0;
         count++;
         LOG.info("Count for {} is {}", word, count);
         counts.put(word, count);
         collector.emit(new Values(word, count));
      } catch (Exception e) {
         LOG.error("WordCountBolt error", e);
         collector.reportError(e);
      }		
   }

   @Override
   public void declareOutputFields(OutputFieldsDeclarer declarer) {
      declarer.declare(new Fields("word", "count"));
   }
}
```

Example project on github: <https://github.com/kitmenke/storm-stlhug-demo>

Another example from the storm-starter project here: [IntermediateRankingsBolt.java](https://github.com/nathanmarz/storm-starter/blob/master/src/jvm/storm/starter/bolt/IntermediateRankingsBolt.java).

Anything INFO or above should be logged to the storm logs. These logs are generated on each storm node in /var/log/storm. A file will be generated for each worker (based on topology name and port), supervisor.log, ui.log, and metrics.log.

In order to view the logs directly, you would need to SSH directly to each node. A better alternative is to use the Storm UI. If you are using the HortonWorks distribution, the UI is accessible from Ambari by going to Storm -> Quick Links -> Storm UI (like http://<server>:8744/index.html).

After clicking on the spout or bolt, click on the port number to view the logs for all of the <span style="text-decoration: underline;">executors in that worker</span>. [<img class="alignnone size-full wp-image-786" src="/uploads/2015/06/storm-ui-executors1.png" alt="storm-ui-executors" width="1019" height="200" srcset="/uploads/2015/06/storm-ui-executors1.png 1019w, /uploads/2015/06/storm-ui-executors1-300x59.png 300w, /uploads/2015/06/storm-ui-executors1-624x122.png 624w" sizes="(max-width: 1019px) 100vw, 1019px" />](/uploads/2015/06/storm-ui-executors1.png)