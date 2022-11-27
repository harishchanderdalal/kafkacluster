# Kafka Single Service Quick Setup
## Install Java
```
yum install java -y
```

## Now, lets set up the Kafka Cluster:
##### Download the Kafka Packages

```
wget https://downloads.apache.org/kafka/3.2.3/kafka_2.13-3.2.3.tgz
tar -xvf kafka_2.13-3.2.3.tgz
sudo mv kafka_2.13-3.2.3 /tmp/kafka
cd /tmp/kafka
```

### Start the ZooKeeper service
```
# Start the ZooKeeper service
$ /tmp/kafka_2.13-3.2.3/bin/zookeeper-server-start.sh /tmp/kafka_2.13-3.2.3/config/zookeeper.properties
```
Open New Terminal and Login
### Start the Kafka broker service
```
$ /tmp/kafka_2.13-3.2.3/bin/kafka-server-start.sh /tmp/kafka_2.13-3.2.3/config/server.properties
```

Open New Terminal and Login
### How to create TOPIC
```
$ /tmp/kafka_2.13-3.2.3/bin/kafka-topics.sh --create --topic harish --bootstrap-server localhost:9092
```

### Topic Produce
```
$ /tmp/kafka_2.13-3.2.3/bin/kafka-console-producer.sh --topic harish --bootstrap-server localhost:9092
BMW
Audi
```

### Consume 
```
$ /tmp/kafka_2.13-3.2.3/bin/kafka-console-consumer.sh --topic harish --from-beginning --bootstrap-server localhost:9092
BMW
Audi
```
