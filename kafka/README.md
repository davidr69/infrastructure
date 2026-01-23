# kafka

## server.properties

```text
process.roles=broker,controller
node.id=1
controller.quorum.bootstrap.servers=localhost:9093
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
inter.broker.listener.name=PLAINTEXT
advertised.listeners=PLAINTEXT://kafka.lavacro.net:9092,CONTROLLER://kafka.lavacro.net:9093
controller.listener.names=CONTROLLER
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/home/david/kafka_2.13-4.1.1/storage/kraft-combined-logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
share.coordinator.state.topic.replication.factor=1
share.coordinator.state.topic.min.isr=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
```

## connect-standalone.properties

```text
bootstrap.servers=localhost:9092

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter

key.converter.schemas.enable=false
value.converter.schemas.enable=false
# key.converter.schemas.enable=true
# value.converter.schemas.enable=true

offset.storage.file.filename=/home/david/kafka_2.13-4.1.1/connect/connect.offsets
# Flush much faster than normal, which is useful for testing/debugging
offset.flush.interval.ms=5000

plugin.path=/home/david/kafka_2.13-4.1.1/plugins
```

## traffic-mon-connector.properties

```text
name=roundrobin-fanout
connector.class=org.apache.kafka.connect.mirror.MirrorSourceConnector
tasks.max=4
topics=traffic-topic
source.cluster.alias=main
target.cluster.alias=fanout

transforms=RoundRobin
transforms.RoundRobin.type=net.lavacro.kafka.connect.RoundRobinTopic
transforms.RoundRobin.target.topic.base=traffic-topic
transforms.RoundRobin.num.topics=4

replication.factor=1

source.cluster.bootstrap.servers=localhost:9092
target.cluster.bootstrap.servers=localhost:9092

#consumer.auto.offset.reset=latest
consumer.auto.offset.reset=earliest

key.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false

value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=false

# producer tuning
producer.batch.size=524288
producer.linger.ms=20
producer.compression.type=lz4
producer.acks=1

# consumer tuning
#consumer.fetch.min.bytes=1048576
#consumer.fetch.max.wait.ms=100
#consumer.max.poll.records=1000

# Convert value from JSON string to structured data
#value.converter=org.apache.kafka.connect.json.JsonConverter
#value.converter.schemas.enable=false

# Transform pipeline
#transforms=extractHost,route
#transforms.extractHost.type=org.apache.kafka.connect.transforms.ExtractField$Value
#transforms.extractHost.field=hostname

# Use the extracted field to name subtopics
#transforms.route.type=org.apache.kafka.connect.transforms.RegexRouter
#transforms.route.regex=.*
#transforms.route.replacement=syslog-watcher-${hostname}
```
