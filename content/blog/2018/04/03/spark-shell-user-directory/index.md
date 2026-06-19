---
author: Kit
categories: 
- Hadoop
date: 2018-04-03T11:50:00Z
guid: http://kitmenke.com/blog/?p=840
id: 840
tags:
- Spark
title: Run Spark Shell with a Different User Directory
---

I needed to be able to the run Spark's shell using a different home directory due to a permissions
issue with my normal HDFS home directory. A little known feature with spark is that it allows
overriding Hadoop configuration properties! 

Command:
```
spark-shell --master yarn --conf spark.hadoop.dfs.user.home.dir.prefix=/tmp/user
```

Normal Spark configuration parameters can be provided using `--conf` but you can also override 
Hadoop configurations as well by prefixing the normal Hadoop property name with `spark.hadoop`.

In this case, the Hadoop property is `dfs.user.home.dir.prefix` which by default is set to `/user`.
By passing `spark.hadoop.dfs.user.home.dir.prefix` that value becomes `/tmp/user`. Now when I 
start up Spark shell a directory is created at `/tmp/user/[my username]`!
