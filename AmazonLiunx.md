 # What is Kafka?
Kafka is a distributed messaging system that provides fast, highly scalable, and redundant messaging through a public subscribe distributed messaging system. Kafka is written in Scala. Kafka was developed by LinkedIn and open-sourced in 2011.
Kafka is a “public subscribe distributed messaging system” rather than a “queue system” since the message is received from the producer and broadcasted to a group of consumers rather than a single consumer.
A remarkable feature of Kafka is that it is highly available, immune from node failures, and has automatic recovery. All the above makes Apache Kafka an ideal tool for communication and integration between different parts of a large-scale data system in real-world data systems.
Kafka was meant to operate in a cluster, and we should be creating a cluster if you’re using Kafka.

# Topic:
Messages are published to a ‘Topic’ and there is a partition associated with each ‘Topic’. Kafka stores and organizes messages across its system and essentially a collection of messages are called Topics.
# Brokers:
Broker in Kafka holds the messages that have been written by the producer before being consumed by the ‘consumer’.
Kafka cluster contains multiple brokers. A broker has a partition and as already communicated each partition is associated with a topic. The brokers receive the messages and they are stored in the “brokers” for ’n’ number of days (which can be configured). After the ’n’ of days has expired, the messages are discarded. It is important to state here again that Kafka does not check whether each consumer or consumer group has read the messages.
# Producer:
Different producers like Apps, DBMS, NoSQL write data to the Kafka cluster and publishes messages to a Kafka topic.
# Consumer:
After the “producers” have produced the message and sent it to the Kafka brokers, the consumers then read the message. A consumer or consumer group is/are subscribed to different topics and in turn, they read from the partition for the topics to which they are subscribed. If a broker goes down, then other brokers support the system and make sure everything is running smoothly.
# ZooKeeper:
The Zookeeper’s primary responsibility is to coordinate with the different components of the Kafka cluster. The producer has the job to give the message to the broker leader/active controller which in turn writes the message onto itself and also takes care of replicating it to other brokers.
Zookeeper helps maintain consensus in the cluster, which basically means all the brokers are aware of each other and know which one is the controller.

- Install open JDK
- Disable Swap

```
yum install java -y
```

## Now, lets set up the Kafka Cluster:
##### Download the Kafka Packages

```
sudo -i
wget https://downloads.apache.org/kafka/3.2.3/kafka_2.13-3.2.3.tgz
tar -xvf kafka_2.13-3.2.3.tgz
mv kafka_2.13-3.2.3 /opt/kafka
```
##### Create a data directory to store Kafka messages and Zookeeper data.

Step- 1. Create a New Directory for Kafka and Zookeeper
```
sudo mkdir -p /data/kafka
sudo mkdir -p /data/zookeeper
```
B. Create a Zookeeper ID on each VM.
```
Create a file in /zookeeper first VM named "myid" with the ID as  "1":
echo "1" > /data/zookeeper/myid
Create a file in /zookeeper Second VM named "myid" with the ID as "2":
echo "2" > /data/zookeeper/myid
Create a file in /zookeeper Third VM named "myid" with the ID as  "3":
echo "3" > /data/zookeeper/myid
```
D. Modify the Kafka and Zookeeper Configuration Files on all VM’s
Step- 1. Open the server.properties file:
```
cd /opt/kafka
> config/server.properties
vim config/server.properties
```
Step- 2. Update broker.id and advertised.listners into server.properties configuration as shown below:
Note: Add the below configuration on all VM’s. Run each command in the parallel console.
```
broker.id=1
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://10.10.10.10:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/data/kafka/data
num.partitions=10
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=1
default.replication.factor=3
min.insync.replicas=1
log.retention.hours=168
log.segment.bytes=104857600
log.retention.check.interval.ms=2000
log.roll.hours=168
log.retention.check.interval.hours=1
log.cleaner.enable=true
delete.topic.enable=false
auto.create.topics.enable=false
compression.type=gzip
zookeeper.connection.timeout.ms=10000
zookeeper.connect=10.10.10.11:2181,10.10.10.11:2181,10.10.10.12:2181
```
Step- 3. open the zookeeper.properties file on all VM’s.
```
vi /opt/zookeer/config/zookeeper.properties
```
Step- 4. Paste the following configuration into the zookeeper.properties :
```
tickTime=2000
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
maxClientCnxns=60
clientPort=2181
initLimit=10
syncLimit=5
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
metricsProvider.httpPort=7000
metricsProvider.exportJvmInfo=true
server.1=10.10.10.10:2888:3888
server.2=10.10.10.11:2888:3888
server.3=10.10.10.12:2888:3888
4lw.commands.whitelist=*
```
E. Create the init.d scripts to start and stop for Kafka and Zookeeper service
Kafka:
Step- 1. Open file /etc/init.d/kafka on each VM on virtual box and paste in the following:

- sudo vim /etc/systemd/system/kafka.service
```
[Unit]
Description=Apache Kafka server (broker)

[Service]
Type=simple
User=kafka
LimitNOFILE=150000
LimitNPROC=100000
ExecStart=/bin/sh -c '/opt/kafka_2.13-3.2.3/bin/kafka-server-start.sh /opt/kafka_2.13-3.2.3/config/server.properties > /data/kafka/logs/kafka-boot.log 2>&1'
ExecStop=/opt/kafka_2.13-3.2.3/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
- Save and close the file.
Step- 2. Make the file /etc/init.d/kafka executable. Also, change the ownership and start the service:
```
sudo systemctl start kafka
sudo systemctl enable kafka
sudo systemctl status kafka
```

## Zookeeper :
Step- 1. Open file /etc/systemd/system/zookeeper.service on all VM’s on virtual box and paste in the following:
- sudo vim /etc/systemd/system/zookeeper.service

```
[Unit]
Description=zookeeper

[Service]
Type=forking
User=zookeeper
ExecStart=/opt/zookeeper-3.6.3/bin/zkServer.sh start
ExecStop=/opt/zookeeper-3.6.3/bin/zkServer.sh stop

Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Save and close the file.

Step- 2. Make the file /etc/init.d/zookeeper executable. Also, change the ownership and start the service:
```
sudo chmod +x /etc/systemd/system/zookeeper.service
sudo chown root:root /etc/systemd/system/zookeeper.service
sudo systemctl start zookeeper 
sudo systemctl enable zookeeper 
```
F. Create a Topic
Create a topic named test:
```
./bin/kafka-topics.sh --zookeeper zookeeper1:2181/kafka --create --topic test --replication-factor 1 --partitions 3
```
Describe the topic:
```
./bin/kafka-topics.sh --zookeeper zookeeper1:2181/kafka --topic test --describe
```
