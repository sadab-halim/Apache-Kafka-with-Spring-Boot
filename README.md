# Apache Kafka with Spring Boot

# Introduction to Apache Kafka
## Event-Driven Architecture with Apache Kafka

## Apache Kafka for Microservices
Apache Kafka is a distributed event streaming platform that is used to collect, process, store and integrate data at scale.

_pic 1 to be added_ <br/>
_pic 2 to be added_ 

## Messages and Events
_pic 1 to be added_

- In Apache Kafka, an event is an indication that _something has happened_.
  - Example:
    - UserLoggedInEvent
    - ProductCreatedEvent
    - OrderCreatedEvent
- Syntax: <Noun><PerformedAction>Event
  - Example:
    - ProductCreatedEvent
    - ProductShippedEvent
    - ProductDeletedEvent

### Kafka Message
- Kafka Message can be considered as an envelope that carries something inside and the event data are the content of this message envelope and this event data. 
- It can be of different formats, a simple string value.
- It can be a Json payload, it can be in other format as well, and even null if needed.
- The default message size of the payload is 1 Mb.
- Kafka message is sent over the network, so the larger the size of the message, the slower your system will perform.
- Whatever information we decide to include in the message payload, it will be serialized into an array of bytes by producer microservice.
- It will be sent over a network and stored in a topic as a byte array
- And when consumer microservice receive this message, the payload will be deserialized into a destination data format.
- Second Main Part of Kafka Message is message key, which can also be of different data types including null value.
- Kafka message can be considered as a key-value pair where both key and the value can be of different data types, including null value.
- Third Important Part is timestamp. This value can be set by the system or we can set this value manually.
- It is important to have timestamp because there can be use cases where we need to know at what time in the history event took place.
- Other important part is the Headers. Headers is a list of key-value pairs that we can use to include additional metadata information to our Kafka message. 
  - For example, we can include authorization header that the destination microservice might need to access

_pic 2 to be added here_

## Kafka Topic and Partitions
### Kafka Topics
- Topics are partitioned and each partition is replicated across multiple Kafka server for durability.
- This way, even if there is an issue with one Kafka server, you still have your data stored on other Kafka servers.
- So, all events that products microservice will publish will first be stored in Kafka topic and microservices that are interested in receiving these events they will read these vents from Kafka topic.
  _pic 1 to be added_ <br/>
- Topic will be referred by its name, each topic should have a unique name.
- Topic is split into three partitions (can vary)
  - Helpful: consuming microservices can read data from topic partitions in parallel, and this can help us increase throughput, scale our application and make our system work faster.

- We specify how many partitions to create in a topic while creating the topic.
- Once the topic is created, we can also increase number of partitions if needed.
- You can increase number of partitions, but you cannot decrease it.

### Kafka Partitions
- Each partition is acts like a small storage unit that holds data and it is stored on a hard disk on Kafka server.
- Each cell in a partition is numbered. This number is called **offset**. Offset is similar to index in java array. It starts with 0
- Kafka topic is always added at the end.
- Once you store event to a partition, you cannot change it, you cannot delete it, and you cannot update it. Just like a real life event
- You also cannot insert events in the middle of partition.
- You cannot insert it somewhere between of existing events.
- A new entry is always added at the end of the partition, and once the message is persistent, its offset is increased by one.
- Events are in order with a partition. Order across partitions is not maintained.
- Event data which is stored in kafka topic is immutable and we cannot change change it and we cannot delete it.
- Also, after the message is consumed from Kafka topic, it remains there and does not get deleted.
- It can stay there for as long as needed, but it does not stay there forever.
- By default, Kafka topic configured to retain events for seven days, but this value can change through configuration properties if needed.


## Ordering of Events
- Kafka message is a key-value pair, where message key can be a string value and message value can be event details in the form of Json payload.
- When message key is not provided, then this event can be stored in any partition.
- Kafka makes the decisions for us. When message key is not provided, Kafka will try to load balance events and store a new event in one of the available partitions.
- Also, Consumer microservice will consume events from topic partitions in parallel.
- And because, Kafka consumer reads events in parallel, there is no guarantee in what order consumer microservice will receive these events.

- So, in the situations, where the order in which events should be processed is important, we should always provide message key.
- If message key is provided, then Kafka will use it to determine which partition this event should be written to in Kafka topic.
- When the same message key is used for different events, those events will be stored in the same topic.
- If we publish one more event with exactly the same message key, Kafka will store this event in the same partition as other events that have exactly the same message key.
- It'll take message key, it will hash it, and then it will use this hash value to determine which partition to use to store this event.
- And it always stores messages with the same message key in the same partition. And this is how Kafka helps us to achieve ordering of events.
- Remember: to store the related events in the same topic partition, each of these events should have the same message key.
  - For example: all events that are related to one particular product will be grouped together under one ID, and all events that are related to another product can be grouped together under another message ID.
  - This is needed so that consumer microservice can process events related to the same product in the same order in which they were published.

_pic 3 to be added here_

-----------------------------------------------------------------------

# Apache Kafka Broker(s)

