---
author: Kit
date: 2017-09-24T10:53:33-05:00
lastmod: 2017-09-24T12:03:46-05:00
categories: ["Hadoop"]
tags: ["Storm"]
title: Apache Storm and ulimits

---

Running Apache Storm in production requires increasing the nofile and nproc default ulimits. 

Issues
------

If the defaults aren't changed, running Storm will result in a variety of weird errors.

Symptom
```
Exception in thread "main" java.io.FileNotFoundException: /tmp/hadoop-unjar6969050838584667868/org/apache/hive/service/cli/thrift/TGetFunctionsResp$TGetFunctionsRespTupleScheme.class (No space left on device)
        at java.io.FileOutputStream.open(Native Method)
        at java.io.FileOutputStream.<init>(FileOutputStream.java:221)
        at java.io.FileOutputStream.<init>(FileOutputStream.java:171)
        at org.apache.hadoop.util.RunJar.unJar(RunJar.java:105)
        at org.apache.hadoop.util.RunJar.unJar(RunJar.java:81)
        at org.apache.hadoop.util.RunJar.run(RunJar.java:209)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
```

Troubleshooting
```sh
# How many inodes are currently being used?
df -i /tmp

Filesystem           Inodes IUsed IFree IUse% Mounted on
/dev/mapper/vg_root-lv_tmp
                      98304 97079  1225   99% /tmp
# Uh oh, 99% of the available inodes are being used

# More symptoms
# running commands on the server could result in
# -bash: fork: retry: Resource temporarily unavailable
# -bash: fork: Cannot allocate memory

# How many processes does storm have?
sudo ps -eLF -U storm | wc -l
# 3380
# How many open files does storm have?
sudo lsof | grep storm | wc -l
#4450
# Whats the default limit for max # of files open?
ulimit -Hn
#4096
# Whats the default limit for max # of processes?
ulimit -Sn
#1024
```

Increasing the ulimit
```sh
# Change the limit as root here:
vim /etc/security/limits.conf

# Add lines:
storm soft nofile 64000
storm hard nofile 64000
storm soft nproc 10240
storm hard nproc 10240
# Apply these changes to the rest of the nodes in the cluster
```

Resources:

 - [Find where inodes are being used](http://unix.stackexchange.com/questions/117093/find-where-inodes-are-being-used)
 - [Cannot switch, ssh to specific user: su: cannot set user id: Resource temporarily unavailable?](http://serverfault.com/questions/412114/cannot-switch-ssh-to-specific-user-su-cannot-set-user-id-resource-temporaril)
 - [Linux Increase The Maximum Number Of Open Files / File Descriptors (FD)](http://www.cyberciti.biz/faq/linux-increase-the-maximum-number-of-open-files/)

