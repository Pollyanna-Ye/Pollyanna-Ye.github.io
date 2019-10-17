---
layout:     post
title:      Hadoop Rack Awareness Configuration
subtitle:   Showing How To Configure Rack Information In Hadoop
date:       2019-10-15
author:     Susu
header-img: img/posts20191015-1/header_bg.jpg
catalog: 	 true
tags:
    - Big Data
    - Hadoop
    - Hadoop Installation
---

## Before Start
If you are running Hadoop in cluster instead of cloud, you may want to configure the rack information in Hadoop, so that you can make full use of Hadoop rack-awareness.

Before start, you should have Hadoop installed in your cluster.  You can refer [this blog](https://pollyanna-ye.github.io/2019/10/13/Install-Hadoop-In-Fully-Distributed-Mode/) to install Hadoop in fully distributed mode.  

Following is my configuration:
1. Hadoop version I use is 2.9.2, which is installed under directory `/home/qianwen`
2. Hadoop is installed on four VMs. `falcon-1` is the master node, `falcon-2 (10.102.2.33)`, `falcon-3 (10.102.2.41)`, `falcon-4 (10.102.2.49)` are slave nodes.

## Start Configuring
enter Hadoop configuration directory:
```
cd ~/hadoop-2.9.2/etc/hadoop
```
then create a new file `rack_topology.data`:
```
vim rack_topology.data
```
add the following content:
```
10.102.2.33 /rack01    
10.102.2.41 /rack01
10.102.2.49 /rack02
```
note that the first column is the IP address of `falcon-2`, `falcon-3`,`falcon-4`, and the second column lists their corresponding rack information.

**Note that you have to put IP address in the first column, putting host name won't work.**

If you don't configure it, then the unconfigured host will be in /default_rack.

After editing `rack_topology.data`, create a file `rack_topology.sh` and add the following content

```
#!/bin/bash
HADOOP_CONF=/home/qianwen/hadoop-2.9.2/etc/hadoop
#while [ $# -gt 0 ] ; do
  nodeArg=$1
  exec<${HADOOP_CONF}/rack_topology.data
  result=""
  while read line ; do
    ar=( $line )
    if [ "${ar[0]}" = "$nodeArg" ]; then
      result="${ar[1]}"
    fi
  done
  shift
  if [ -z "$result" ] ; then
    echo -n "/default-rack"
  else
    echo -n "$result"
  fi
# done
```
Note that the HADOOP_CONF should be changed to the **abusolte path** of your Hadoop configuration path.

Also, if the name of the afore-created file is not `rack_topology.data`, you have to also change the second circled part.

![](https://raw.githubusercontent.com/Pollyanna-Ye/Pollyanna-Ye.github.io/master/img/posts20191015-1/001.jpg)

Then change the mode of this file to make it executable:

```
chmod 777 rack_topology.sh
```

Then, add the following content in `core-site.xml` of Hadoop, also make sure the path is the `rack_topology.sh` path your computer:
```
<property>
    <name>topology.script.file.name</name>
    <value>/home/qianwen/hadoop-2.9.2/etc/hadoop/rack_topology.sh</value>
</property>
```

## Check Configuration Result

start HDFS deamon first:
```
start-dfs.sh
```
then using the following command to check rack information:
```
hdfs dfsadmin -printTopology
```
You should see the following:
![](https://raw.githubusercontent.com/Pollyanna-Ye/Pollyanna-Ye.github.io/master/img/posts20191015-1/002.jpg)


## Reference
[http://fibrevillage.com/storage/638-hadoop-rack-awareness-what-s-it-and-how-to-config](http://fibrevillage.com/storage/638-hadoop-rack-awareness-what-s-it-and-how-to-config)

