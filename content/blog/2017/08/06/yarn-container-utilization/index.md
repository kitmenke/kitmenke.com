---
tags: ["Hadoop"]
title: YARN container utilization
author: Kit
categories: ["Hadoop"]
date: 2017-08-06T20:38:16-05:00
draft: true
---

Optimizing cluster utilization. YARN container metrics. Show metrics for YARN container memory utilization.  

## adfadf

 
 
Were false, set to true

yarn.nodemanager.container-metrics.enable	true	Flag to enable container metrics

yarn.nodemanager.pmem-check-enabled	true	Whether physical memory limits will be enforced for containers.
yarn.nodemanager.vmem-check-enabled	true	Whether virtual memory limits will be enforced for containers.
yarn.nodemanager.vmem-pmem-ratio	2.1
 
2017-08-04 20:29:12,682 INFO  monitor.ContainersMonitorImpl (ContainersMonitorImpl.java:run(464)) - Memory usage of ProcessTree 34555 for container-id container_e06_1499802738262_0004_01_000001: 404.7 MB of 1.5 GB physical memory used; 2.7 GB of 3.1 GB virtual memory used

As a part of [YARN-2984](https://issues.apache.org/jira/browse/YARN-2984) the feature was added in v2.7.0 

[YARN-2984] Metrics for container's actual memory usage ...
issues.apache.org
It would be nice to capture resource usage per container, for a variety of reasons. This JIRA is to track memory usage. YARN-2965 tracks the resource usage on the ...

yarn.nodemanager.container-metrics.enable default is true
https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-common/yarn-default.xml
hadoop.apache.org
hadoop.apache.org
Factory to create client IPC classes. yarn.ipc.client.factory.class Factory to create server IPC classes. yarn.ipc.server.factory.class Factory to create ...

 
 
 