## What is Apache Kafka Broker?
- Kafka Broker can be a physical computer or it can be a virtual machine that runs Kafka processes.
- And to make your system always available and fault tolerant, you can start multiple brokers in a cluster.
- These brokers work together to make sure that your events are stored reliably.
- Their will be one broker that will act as a leader and other will act as followers.
- Each of these broker hosts Kafka topics, and each Kafka Topic stores events in partitions.
- The leader will handle all read and write requests for the partitions, while followers will passively replicate leader's data.
- This way, Kafka can achieve high availability and fault tolerance.
- The leader and the follower roles are not fixed to a single broker, but change dynamically depdencing on availability of the broker.
- If a leader goes gown for some reason, one of the follower brokers will become a new leader and will continue servicing requests.
- Kafka Broker is a software application that will run on one or moe computers.
- Kafka Broker will handle all the work to accept messages from producer, store it reliability in topic partitions, and replicate it across other brokers in the cluster.
- It will also handle all the work to enable consumer microservices read events from topic partitions.
- Kafka Broker is customizable and you can configure its behavior using configuration file.

_pic to be added here_

## Leader
- A leader is responsible for handling all read and write requests for partiions in a tpics.
- It;s called a leader because it leads or it manages all operations for topic partisions.
- Think of a leader as of a amanger in a store who is in change of a specific sectin.
- If Kafka client wants to write a message into a topic partition, it will need to go through a leader.
- And once the message is succesfully persisted in topic parition, it will be replicated to other Kafka brokers that act as followers.
- This replication is controlled and consistent.
- Followers replicate that from the leader in exact order it was written, maintaining data consistency across cluster.
- So leader is a single source of truth for all writes and reads in topic parition.
- If followers were allowed to accept writed, it could lead to inconsistencies and conflicts because different followers might have different versions of the data.

### Follower
- A follower in Kafka is a replica of partition.
- It copies all the data from the leader.
- It follows a leader, meaning that it replicates whatever the leader has.
- If leader fails one of the follower can take over.
- But if you have only one broker in the cluster and if that broker goes down, then your entire system is down and there is no replica of your data.
- Having multiple brokers in the cluster allows you to have more followers, and this helps you to ensure high availability of the data.
- If one broker including leader goes down, then data is still available through other brokers.
- Followers allows you to scale your Kafka cluster horizontally.
- As we add more brokers, we can increase the number of followers and thereby enhance data redundancy and the capacity to handle more data and more clients.
- But it is not always that the same Kafka broker is a leader, and all other Kafka brokers are always followers.
- If the same Kafka broker is always a leader, this will mean that all read and write operations are always done through the same server, and this would create a bottleneck.
- To avoid bottleneck:
  - Every Kafka broker can be a leader and a follower at the same time.
- Each partition will have a leader assigned to it, and a leader to a partition is assigned when the topic is created.
- When you create a topic in Kafka, it also involves specifying the number of partitions for that topic.
- Kafka, through its internal processes, assigns a leader for each partition right away, so each partition will have its own leader and followers.

## Start Single Apache Kafka Broker with KRaft
- Kraft is name of consensus protocol that Kafka uses to coordinate the metadata changes among brokers of Kafka servers.
- kraft folder is available inside config folder of kafka
- `controller.properties` : contains configuration for the sever that acts as a controller, which is responsible for managing the cluster metadata and coordinating the leader election for partitions.
- `server.properties` : contains configuration for a server that acts as both controller and broker.
- `broker.properties` : contains configuration for server that acts as a broker which is responsible for storing and serving data for topics and partitions.

### Before starting kafka server
- command to generate a unique ID for my kafka cluster: `./bin/kafka-storage.sh random-uuid`
  - it generates a unique identifier that will required to use to configure kafka cluster
- command to format log directories using the unique identifier: `./bin/kafka-storage.sh format -t generated_unique_id -c conifg/kraft/server.properties`
  - this will prepare the storage directories for running kafka in craft mode.
- command to run kafka server: `./bin/kafka-storage.sh config/kraft/server.properties`

## Multiple Kraft Broker: Configuration Files
- Open Kafka folder first, config >> kraft >> create new server.property file for each new server.
- Each Kafka Server must have a unique node id.
- The next configuration property that need to be updated is called **listeners**.
  - This property is used to specify a list of listeners in Kafka Cluster.
  - And these are network interfaces that Kafka uses to communicate with other servers and clients.
  - Now, this configuration file, it is called sever properties, and it is used to configure Kafka Server that performs two roles.
    - A role of a broker and a role of a controller.

## Multiple Kafka Broker: Storage Folders
- In order to prepare the storage directory for Kafka Server, we will need to first generate a unique identifier for our Kafka cluster.
- Make sure that you are in the Kfka folder.
- Run the following command: 
  - `kafka-storage.sh random-uuid`
  -  for server 1: `kafka-storage.sh format -t generatedRandomUUID -c config/kraft/server-1.properties`
  -  for server 2: `kafka-storage.sh format -t generatedRandomUUID -c config/kraft/server-2.properties`
  -  for server 3: `kafka-storage.sh format -t generatedRandomUUID -c config/kraft/server-3.properties`
- 

