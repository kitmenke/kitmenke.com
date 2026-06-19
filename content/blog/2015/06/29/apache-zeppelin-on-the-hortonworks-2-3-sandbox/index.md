---
author: Kit
categories:
- Hadoop
date: 2015-06-29T21:23:19Z
guid: http://kitmenke.com/blog/?p=791
id: 791
title: Apache Zeppelin on the Hortonworks 2.3 sandbox
---

A few notes from playing with zeppelin on the Hortonworks HDP 2.3 sandbox. 

[<img class="alignnone size-medium wp-image-795" src="/uploads/2015/06/zeppelin-300x231.png" alt="zeppelin" width="300" height="231" srcset="/uploads/2015/06/zeppelin-300x231.png 300w, /uploads/2015/06/zeppelin-1024x789.png 1024w, /uploads/2015/06/zeppelin-624x481.png 624w, /uploads/2015/06/zeppelin.png 1273w" sizes="(max-width: 300px) 100vw, 300px" />](/uploads/2015/06/zeppelin.png)

Download the HDP 2.3 sandbox from the [Hortonworks download site](http://hortonworks.com/products/hortonworks-sandbox/#install).

Install maven first since we'll compile zeppelin from source: [Download Apache Maven 3.3.3](https://maven.apache.org/download.cgi). I just downloaded the binary tar.gz and installed in to /usr/local/bin since we just need it once.

Most of these instructions came from [Introduction to Data Science with Apache Spark](http://hortonworks.com/blog/introduction-to-data-science-with-apache-spark/):

```bash
git clone https://github.com/apache/incubator-zeppelin.git
cd incubator-zeppelin
mvn clean install -DskipTests -Pspark-1.3 -Dspark.version=1.3.1 -Phadoop-2.6 -Pyarn
# go get a coffee, takes about 15 minutes to complete
cd conf
cp /etc/hive/conf/hive-site.xml .
cp zeppelin-env.sh.template zeppelin-env.sh
cp zeppelin-site.xml.template zeppelin-site.xml
vi hive-site.xml
# search for hive.metastore.client.connect.retry.delay
# change<value>5s</value> with <value>5</value> otherwise you get issue #1 below
# search for hive.metastore.client.socket.timeout
# change to <value>1800</value>
# :wq
vi zeppelin-site.xml
# search for zeppelin.server.port
# change to <value>10008</value> otherwise it conflicts with ambari
```

### Issue #1: java.lang.NumberFormatException: For input string: &#8220;5s&#8221;

java.lang.NumberFormatException: For input string: &#8220;5s&#8221;

Fixed by editing conf/hive-site.xml and changing 5s to 5. https://issues.apache.org/jira/browse/ZEPPELIN-93

### **Issue #2: hql interpreter not found**

```
hql interpreter not found
org.apache.zeppelin.notebook.NoteInterpreterLoader.get(NoteInterpreterLoader.java:148)
org.apache.zeppelin.notebook.Note.run(Note.java:267)
org.apache.zeppelin.socket.NotebookServer.runParagraph(NotebookServer.java:534)
org.apache.zeppelin.socket.NotebookServer.onMessage(NotebookServer.java:119)
org.java_websocket.server.WebSocketServer.onWebsocketMessage(WebSocketServer.java:469)
org.java_websocket.WebSocketImpl.decodeFrames(WebSocketImpl.java:368)
org.java_websocket.WebSocketImpl.decode(WebSocketImpl.java:157)
org.java_websocket.server.WebSocketServer$WebSocketWorker.run(WebSocketServer.java:657)
```

I was trying to use %hql, but it should have been %hive.hql:

```
%hive.hql
select * from default.sample_08
```

&nbsp;