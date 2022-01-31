# Introduction to Kafka

Official [Docs](https://kafka.apache.org/)

## Building a Docker file

As always, we start with a `dockerfile` </br>
We can build our `dockerfile`

```
docker build . -t pradipta/kafka:latest

```

## Exploring the Kafka Install

We can then run it to explore the contents:

```
docker run --rm --name kafka -it pradipta/kafka:latest bash

ls -l /opt/kafka/bin/
cat /opt/kafka/config/server.properties
```

Note: We'll need the Kafka configuration to tune our server and Kafka also requires
at least one Zookeeper instance in order to function. To achieve high availability, we should run
multiple kafka as well as multiple zookeeper instances in the future

# Zookeeper

Let's build a Zookeeper image. The Apache folks have made it easy to start a Zookeeper instance the same way as the Kafka instance by simply running the `start-zookeeper.sh` script.

```
cd ./zookeeper
docker build . -t pradipta/zookeeper:latest
cd ..
```

Let's create a kafka network and run 1 zookeeper instance

```
docker network create kafka
docker run -d --rm --name zookeeper-1 --net kafka -v ${PWD}/config/zookeeper-1/zookeeper.properties:/opt/kafka/config/zookeeper.properties pradipta/zookeeper:latest

docker logs zookeeper-1
```

# Kafka - 1

```
docker run -d --rm --name kafka-1 --net kafka -v ${PWD}/config/kafka-1/server.properties:/opt/kafka/config/server.properties pradipta/kafka:latest

docker logs kafka-1
```

# Kafka - 2

```
docker run -d --rm --name kafka-2 --net kafka -v ${PWD}/config/kafka-2/server.properties:/opt/kafka/config/server.properties pradipta/kafka:latest

docker logs kafka-2
```

# Kafka - 3

```
docker run -d --rm --name kafka-3 --net kafka -v ${PWD}/config/kafka-3/server.properties:/opt/kafka/config/server.properties pradipta/kafka:latest

docker logs kafka-3
```


# Topic

Let's create a Topic that allows us to store `Test` information. </br>
To create a topic, Kafka and Zookeeper have scripts with the installer that allows us to do so. </br>

Access the container:
```
docker exec -it zookeeper-1 bash
```
Create the Topic:
```
/opt/kafka/bin/kafka-topics.sh \
--create \
--bootstrap-server kafka-1:9092 \
--replication-factor 1 \
--partitions 3 \
--topic Test
```

Describe our Topic:
```
/opt/kafka/bin/kafka-topics.sh \
--describe \
--topic Test \
--bootstrap-server kafka-1:9092
```

# Simple Producer & Consumer

The Kafka installation also ships with a script that allows us to produce
and consume messages to our Kafka network: <br/>

We can then run the consumer that will receive that message on that Orders topic:

```
docker exec -it zookeeper-1 bash

/opt/kafka/bin/kafka-console-consumer.sh \
--bootstrap-server kafka-1:9092,kafka-2:9092,kafka-3:9092 \
--topic Test --from-beginning

```

With a consumer in place, we can start producing messages

```
docker exec -it zookeeper-1 bash

echo "New Message: 1" | \
/opt/kafka/bin/kafka-console-producer.sh \
--broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
--topic Test > /dev/null
```


Once we have a message in Kafka, we can explore where it got stored in which partition