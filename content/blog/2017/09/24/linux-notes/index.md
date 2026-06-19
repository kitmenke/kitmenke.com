---
author: Kit
date: 2017-09-24T10:53:33-05:00
lastmod: 2017-09-24T12:03:46-05:00
categories:
- Notes
title: Linux notes
---

A collection of code snippets and notes from working with linux. 

awk
---

```sh
# todo: length more than 6 then print
#FNR > 6 {
#   awk '{print $1}'
#}

# custom delimiter and sum
# | awk 'BEGIN {FS=":"}; {sum+=$3} END {printf "%.0f\n", sum}' && date
# 
hdfs dfs -count /apps/hive/warehouse/example.db/exampletbl/dt_date=2015-12-* | awk '{sum+=$3} END {printf "%.0f\n", sum}'
```

find
----

```sh
# find a file named sqljdbc4.jar and follow symlinks
find -L /usr/hdp/current -type f -name "sqljdbc4.jar"

#find . -name '*.py' -print0 | xargs -0 grep 'Processing identities'
#find . -type f -exec grep 2181 {}
#find . -name "*.xml" -type f -exec grep "2181" {} \;
```

iptables
--------

```sh
# Open up port 9042 in the firewall on the VM
#
# In CentOS you have the file /etc/sysconfig/iptables if you don't have it there, you can create it simply by using iptables-save to dump the current rule set into a file.

iptables-save > /etc/sysconfig/iptables
#To load the file you don't need to restart the machine, you can use iptables-restore

vi /etc/sysconfig/iptables
#Add port for cassandra as another line
#-A INPUT -p tcp -m tcp --dport 9042 -j ACCEPT

iptables-restore < /etc/sysconfig/iptables
```

Source: https://unix.stackexchange.com/questions/144600/how-to-modify-iptables-rules-via-editing-a-file-rather-than-interacting-via-comm

ntp
---

```sh
sudo yum install ntp
[kit@centos ~]$ sudo chkconfig ntpd on
[kit@centos ~]$ date
Wed May 24 11:04:28 EDT 2017

[kit@centos ~]$ sudo service ntpd stop
Shutting down ntpd:                                        [  OK  ]
[kit@centos ~]$ sudo ntpdate 10.10.192.15
25 Jul 18:16:26 ntpdate[24957]: step time server 10.10.192.15 offset 5382677.021993 sec
[kit@centos ~]$ sudo service ntpd start
Starting ntpd:                                             [  OK  ]
[kit@centos ~]$ date
Tue Jul 25 18:16:39 EDT 2017

```

Source: https://serverfault.com/questions/368602/how-do-i-update-a-centos-servers-time-from-an-authoritative-time-server

pdsh and pdcp
-------------
pdsh is a tool for executing commands across multiple nodes

pdcp is for copying files to multiple nodes

```sh
# initial setup - create a group of nodes
# become root:
sudo su -
mkdir -p .dsh/group
vi .dsh/group/c1
# c1 contains the hostnames of machines you want to run commands on
# example:
#   data4
#   data5
#   data6
# then you use pdsh to run a command on each server
# which is piped through dshbak which groups it by server
# ex:
pdsh -g c1 "tail /var/log/storm/metrics.log" | dshbak
pdsh -g c1 "date" | dshbak
----------------
data4
----------------
Thu Jun  5 14:35:57 EDT 2014
----------------
data5
----------------
Thu Jun  5 14:35:57 EDT 2014
----------------
data6
----------------
Thu Jun  5 14:35:57 EDT 2014
```

Other stuff

```sh
# this will give you the disk status on all nodes 
pdsh -g all 'df -h' | dshbak | less
pdsh -g all 'ls -l /usr/lib/test/' | dshbak | less

# copy files to all nodes (from ambari machine)
pdcp -g all /root/tdchtest/ /usr/lib/

# copy a dir (no slash on the end will copy entire dir)
pdcp -w hdpdata-01,hdpmst-01,hdpmst-02,hdpdata-02,hdpdata-03,hdpdata-04 -r /usr/lib/something /usr/lib/

# grep for something and display the output per node
pdsh -g c1 "grep 2014-06-11.*METER /var/log/storm/metrics.log | tail -100" | dshbak
pdsh -g c1 "grep METER /var/log/storm/metrics.log | tail -100" | dshbak
```

tar
---

Remember it this way: eXtract Ze Files or Compress Ze Files.

```sh
# compress file(s) into a .tar.gz file
tar -cvzf ~/supervisor.tar.gz supervisor.log
tar -cvzf ~/asdfadsfads.tar.gz -T ~/list.txt
tar -cvzf solr.tar.gz /hadoop/hdfs/datasdf/solr
tar -cvzf banana_shard1_replica2.tar.gz node1/solr/banana_shard1_replica2

# uncompress
tar -xvzf RateProcessingTopologyQ2-72-1438798709.tar.gz
```

grep
----

```sh
# only print match, use regex
grep -o -E "AVL[A-Z][0-9]{2}" avl_msgs.txt | sort > locations.txt

# find carriage return chars in files
-U treat the file as binary
grep -U $'\015' storm_rate_compress.sh
grep -rlU $'\015' /opt/apps/hive/

# find all matching files in directory and sub directories
grep -rl hive /opt/apps/hive/

# -r (or --recursive) option is used to traverse also all sub-directories of /path, whereas
# -l (or --files-with-matches) option is used to only print filenames of matching files, and not the matching lines (this could also improve the speed, given that grep stop reading a file at first match with this option).
```

Source: http://askubuntu.com/questions/55325/how-to-use-grep-command-to-find-text-including-subdirectories

grep and less working together

```sh
# get line number
grep -n Creat move-hdfs-log-files.log
# open file to that line number
less +577g move-hdfs-log-files.log
# get the line number for first match
grep -m 1 -n 2014-10-06 indexer.log
less +33979g indexer.log
```

misc
----

```sh
# split string on comma
echo $LIB_JARS | sed s/,/\\n/g | xargs ls -l
```

disk cleanup

```sh
#to get mount that is full
df -h
#cd to that mount
cd /var
#find largest files on that mount point
find . -mount -type f -exec du -m {} + | sort -nr | head -42
# then rm the archived log files
# ./hadoop/hdfs/hadoop-hdfs-datanode-hdpdata-04.log.5
rm -f ./hadoop/hdfs/hadoop-hdfs-datanode-hdpdata-04.log.*
```
locate

```sh
# find a list of hdfs XML files, pipe that list through group and show the output in less
locate -b hdfs-*.xml | xargs grep "replication" | less

# plus one line AFTER
locate -b hdfs-*.xml | xargs grep -A 1 "replication" | less
locate -b hadoop*.jar
```

sort

```sh
hdfs dfs -ls -h /apps/storm/batch | sort -k 7
```
