---
author: Kit
categories:
- Hadoop
date: 2014-08-04T14:05:45Z
guid: http://kitmenke.com/blog/?p=704
id: 704
tags:
- Java
- Storm
title: Tick tuples within Storm
---

Tick tuples are useful when you want to execute some logic within your bolts on a schedule. For example, refreshing a cache every 5 minutes.

Within your **bolt**, first enable receiving the tick tuples and configure how often you want to receive them:

```java
@Override
public Map<String, Object> getComponentConfiguration() {
    // configure how often a tick tuple will be sent to our bolt
    Config conf = new Config();
    conf.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, 300);
    return conf;
}
```

Next create the isTickTuple helper method to determine whether or not we've received a tick tuple:

```java
protected static boolean isTickTuple(Tuple tuple) {
    return tuple.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
        && tuple.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID);
}
```

Note: the tick tuples your bolt receives will be mixed in with your other normal tuples you are processing.

Finally, you can add a small block in your bolt's execute method:

```java
@Override
public void execute(Tuple tuple, BasicOutputCollector collector) {
    try {
        if (isTickTuple(tuple)) {
        _cache.rotate();
        return;
        }
        
        // do your bolt stuff
    } catch (Exception e) {
        LOG.error("Bolt execute error: {}", e);
        collector.reportError(e);
    }
}
```

Now your bolt will receive a tick tuple every 300 seconds!