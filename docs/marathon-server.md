---
layout: docs
project: depcon
menu: docs
title: Marathon Server Information
first-section: Server Information
---

{:.no-margin-top}
## Server Information

To quickly find out information about the existing deployment:

{:.console}
```
$ depcon server info
```

Output:

{:.console}
```nohighlight
INFO
Name:           marathon
Version:        1.1.1
FrameworkId:    d4eff294-5698-408f-b051-ca4b1a445318-0000
Leader:         192.168.0.200:8080

HTTP CONFIG
HTTP_Port:      8080
HTTPS_Port:     8443

MARATHON CONFIG
Checkpoint:       true
Executor:         //cmd
HA:               true
Master:           zk://10.228.23.16:2181,10.228.23.12:2181/mesos
Failover_Timeout: 604800
LocalPort_Min:    10000
LocalPort_Max:    20000

ZOOKEEPER CONFIG
ZK:             zk://10.228.23.16:2181,10.228.23.12:2181/marathon
ZK_Timeout:     10000
```

## Cluster Leader

To find the host and port of the current cluster leader:

{:.console}
```nohighlight
$ depcon server leader get

Leader:		14.13.12.8:8080
```

To force the current leader to relinquish control (elect a new leader):

{:.console}
```nohighlight
$ depcon server leader abdicate
```

## Ping

To ping the current Marathon host for health check and timing:

{:.console}
```nohighlight
$ depcon server ping

HOST                DURATION
192.253.44.10:8080  23 ms
```
