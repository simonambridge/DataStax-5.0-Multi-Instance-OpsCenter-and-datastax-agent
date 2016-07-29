#DataStax-5.0-Multi-Instance-OpsCenter-and-datastax-agent

The new Multi-Instance feature released with DSE 5.0 allows for the simple deployment of multiple DSE instances on a single machine.

For the first part in this series on Multi-Instance look *here*.

DataStax Multi-Instance documentation can be found here: https://docs.datastax.com/en/latest-dse/datastax_enterprise/multiInstance/configMultiInstance.html

##Install DataSTax OpsCenter

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

Install OpsCenter:
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




sudo mkdir -p /usr/share/datastax-agent/dse-node1
cd /usr/share/datastax-agent/dse-node1

curl --user simon.ambridge@datastax.com:Yzf600rr1 -L http://downloads.datastax.com/enterprise/datastax-agent-6.0.tar.gz | tar xz

ls /usr/share/datastax-agent/dse-node1
datastax-agent-6.0.0

cd datastax-agent-6.0.0
vi ./conf/address.yaml

stomp_interface: 127.0.0.1
agent_rpc_interface: 127.0.0.2
jmx_port: 7299

Check any old process hanging around:
sudo netstat -tulpn | grep 61621

Kill it:
ps -eaf | grep datastax-agent
vagrant   2299     1  0 Jul23 pts/1    00:15:22 /usr/lib/jvm/java-8-oracle/bin/java -Xmx128M -Djclouds.mpu.parts.magnitude=100000 -Djclouds.mpu.parts.size=16777216 -Dopscenter.ssl.trustStore=ssl/agentKeyStore -Dopscenter.ssl.keyStore=ssl/agentKeyStore -Dopscenter.ssl.keyStorePassword=opscenter -Dagent-pidfile=./datastax-agent.pid -Dlog4j.configuration=file:./conf/log4j.properties -Djava.security.auth.login.config=./conf/kerberos.config -jar datastax-agent-6.0.0-standalone.jar ./conf/address.yaml
vagrant  16461  9973  0 15:03 pts/1    00:00:00 grep --color=auto datastax-agent

kill -9 2299


Start agent as root:
sudo bin/datastax-agent

  INFO [async-dispatch-2] 2016-07-25 15:17:32,196 Configuration change for component class opsagent.http.server.HTTPServer: before: null, after: {:keystore "ssl/agentKeyStore", :api-port 61621, :opscenter-id nil, :agent-rpc-interface "127.0.0.2", :key-password "*REDACTED*", :truststore "ssl/agentKeyStore", :version "6.0.0", :use-ssl false, :production? true}
  INFO [async-dispatch-2] 2016-07-25 15:17:32,205 Starting Jetty server: {:host "127.0.0.2", :ssl? false, :join? false, :port 61621}
  INFO [Jetty] 2016-07-25 15:17:32,368 Jetty server started
  INFO [async-dispatch-2] 2016-07-25 15:17:32,398 Starting SubscriptionService
  INFO [async-dispatch-2] 2016-07-25 15:17:32,401 Finished starting system.
  INFO [async-dispatch-2] 2016-07-25 15:17:32,403 Starting system.
  INFO [async-dispatch-2] 2016-07-25 15:17:32,410 Configuration change for component class opsagent.messaging.StompComponent: before: {:local-interface nil}, after: {:local-interface "127.0.0.2"}
  INFO [async-dispatch-2] 2016-07-25 15:17:32,420 Configuration change for component class opsagent.rollup.RollupComponent: before: {:rollup-id nil}, after: {:rollup-id 127.0.0.2}
  INFO [async-dispatch-2] 2016-07-25 15:17:32,434 Configuration change for component class opsagent.os.collection.OSStatCollection: before: {:local-interface nil}, after: {:local-interface 127.0.0.2}
  INFO [async-dispatch-2] 2016-07-25 15:17:32,442 The following components have had a config change and will be rebuilt and restarted:  (:stomp-component :rollup-component :os-stat-component)
  INFO [async-dispatch-2] 2016-07-25 15:17:32,444 The component restart for  (:stomp-component :rollup-component :os-stat-component)  when accounting for dependencies requires these components to be restarted  #{:jmx-metric-component :status-component :disk-usage-collection :request-component :stomp-component :rollup-component :http-server :config-request :nodedetails-component :os-stat-component :staging-component}
  INFO [async-dispatch-2] 2016-07-25 15:17:32,450 Starting StompComponent
  INFO [async-dispatch-2] 2016-07-25 15:17:32,451 SSL communication is disabled
  INFO [async-dispatch-2] 2016-07-25 15:17:32,451 Creating stomp connection to 192.168.56.10:61620
 ERROR [async-dispatch-2] 2016-07-25 15:17:32,455 Jul 25, 2016 3:17:32 PM org.jgroups.client.StompConnection connect