## Starting Multiple Kraft Broker
- Run the following command: 
  - `kafka-server-start.sh config/kraft/server-1.properties`
  - `kafka-server-start.sh config/kraft/server-2.properties`
  - `kafka-server-start.sh config/kraft/server-3.properties`

## Stopping Apache Kafka Brokers
- Before shutting down your severs, it's a good practice to stop all running producers and consumers.
- It helps to avoid loosing messages
- Stopping producers and consumers first also helps you to avoid erros in your system.
- This is because, if your produces and consumers keep on interacting with the server that is going down, this will cause exceptions.
- These exception will mostly kick in the exception handling or retry logic on your application which might complicate everything.
- Good Practice: Stop all running producers and consumers
- Stopping producers and consumers keep on interactions with the server that is going down, this will cause exceptions.
- Ctrl + C is an option
- But, a better way to shut down the servers gracefully is by using the following command: `kafka-server-stop.sh`
- Graceful or controlled shutdown of Kafka Server is enabled by default and the controlled shutdown of Kafka Server is controlled with the property that is called controlled shutdown.
- Enabled by default, this property is enabled and its value is true.

----------------------------------------------------------------------

# Kafka CLI: Topics
Kafka topic CLI is used to manage and interact with topics within Kafka cluster using command line interface.
## Introduction to Kafka Topic: CLI
- kafka-topics.bat
  - creates a new topic
  - list
  - describe
  - delete
  - modify
  
## Creating a new Kafka Topic
- first start the kafka server
- topic name must be unique and meaningful
- topics ca have letters, digits, but not special characters
- `./kafka-topics.sh --create --topic topicName --partitions 3 --replication-factor 3 --bootstrap-server localhost:9092, localhost:9094` 

## List and Describe Kafka Topic
- `./kafka-topics.sh --list --bootstrap-server localhost:9092`
- to see additional information
  - `./kafka-topics.sh --describe --bootstrap-server localhost:9092`

## Deleting Kafka Topic
- `./kafka-topics.sh --delete --topic topicName --bootstrap-server localhost:9092`

----------------------------------------------------------------------

# Kafka CLI: Producers
- The main use of `kafka-console-producer.sh` script is to send a message to a particular Kafka topic.
- Send message with a Key, without a Key
- Send multiple messages from a file
- Example: `kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic`

## Producing Kafka Message without a Key
- Run the following command: `kafka-console-producer.sh --bootstrap-sever --topic my-topic localhost:9092, localhost:9094`
- If you send a message to a topic that does not exist, Kafka Producer will respond with error, but it will also automatically create the topic.
- This is the default configuration and it is controlled with the following property in the broker configuration.
- The default value of this property is true: `auto.create.topics.enable=true`

## Sending Kafka Message as a Key-Value Pair
- Run the following command: `kafka-console-producer.sh --bootstrap-sever --topic my-topic --property "parse.ley=true" --property "key.separator=:" localhost:9092, localhost:9094`
- When the passkey property is set to true, producer excepts each message to be in the format of key-value pair, and then we can specify the separator character which should be used between key and the value.
- But, if we send a message that does not include a valid separator, I should get an error.

-----------------------------------------------------------------------------

# Kafka CLI: Consumers
- The main use of `kafka-console-consumer.sh` is to fetch and display messages from a Kafka topic to your terminal.
- Read messages only
- Read all messages from the beginning
- Example: `kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092`

## Consuming Messages from Kafka Topic from the Beginning
Run the following command: `kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092`

## Consuming New Kafka Messages Only
Run the following command: `kafka-console-consumer.sh --topic my-topic --bootstrap-server localhost:9092`
 
## Consuming Key:Value Pair Messages from Kafka Topic
- Run the following command: 
  - prints key-value: `kafka-console-consumer.sh --topic my-topic --bootstrap-server localhost:9092 --property print.key=true`
  - prints only key: `kafka-console-consumer.sh --topic my-topic --bootstrap-server localhost:9092 --property print.value=false`

## Consuming Kafka Messages In Order
Run the following command: `kafka-console-consumer.sh --topic messages-order --bootstrap-server localhost:9092 --from-beginning --property print.key=true`

---------------------------------------------------------------

# Kafka Producer
- The primary role of a Kafka Producer is to publish messages or events to one or more Kafka topics.
- In Spring Boot App, this is typically achieved using Spring for Apache Kafka library, which simplifies the process of integrating Kafka functionality into your spring application.
- Before sending message, Kafka Producer serializes it into a byte array. So, another responsibility of Kafka Producer is to serialize messages to binary format that can be transmitted over the network to Kafka brokers.
- Another responsibility of Kafka Producer is to specify the name of Kafka Topic, where these messages should be sent.
- When sending a new message to the Kafka Topic, we will also specify the partition where we want to store it, although this parameter is optional and if we do not specify topic patter, then Kafka producer will make this decision for us.
- And if neither partition nor key is provided, then Kafka producer will distribute messages across partition in a round-robin fashion, ensuring a balanced load.
- If a partition is not specified but a message key is provided, then Kafka will hash the key and use the result to determine the partition, and this ensures that all messages with the same key got to the same partition.














