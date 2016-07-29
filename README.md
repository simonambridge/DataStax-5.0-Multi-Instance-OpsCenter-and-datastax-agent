#DataStax-5.0-Multi-Instance-OpsCenter-and-datastax-agent

The new Multi-Instance feature released with DSE 5.0 allows for the simple deployment of multiple DSE instances on a single machine.

For the first part in this series on Multi-Instance look [here](https://github.com/simonambridge/DataStax-5.0-Multi-Instance-Demo).

DataStax Multi-Instance documentation can be found [here](https://docs.datastax.com/en/latest-dse/datastax_enterprise/multiInstance/configMultiInstance.html)

##Install DataStax OpsCenter

```
deb http://datastaxrepo_gmail.com:utJVKEg4lKeaWTX@debian.datastax.com/enterprise stable main
curl -L https://debian.datastax.com/debian/repo_key | sudo apt-key add -
sudo apt-get update
```

Let's check the nodes we have installed from the previous lesson:
```
sudo dse list-nodes
dse  dse-node1	dse-node2  dse-node3

```
That's:
- dse-node1 is on 127.0.0.2 JMX=7299 cluster
- dse-node2 is on 127.0.0.3 JMX=7399 cluster
- dse-node3 is on 127.0.0.4 JMX=7499 single

This document will describe how to set up datastax-agent for the two noes that form the cluster.

##Install OpsCenter

```
sudo apt-get install opscenter
```
You should get some output like this:
```
....
writing new private key to '/var/lib/opscenter/ssl/opscenter.key'
-----
MAC verified OK
Certificate was added to keystore
Warning: Overwriting existing alias agent_key in destination keystore
[Storing /var/lib/opscenter/ssl/agentKeyStore.p12]
MAC verified OK
```
Now start OpsCenter:
```
sudo service opscenterd start
```
Check its working correctly:
```
sudo tail -100 /var/log/opscenter/startup.log
sudo tail -100 /var/log/opscenter/opscenterd.log
```
Check the web service is working:
```
http://192.168.56.10:8888/opscenter/index.html
```

You'll be prompted to manage an existing cluster:

You can set up nodes at 127.0.0.2/7299 and 127.0.0.3/7399 but we havent installed the agents yet so....

##Tarball agent install - dse-node1

```
sudo mkdir -p /usr/share/datastax-agent/dse-node1
cd /usr/share/datastax-agent/dse-node1

curl --user simon.ambridge@datastax.com:Yzf600rr1 -L http://downloads.datastax.com/enterprise/datastax-agent-6.0.tar.gz | tar xz
```
What have we got?
```
ls /usr/share/datastax-agent/dse-node1
datastax-agent-6.0.1
cd /usr/share/datastax-agent/dse-node1/datastax-agent-6.0.1
```
Edit the address.yaml to set the OpsCenter parameters - the stomp interface points to the node with OpsCenter:
```
vi ./conf/address.yaml

stomp_interface: 127.0.0.1
agent_rpc_interface: 127.0.0.2
jmx_port: 7299
```

Start agent as root:
```
sudo bin/datastax-agent
```
You should get output like this:
```
  INFO [async-dispatch-2] 2016-07-25 15:17:38,104 Starting RollupComponent at 200/second [<127]
  INFO [async-dispatch-2] 2016-07-25 15:17:38,130 Attempting to load stored metric values.
  INFO [async-dispatch-2] 2016-07-25 15:17:38,138 Completed loading 0 stored rollup states
  INFO [async-dispatch-2] 2016-07-25 15:17:38,140 Starting OSStatCollection (Linux)
  INFO [async-dispatch-2] 2016-07-25 15:17:38,152 Starting Performance Service
  INFO [async-dispatch-2] 2016-07-25 15:17:38,165 Starting JMXMetricComponent
  INFO [async-dispatch-2] 2016-07-25 15:17:38,166 Starting Cassandra JMX metric collectors
  INFO [async-dispatch-2] 2016-07-25 15:17:38,180 Finished starting system.
  INFO [qtp1424650534-55] 2016-07-25 15:17:55,489 HTTP: :get /connection-status {} - 200
```  
After a few seconds you should see your agent on the cluster you defined earlier.  
  
```
http://192.168.56.10:8888/opscenter/index.html
```
  
  
##Tarball agent install - dse-node2

Create a directory for the agent
```
sudo mkdir -p /usr/share/datastax-agent/dse-node2
cd /usr/share/datastax-agent/dse-node2
```
Download and unpack the Agent Install File
```
curl --user simon.ambridge@datastax.com:Yzf600rr1 -L http://downloads.datastax.com/enterprise/datastax-agent-6.0.tar.gz | tar xz
```
All there?
```
ls /usr/share/datastax-agent/dse-node2
datastax-agent-6.0.1

cd /usr/share/datastax-agent/dse-node2/datastax-agent-6.0.1
```
Edit the address.yaml file for node2 - the stomp interface points to the node with OpsCenter:
```
vi ./conf/address.yaml

stomp_interface: 127.0.0.1
agent_rpc_interface: 127.0.0.3
jmx_port: 7399
```

Start the second agent as root:
```
sudo bin/datastax-agent
```
Again, look for success:
```
  INFO [async-dispatch-2] 2016-07-28 23:25:15,340 Starting Cassandra JMX metric collectors
  INFO [async-dispatch-2] 2016-07-28 23:25:15,409 Finished starting system.
  INFO [qtp1926072223-57] 2016-07-28 23:25:33,542 HTTP: :get /connection-status {} - 200
  INFO [qtp1926072223-55] 2016-07-28 23:26:33,460 HTTP: :get /connection-status {} - 200  
```

And check OpsCenter:
```
  http://192.168.56.10:8888/opscenter/index.html
```  

Now, while we're here....

##Quick Look At The Cassandra 3.x Storage Engine

###Create The Schema On My Two Node Cluster 
I'm using something based on [Andy Tolbert's](http://www.datastax.com/dev/blog/debugging-sstables-in-3-0-with-sstabledump) example.

Create the keyspace:
```
# cqlsh dse-node1
Connected to Cassandra at dse-node1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.7.1159 | DSE 5.0.1 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> CREATE KEYSPACE IF NOT EXISTS ticker WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 2 };
cqlsh>

USE ticker;
```
Create the table:
```
CREATE TABLE IF NOT EXISTS symbol_history ( 
  symbol    text,
  year      int,
  month     int,
  day       int,
  volume    bigint,
  close     double,
  open      double,
  low       double,
  high      double,
  idx       text static,
  dummy     text static,
  PRIMARY KEY ((symbol, year), month, day)
) with CLUSTERING ORDER BY (month desc, day desc);
``` 
###Insert Some Records
``` 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high, idx, dummy) 
VALUES ('CORP', 2015, 12, 31, 1054342, 9.33, 9.55, 9.21, 9.57, 'NYSE','CORP_2015') USING TTL 604800;
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high, idx, dummy) 
VALUES ('CORP', 2016, 1, 1, 1055334, 8.2, 9.33, 8.02, 9.35, 'NASDAQ', 'CORP_2016') USING TTL 604800;
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high) 
VALUES ('CORP', 2016, 1, 4, 1054342, 8.54, 8.2, 8.2, 8.65) USING TTL 604800;
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high) 
VALUES ('CORP', 2016, 1, 5, 1054772, 8.73, 8.54, 8.44, 8.75) USING TTL 604800;
``` 
###Update a column value
``` 
UPDATE symbol_history USING TTL 604800 set close = 8.55 where symbol = 'CORP' and year = 2016 and month = 1 and day = 4;
```
Next, fro the OS prompt let’s flush memtables to disk as SSTables using nodetool (note the remote JMX port):

nodetool -p 7299 flush
nodetool -p 7299 status

Then back in a cqlsh session we will set a column value to null and delete an entire row to generate some tombstones:

```
cqlsh node2
```
###Set Column Value To Null
``` 
USE ticker;
UPDATE symbol_history SET high = null WHERE symbol = 'CORP' and year = 2016 and month = 1 and day = 1;
``` 

###Delete An Entire Row
 
```
DELETE FROM symbol_history WHERE symbol = 'CORP' and year = 2016 and month = 1 and day = 5;
```
Flush again to generate a new SSTable, and then run a major compaction to create a single SSTable.

nodetool -p 7299 flush
nodetool -p 7299 compact ticker

Now that we have a single SSTable representing operations on our CQL table we can use the appropriate tool to examine its contents.

On-disk representation:
```
sstabledump /var/lib/dse-node1/data/ticker/symbol_history-951ebbb154b211e6980f89474b209cfc/mb-3-big-Data.db -d

[CORP:2016]@0 Row[info=[ts=-9223372036854775808] ]: STATIC | [dummy=CORP_2016 ts=1469703689402299 ttl=604800 ldt=1470308489], [idx=NASDAQ ts=1469703689402299 ttl=604800 ldt=1470308489]
[CORP:2016]@0 Row[info=[ts=-9223372036854775808] del=deletedAt=1469730379207681, localDeletion=1469730379 ]: 1, 5 |
[CORP:2016]@90 Row[info=[ts=1469703698486142 ttl=604800, let=1470308498] ]: 1, 4 | [close=8.55 ts=1469703728290212 ttl=604800 ldt=1470308528], [high=8.65 ts=1469703698486142 ttl=604800 ldt=1470308498], [low=8.2 ts=1469703698486142 ttl=604800 ldt=1470308498], [open=8.2 ts=1469703698486142 ttl=604800 ldt=1470308498], [volume=1054342 ts=1469703698486142 ttl=604800 ldt=1470308498]
[CORP:2016]@167 Row[info=[ts=1469703689402299 ttl=604800, let=1470308489] ]: 1, 1 | [close=8.2 ts=1469703689402299 ttl=604800 ldt=1470308489], [high=<tombstone> ts=1469730366699270 ldt=1469730366], [low=8.02 ts=1469703689402299 ttl=604800 ldt=1470308489], [open=9.33 ts=1469703689402299 ttl=604800 ldt=1470308489], [volume=1055334 ts=1469703689402299 ttl=604800 ldt=1470308489]
[CORP:2015]@233 Row[info=[ts=-9223372036854775808] ]: STATIC | [dummy=CORP_2015 ts=1469703676193714 ttl=604800 ldt=1470308476], [idx=NYSE ts=1469703676193714 ttl=604800 ldt=1470308476]
[CORP:2015]@233 Row[info=[ts=1469703676193714 ttl=604800, let=1470308476] ]: 12, 31 | [close=9.33 ts=1469703676193714 ttl=604800 ldt=1470308476], [high=9.57 ts=1469703676193714 ttl=604800 ldt=1470308476], [low=9.21 ts=1469703676193714 ttl=604800 ldt=1470308476], [open=9.55 ts=1469703676193714 ttl=604800 ldt=1470308476], [volume=1054342 ts=1469703676193714 ttl=604800 ldt=1470308476]
```
Or in JSON format:
```
sstabledump /var/lib/dse-node1/data/ticker/symbol_history-951ebbb154b211e6980f89474b209cfc/mb-3-big-Data.db
[
  {
    "partition" : {
      "key" : [ "CORP", "2016" ],
      "position" : 0
    },
    "rows" : [
      {
        "type" : "static_block",
        "position" : 71,
        "cells" : [
          { "name" : "dummy", "value" : "CORP_2016", "tstamp" : "2016-07-28T11:01:29.402299Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:29Z", "expired" : false },
          { "name" : "idx", "value" : "NASDAQ", "tstamp" : "2016-07-28T11:01:29.402299Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:29Z", "expired" : false }
        ]
      },
      {
        "type" : "row",
        "position" : 71,
        "clustering" : [ "1", "5" ],
        "deletion_info" : { "marked_deleted" : "2016-07-28T18:26:19.207681Z", "local_delete_time" : "2016-07-28T18:26:19Z" },
        "cells" : [ ]
      },
      {
        "type" : "row",
        "position" : 90,
        "clustering" : [ "1", "4" ],
        "liveness_info" : { "tstamp" : "2016-07-28T11:01:38.486142Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:38Z", "expired" : false },
        "cells" : [
          { "name" : "close", "value" : "8.55", "tstamp" : "2016-07-28T11:02:08.290212Z" },
          { "name" : "high", "value" : "8.65" },
          { "name" : "low", "value" : "8.2" },
          { "name" : "open", "value" : "8.2" },
          { "name" : "volume", "value" : "1054342" }
        ]
      },
      {
        "type" : "row",
        "position" : 167,
        "clustering" : [ "1", "1" ],
        "liveness_info" : { "tstamp" : "2016-07-28T11:01:29.402299Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:29Z", "expired" : false },
        "cells" : [
          { "name" : "close", "value" : "8.2" },
          { "name" : "high", "deletion_info" : { "local_delete_time" : "2016-07-28T18:26:06Z" },
            "tstamp" : "2016-07-28T18:26:06.699270Z"
          },
          { "name" : "low", "value" : "8.02" },
          { "name" : "open", "value" : "9.33" },
          { "name" : "volume", "value" : "1055334" }
        ]
      }
    ]
  },
  {
    "partition" : {
      "key" : [ "CORP", "2015" ],
      "position" : 233
    },
    "rows" : [
      {
        "type" : "static_block",
        "position" : 296,
        "cells" : [
          { "name" : "dummy", "value" : "CORP_2015", "tstamp" : "2016-07-28T11:01:16.193714Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:16Z", "expired" : false },
          { "name" : "idx", "value" : "NYSE", "tstamp" : "2016-07-28T11:01:16.193714Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:16Z", "expired" : false }
        ]
      },
      {
        "type" : "row",
        "position" : 296,
        "clustering" : [ "12", "31" ],
        "liveness_info" : { "tstamp" : "2016-07-28T11:01:16.193714Z", "ttl" : 604800, "expires_at" : "2016-08-04T11:01:16Z", "expired" : false },
        "cells" : [
          { "name" : "close", "value" : "9.33" },
          { "name" : "high", "value" : "9.57" },
          { "name" : "low", "value" : "9.21" },
          { "name" : "open", "value" : "9.55" },
          { "name" : "volume", "value" : "1054342" }
        ]
      }
    ]
  }
```
  