INFO: Connected to 192.168.56.10:61620

  INFO [async-dispatch-2] 2016-07-25 15:17:32,511 Starting DynamicEnvironmentComponent
  INFO [StompConnection receiver] 2016-07-25 15:17:32,577 Got new config [note values in address.yaml override those from OpsCenter]: {:rollup_subscriptions [], :destinations [], :metrics_ignored_column_families "", :metrics_enabled true, :cassandra_log_location "/var/log/cassandra/system.log", :hosts ["127.0.0.2"], :jmx_user "*REDACTED*", :kerberos_service "", :config_md5 "33301319c8374afc22e0c9ed76ea86ea", :monitored_cassandra_pass "*REDACTED*", :monitored_thrift_port 9160, :cassandra_rpc_interface "127.0.0.2", :metrics_ignored_solr_cores "", :rollups60_ttl 604800, :max_pending_repairs 5, :api_port "61621", :cassandra_port 9042, :jmx_pass "*REDACTED*", :kerberos_keytab_location "", :rollups300_ttl 2419200, :cassandra_pass "*REDACTED*", :metrics_ignored_keyspaces "system, system_traces, system_auth, system_distributed, dse_auth, OpsCenter", :use_ssl false, :cassandra_user "*REDACTED*", :monitored_cassandra_user "*REDACTED*", :thrift_port 9160, :storage_keyspace "OpsCenter", :monitored_cassandra_port 9042, :kerberos_client_principal "", :cassandra_install_location "", :restore_req_update_period 1, :jmx_port 7199, :ec2_metadata_api_host "169.254.169.254", :rollups86400_ttl 0, :jmx_operations_pool_size 4, :backup_staging_dir "", :rollups7200_ttl 31536000}
  INFO [StompConnection receiver] 2016-07-25 15:17:32,589 Configuration change for component class opsagent.cassandra.StorageDatabase: before: {:storage-hosts ["127.0.0.1"]}, after: {:storage-hosts ["127.0.0.2"]}
  INFO [async-dispatch-2] 2016-07-25 15:17:32,773 Dynamic environment script output:  paths:
  cassandra-conf: /etc/dse-node1/cassandra
  cassandra-log: /var/log/dse-node1
  hadoop-log: /var/log/dse-node1/hadoop/userlogs
  spark-log: /var/log/dse-node1/spark/worker
  dse-env: /etc/dse-node1
  dse-conf: /etc/dse-node1
  hadoop-conf: /etc/dse-node1/hadoop2-client
  spark-conf: /etc/dse-node1/spark
  INFO [async-dispatch-2] 2016-07-25 15:17:32,803 Starting storage database connection.
 ERROR [async-dispatch-2] 2016-07-25 15:17:35,029 Can't connect to Cassandra (All host(s) tried for query failed (tried: /127.0.0.1:9042 (com.datastax.driver.core.exceptions.TransportException: [/127.0.0.1] Cannot connect))), retrying soon.
  INFO [async-dispatch-2] 2016-07-25 15:17:35,030 Starting NodeDetailsComponent
  INFO [async-dispatch-2] 2016-07-25 15:17:35,239 Starting RequestComponent.
  INFO [async-dispatch-2] 2016-07-25 15:17:35,240 Starting DiskUsageCollection
  INFO [async-dispatch-2] 2016-07-25 15:17:35,255 Finished starting system.
  INFO [async-dispatch-2] 2016-07-25 15:17:35,255 Starting system.
  INFO [async-dispatch-2] 2016-07-25 15:17:35,262 Configuration change for component class opsagent.cassandra.StorageDatabase: before: {:storage-hosts ["127.0.0.1"]}, after: {:storage-hosts ["127.0.0.2"]}
  INFO [async-dispatch-2] 2016-07-25 15:17:35,263 Configuration change for component class opsagent.cassandra.MonitoredDatabase: before: {:monitored-hosts [127.0.0.1]}, after: {:monitored-hosts [127.0.0.2]}
  INFO [async-dispatch-2] 2016-07-25 15:17:35,269 Configuration change for component class opsagent.metrics.jmx.JMXMetricComponent: before: {:ignored-keyspaces nil}, after: {:ignored-keyspaces [system system_traces system_auth system_distributed dse_auth OpsCenter]}
  INFO [async-dispatch-2] 2016-07-25 15:17:35,271 Configuration change for component class opsagent.backups.destinations.DestinationService: before: {:destinations nil}, after: {:destinations []}
  INFO [async-dispatch-2] 2016-07-25 15:17:35,271 The following components have had a config change and will be rebuilt and restarted:  (:storage-database :monitored-database :jmx-metric-component :destination-service)
  INFO [async-dispatch-2] 2016-07-25 15:17:35,274 The component restart for  (:storage-database :monitored-database :jmx-metric-component :destination-service)  when accounting for dependencies requires these components to be restarted  #{:monitored-database :jmx-metric-component :status-component :metrics-config :storage-database :destination-service :rollup-component :http-server :config-request :performance-service :os-stat-component :staging-component}
  INFO [async-dispatch-2] 2016-07-25 15:17:35,277 Stopping Performance Service.
  INFO [async-dispatch-2] 2016-07-25 15:17:35,278 Shutting down Performance Service threadpool.
  INFO [async-dispatch-2] 2016-07-25 15:17:35,307 Stopping monitored database connection.
  INFO [async-dispatch-2] 2016-07-25 15:17:37,570 Starting DynamicEnvironmentComponent
  INFO [async-dispatch-2] 2016-07-25 15:17:37,772 Dynamic environment script output:  paths:
  cassandra-conf: /etc/dse-node1/cassandra
  cassandra-log: /var/log/dse-node1
  hadoop-log: /var/log/dse-node1/hadoop/userlogs
  spark-log: /var/log/dse-node1/spark/worker
  dse-env: /etc/dse-node1
  dse-conf: /etc/dse-node1
  hadoop-conf: /etc/dse-node1/hadoop2-client
  spark-conf: /etc/dse-node1/spark
  INFO [async-dispatch-2] 2016-07-25 15:17:37,791 Starting storage database connection.
  INFO [async-dispatch-2] 2016-07-25 15:17:37,882 The storage connection state is closed?  false
  INFO [async-dispatch-2] 2016-07-25 15:17:37,883 Starting FileTransferStagingComponent
  INFO [async-dispatch-2] 2016-07-25 15:17:37,886 Starting monitored database connection.
  INFO [async-dispatch-2] 2016-07-25 15:17:37,983 The monitored connection state is closed?   false
  INFO [async-dispatch-2] 2016-07-25 15:17:37,985 Building Metrics Definition
  INFO [async-dispatch-2] 2016-07-25 15:17:38,018 Using version 3.0.0 from ["2.0.11.83" "2.0.12.200" "2.1.5.469" "2.1.8.621" "3.0.0" "uploaded"] [3.0.7.1159]
  INFO [async-dispatch-2] 2016-07-25 15:17:38,019 Downloading definition...  ./conf/cassandra-metrics.json
  INFO [async-dispatch-2] 2016-07-25 15:17:38,033 Download complete
  INFO [async-dispatch-2] 2016-07-25 15:17:38,104 Starting RollupComponent at 200/second [<127]
  INFO [async-dispatch-2] 2016-07-25 15:17:38,130 Attempting to load stored metric values.
  INFO [async-dispatch-2] 2016-07-25 15:17:38,138 Completed loading 0 stored rollup states
  INFO [async-dispatch-2] 2016-07-25 15:17:38,140 Starting OSStatCollection (Linux)
  INFO [async-dispatch-2] 2016-07-25 15:17:38,152 Starting Performance Service
  INFO [async-dispatch-2] 2016-07-25 15:17:38,165 Starting JMXMetricComponent
  INFO [async-dispatch-2] 2016-07-25 15:17:38,166 Starting Cassandra JMX metric collectors
  INFO [async-dispatch-2] 2016-07-25 15:17:38,180 Finished starting system.
  INFO [qtp1424650534-55] 2016-07-25 15:17:55,489 HTTP: :get /connection-status {} - 200
  
  
  
  http://192.168.56.10:8888/opscenter/index.html
  
  
  
