---
type: post
layout: post
blog: true
title: Hadoop HDFS on EC2 Tips
categories: tech
tags: hadoop, hdfs
---

I deployed a YARN cluster with HDFS on EC2 using a [forked version](https://github.com/milinda/yarn-ec2) of [yarn-ec2](https://github.com/tqchen/yarn-ec2) and submit a YARN application. But the application submission failed with following error:

```
01:20:05.125 [main] INFO  o.p.fdbench.yarn.YarnClientWrapper - Creating YARN application....
01:20:05.380 [main] INFO  o.p.fdbench.yarn.YarnClientWrapper - YARN application was created with id application_1475642094699_0004
01:20:05.380 [main] INFO  o.p.fdbench.yarn.YarnClientWrapper - Preparing to request resources for app id application_1475642094699_0004
01:20:05.691 [main] INFO  o.p.fdbench.yarn.YarnClientWrapper - Copying file /Users/mpathira/PhD/Code/FDBench/fdbench-yarn/build/distributions/fdbench-yarn-0.1-SNAPSHOT-dist.tgz to hdfs://ec2-52-26-132-68.us-west-2.compute.amazonaws.com:9000/user/ubuntu/fdbench-producer-throughput-28/application_1475642094699_0004/package.tgz
16/10/05 01:21:06 INFO hdfs.DFSClient: Exception in createBlockOutputStream
org.apache.hadoop.net.ConnectTimeoutException: 60000 millis timeout while waiting for channel to be ready for connect. ch : java.nio.channels.SocketChannel[connection-pending remote=/172.31.12.45:50010]
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:534)
	at org.apache.hadoop.hdfs.DFSOutputStream.createSocketForPipeline(DFSOutputStream.java:1508)
	at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.createBlockOutputStream(DFSOutputStream.java:1284)
	at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.nextBlockOutputStream(DFSOutputStream.java:1237)
	at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:449)
16/10/05 01:21:06 INFO hdfs.DFSClient: Abandoning BP-822330611-172.31.12.45-1475643620638:blk_1073741826_1002
16/10/05 01:21:06 INFO hdfs.DFSClient: Excluding datanode DatanodeInfoWithStorage[172.31.12.45:50010,DS-6d3ed80c-8826-4510-a0c6-e17d914a45a5,DISK]
....
....
Caused by: org.apache.hadoop.ipc.RemoteException(java.io.IOException): File /user/ubuntu/fdbench-producer-throughput-28/application_1475642094699_0004/package.tgz could only be replicated to 0 nodes instead of minReplication (=1).  There are 1 datanode(s) running and 1 node(s) are excluded in this operation.
	at org.apache.hadoop.hdfs.server.blockmanagement.BlockManager.chooseTarget4NewBlock(BlockManager.java:1571)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getNewBlockTargets(FSNamesystem.java:3107)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalBlock(FSNamesystem.java:3031)
....
```

It turns out that this is happening mainly because use of EC2 private DNS names to identify data nodes. Host name to use with datanode is automatically discovered during startup and we always get private DNS name inside EC2 VMs. So the solution is to add following to your hdfs-site.xml:

```xml
<property>
		<name>dfs.client.use.datanode.hostname</name>
		<value>true</value>
</property>
```

and add entry to your ```/etc/hosts``` file to resolve data node's private DNS to the public ip like below.

```
52.26.132.68 ip-172-31-12-45.us-west-2.compute.internal
```

Above will make sure the data nodes on EC2 are accessible from outside. Only drawback is you have to add ```hosts``` file entries for each data node you have.
