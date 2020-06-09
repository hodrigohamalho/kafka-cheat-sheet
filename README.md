# Kafka Cheat Sheet

First, set your kafka home environment.

```bash
KAFKA_BIN=/opt/kafka/bin
ZOOKEEPER_HOST=ip-zookeeper
BROKER_HOST=ip-broker
```

## Broker Operations

### List active brokers

```bash
$KAFKA_BIN/zookeeper-shell.sh $ZOOKEEPER_HOST:2181 ls /brokers/ids
```

### List broker details

```bash
$KAFKA_BIN/zookeeper-shell.sh $ZOOKEEPER_HOST:2181 ls /brokers/ids/{id}
```

### List topics

```bash
$KAFKA_BIN/zookeeper-shell.sh $ZOOKEEPER_HOST:2181 ls /brokers/topics
```

## Topic Operations

### List topics

```bash
$KAFKA_BIN/kafka-topics.sh \
    --list \
    --zookeeper $ZOOKEEPER_HOST:2181
```

### Describe topic

```bash
$KAFKA_BIN/kafka-topics.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --topic <topic_name> \
    --describe
```

### Create topic

```bash
$KAFKA_BIN/kafka-topics.sh \
    --create \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --replication-factor 1 \
    --partitions 1 \
    --topic <topic_name>
```

### Alter topic

#### Alter retention time

```bash
$KAFKA_BIN/kafka-topics.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --alter \
    --topic <topic_name>\
    --config retention.ms=1000
```

#### Alter min.insync.replicas

```bash
$KAFKA_BIN/kafka-topics.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --alter \
    --topic <topic_name> \
    --config min.insync.replicas=2
```

#### Delete retention time

```bash
$KAFKA_BIN/kafka-topics.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --alter \
    --topic <topic_name> \
    --delete-config retention.ms
```

### List topics under-replicated

```bash
$KAFKA_BIN/kafka-topics.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --describe \
    --under-replicated-partitions
```

### Delete topic

```bash
$KAFKA_BIN/kafka-topics.sh \
    --delete \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --topic <topic_name>
```

### Get earliest offset

```bash
$KAFKA_BIN/kafka-run-class.sh \
    kafka.tools.GetOffsetShell \
    --broker-list $BROKER_HOST:9092 \
    --topic <topic_name> \
    --time -2
```

### Get latest offset

```bash
$KAFKA_BIN/kafka-run-class.sh \
    kafka.tools.GetOffsetShell \
    --broker-list $BROKER_HOST:9092 \
    --topic <topic_name> \
    --time -1
```

## Partition Operations

### Add partitions

```bash
$KAFKA_BIN/kafka-topics.sh \
    --alter \
    --topic <topic_name> \
    --partitions 8
```

### Reassign partitions

```bash
$KAFKA_BIN/kafka-reassign-partitions.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --reassignment-json-file increase-replication-factor.json  \
    --execute

$KAFKA_BIN/kafka-reassign-partitions.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --reassignment-json-file increase-replication-factor.json  \
    --verify
```

### List unavailable partitions

```bash
$KAFKA_BIN/kafka-topics.sh \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --describe \
    --unavailable-partitions
```

## Consumer

### List consumer groups

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --list \
    --bootstrap-server $BROKER_HOST:9092
```

### Describe consumer groups

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --describe \
    --group <group_id> \
    --bootstrap-server $BROKER_HOST:9092
```

### Consuming message from the beginning

```bash
$KAFKA_BIN/kafka-console-consumer.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --topic <topic_name> \
    --from-beginning
```

### Consuming message from the end

```bash
$KAFKA_BIN/kafka-console-consumer.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --topic <topic_name>
```

### Read one message

```bash
$KAFKA_BIN/kafka-console-consumer.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --topic <topic_name> \
    --max-messages 1
```

### Read from __consumer_offsets

```bash
$KAFKA_BIN/kafka-console-consumer.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --topic __consumer_offsets \
    --formatter 'kafka.coordinator.group.GroupMetadataManager$OffsetsMessageFormatter' \
    --max-messages 1
```

### Consume using consumer group

```bash
$KAFKA_BIN/kafka-console-consumer.sh \
    --topic <topic_name> \
    --bootstrap-server $BROKER_HOST:9092 \
    --group <group-id>
```

### Topics to which group is subscribed

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --group <group_id> \
    --describe
```

### Reset offset

#### Reset offset for a consumer group in a topic

```bash
# There are many other resetting options
# --shift-by <positive_or_negative_integer> / --to-current / --to-latest / --to-offset <offset_integer>
# --to-datetime <datetime_string> --by-duration <duration_string>
$KAFKA_BIN/kafka-consumer-groups.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --group <group_id> \
    --topic <topic_name> \
    --reset-offsets \
    --to-earliest \
    --execute
```

#### Reset offset from all consumer groups

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --all-groups \
    --reset-offsets \
    --topic <topic_name> \
    --to-earliest
```

#### Forward by 2 for example

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --group <groud_id> \
    --reset-offsets \
    --shift-by 2 \
    --execute \
    --topic <topic_name>
```

#### Backward by 2 for example

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --group <groud_id> \
    --reset-offsets \
    --shift-by -2 \
    --execute \
    --topic <topic_name>
```

### Describe consumer group

```bash
$KAFKA_BIN/kafka-consumer-groups.sh \
    --bootstrap-server $BROKER_HOST:9092 \
    --describe \
    --group <group_id>
```

### Check offset for consumer group

```bash
$KAFKA_BIN/kafka-consumer-offset-checker.sh  \
    --zookeeper $ZOOKEEPER_HOST:2181 \
    --group <group_id> \
    --topic <topic_name>
```

## Producer

### Send message using file

```bash
$KAFKA_BIN/kafka-console-producer.sh \
    --broker-list $BROKER_HOST:9092 \
    --topic <topic_name> < messages.txt
```

### Send message using standard input

```bash
$KAFKA_BIN/kafka-console-producer \
    --broker-list $BROKER_HOST:9092 \
    --topic <topic_name>
```

### Send message using string

```bash
echo "My Message" | $KAFKA_BIN/kafka-console-producer.sh \
    --broker-list $BROKER_HOST:9092 \
    --topic <topic_name>
```

### Send message using ack=all

```bash
$KAFKA_BIN/kafka-console-producer.sh \
    --broker-list $BROKER_HOST:9092 \
    --topic <topic_name> \
    --producer-property acks=all
```

## ACLs

```bash
$KAFKA_BIN/kafka-acls.sh \
    --authorizer-properties zookeeper.connect=$ZOOKEEPER_HOST:2181 \
    --add \
    --allow-principal User:Gus \
    --consumer \
    --topic <topic_name> \
    --group <group_id>
```

```bash
$KAFKA_BIN/kafka-acls.sh
    --authorizer-properties zookeeper.connect=$ZOOKEEPER_HOST:2181 \
    --add \
    --allow-principal User:Gus \
    --producer \
    --topic <topic_name>
```

### List topics ACLs

```bash
$KAFKA_BIN/kafka-acls.sh \
    --authorizer-properties zookeeper.connect=$ZOOKEEPER_HOST:2181 \
    --list \
    --topic <topic_name>
```