Tarball agent install - dse-node2
---------------------------------

sudo mkdir -p /usr/share/datastax-agent/dse-node2
cd /usr/share/datastax-agent/dse-node2

curl --user simon.ambridge@datastax.com:Yzf600rr1 -L http://downloads.datastax.com/enterprise/datastax-agent-6.0.tar.gz | tar xz

ls /usr/share/datastax-agent/dse-node2
datastax-agent-6.0.0

cd /usr/share/datastax-agent/dse-node2/datastax-agent-6.0.1
vi ./conf/address.yaml

stomp_interface: 127.0.0.1
agent_rpc_interface: 127.0.0.3
jmx_port: 7399

Check any old process hanging around:
sudo netstat -tulpn | gr

Kill it:
ps -eaf | grep datastax-agent
vagrant   2299     1  0 Jul23 pts/1    00:15:22 /usr/lib/jvm/java-8-oracle/bin/java -Xmx128M -Djclouds.mpu.parts.magnitude=100000 -Djclouds.mpu.parts.size=16777216 -Dopscenter.ssl.trustStore=ssl/agentKeyStore -Dopscenter.ssl.keyStore=ssl/agentKeyStore -Dopscenter.ssl.keyStorePassword=opscenter -Dagent-pidfile=./datastax-agent.pid -Dlog4j.configuration=file:./conf/log4j.properties -Djava.security.auth.login.config=./conf/kerberos.config -jar datastax-agent-6.0.0-standalone.jar ./conf/address.yaml
vagrant  16461  9973  0 15:03 pts/1    00:00:00 grep --color=auto datastax-agent

kill -9 2299


Start agent as root:
sudo bin/datastax-agent

  INFO [async-dispatch-2] 2016-07-28 23:25:15,340 Starting Cassandra JMX metric collectors
  INFO [async-dispatch-2] 2016-07-28 23:25:15,409 Finished starting system.
  INFO [qtp1926072223-57] 2016-07-28 23:25:33,542 HTTP: :get /connection-status {} - 200
  INFO [qtp1926072223-55] 2016-07-28 23:26:33,460 HTTP: :get /connection-status {} - 200  
  
  http://192.168.56.10:8888/opscenter/index.html
  
  
  -- Create the schema using the this example on my two node cluster
 
CREATE KEYSPACE IF NOT EXISTS ticker WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 2 };
 
# cqlsh dse-node1
Connected to Cassandra at dse-node1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.7.1159 | DSE 5.0.1 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> CREATE KEYSPACE IF NOT EXISTS ticker WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 2 };
cqlsh>


USE ticker;
 
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
 
-- Insert some records
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high, idx, dummy) 
VALUES ('CORP', 2015, 12, 31, 1054342, 9.33, 9.55, 9.21, 9.57, 'NYSE','CORP_2015') USING TTL 604800;
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high, idx, dummy) 
VALUES ('CORP', 2016, 1, 1, 1055334, 8.2, 9.33, 8.02, 9.35, 'NASDAQ', 'CORP_2016') USING TTL 604800;
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high) 
VALUES ('CORP', 2016, 1, 4, 1054342, 8.54, 8.2, 8.2, 8.65) USING TTL 604800;
 
INSERT INTO symbol_history (symbol, year, month, day, volume, close, open, low, high) 
VALUES ('CORP', 2016, 1, 5, 1054772, 8.73, 8.54, 8.44, 8.75) USING TTL 604800;
 
-- Update a column value
 
UPDATE symbol_history USING TTL 604800 set close = 8.55 where symbol = 'CORP' and year = 2016 and month = 1 and day = 4;

Next, let’s flush memtables to disk as SSTables using nodetool:

nodetool -p 7299 flush
nodetool -p 7299 status



Then in a cqlsh session we will set a column value to null and delete an entire row to generate some tombstones:

cqlsh node2

-- Set column value to null
 
USE ticker;
UPDATE symbol_history SET high = null WHERE symbol = 'CORP' and year = 2016 and month = 1 and day = 1;
 
-- Delete an entire row
 
DELETE FROM symbol_history WHERE symbol = 'CORP' and year = 2016 and month = 1 and day = 5;
We proceed to flush again to generate a new SSTable, and then perform a major compaction yielding a single SSTable.

nodetool -p 7299 flush
nodetool -p 7299 compact ticker

Now that we have a single SSTable representing operations on our CQL table we can use the appropriate tool to examine its contents.

sstabledump /var/lib/dse-node1/data/ticker/symbol_history-951ebbb154b211e6980f89474b209cfc/mb-3-big-Data.db -d
[CORP:2016]@0 Row[info=[ts=-9223372036854775808] ]: STATIC | [dummy=CORP_2016 ts=1469703689402299 ttl=604800 ldt=1470308489], [idx=NASDAQ ts=1469703689402299 ttl=604800 ldt=1470308489]
[CORP:2016]@0 Row[info=[ts=-9223372036854775808] del=deletedAt=1469730379207681, localDeletion=1469730379 ]: 1, 5 |
[CORP:2016]@90 Row[info=[ts=1469703698486142 ttl=604800, let=1470308498] ]: 1, 4 | [close=8.55 ts=1469703728290212 ttl=604800 ldt=1470308528], [high=8.65 ts=1469703698486142 ttl=604800 ldt=1470308498], [low=8.2 ts=1469703698486142 ttl=604800 ldt=1470308498], [open=8.2 ts=1469703698486142 ttl=604800 ldt=1470308498], [volume=1054342 ts=1469703698486142 ttl=604800 ldt=1470308498]
[CORP:2016]@167 Row[info=[ts=1469703689402299 ttl=604800, let=1470308489] ]: 1, 1 | [close=8.2 ts=1469703689402299 ttl=604800 ldt=1470308489], [high=<tombstone> ts=1469730366699270 ldt=1469730366], [low=8.02 ts=1469703689402299 ttl=604800 ldt=1470308489], [open=9.33 ts=1469703689402299 ttl=604800 ldt=1470308489], [volume=1055334 ts=1469703689402299 ttl=604800 ldt=1470308489]
[CORP:2015]@233 Row[info=[ts=-9223372036854775808] ]: STATIC | [dummy=CORP_2015 ts=1469703676193714 ttl=604800 ldt=1470308476], [idx=NYSE ts=1469703676193714 ttl=604800 ldt=1470308476]
[CORP:2015]@233 Row[info=[ts=1469703676193714 ttl=604800, let=1470308476] ]: 12, 31 | [close=9.33 ts=1469703676193714 ttl=604800 ldt=1470308476], [high=9.57 ts=1469703676193714 ttl=604800 ldt=1470308476], [low=9.21 ts=1469703676193714 ttl=604800 ldt=1470308476], [open=9.55 ts=1469703676193714 ttl=604800 ldt=1470308476], [volume=1054342 ts=1469703676193714 ttl=604800 ldt=1470308476]

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
  
  

