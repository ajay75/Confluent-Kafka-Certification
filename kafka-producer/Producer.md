# Kafka producer

## Key points

**What happens after producerRecord is sent?

    Step #1: Once we send the ProducerRecord, the first thing the producer will do is serialize the key and value objects to ByteArrays so they can be sent over the network
    Step #2: Data is sent to partitioner, If we specified a partition in the ProducerRecord, the partitioner doesn’t do anything and simply returns the partition we specified.
             If we didn’t, the partitioner will choose a partition for us, usually based on the ProducerRecord key.
    Step #3: Adds the record to a batch of records that will also be sent to the same topic and partition
    Step #4: A separate thread is responsible for sending those batches of records to the appropriate Kafka brokers.
    Step #5: Broker receives the messages, it sends back a response
             Successful : it will return a RecordMetadata object with the topic, partition, and the offset of the record within the partition.
             Failed: it will return a error code, producer may retry sending few more times before giving up and returning an error.


**When will a produced message will be ready to consume?**

    Messages written to the partition leader are not immediately readable by consumers regardless of the producer's acknowledgment settings.
    When all in-sync replicas have acknowledged the write, then the message is considered committed, which makes it available for reading.
    This ensures that messages cannot be lost by a broker failure after they have already been read.

**What are the producer key configuration?**

    key configurations are explained here : https://docs.confluent.io/current/clients/producer.html

**key.serializer**

    producer interface allows user to provide the key.serializer.
    Available serializers: ByteArraySerializer, StringSerializer and IntegerSerializer.
    setting key.serializer is required even if you intend to send only values.

**value.serializer**

    producer interface allows the user to provide the value.serializer. like the same way as key.serializer.

**retries**

    Setting a value greater than zero will cause the client to resend any record whose send fails with a potentially transient error. Note that this retry is no different than if the client resents the record upon receiving the error. Allowing retries without setting max.in.flight.requests.per.connection to 1 will potentially change the ordering of records because if two batches are sent to a single partition, and the first fails and is retried but the second succeeds, then the records in the second batch may appear first. Note additionally that produce requests will be failed before the number of retries has been exhausted if the timeout configured by delivery.timeout.ms expires first before the successful acknowledgment. Users should generally prefer to leave this config unset and instead use delivery.timeout.ms to control retry behavior.
    default: 2147483647
    By default, the producer will wait 100ms between retries, but you can control this using the retry.backoff.ms parameter.

**acks**

    If acks=0, the producer will not wait for a reply from the broker before assuming the message was sent successfully.
    If acks=1, the producer will receive a success response from the broker the moment the leader replica received the message
                If the client uses callbacks, latency will be hidden, but throughput will be limited by the number of in-flight messages (i.e., how many messages the producer will send before receiving replies from the server).
    If acks=all, the producer will receive a success response from the broker once all in-sync replicas received the message.
                if acks is set to all, the request will be stored in a buffer called purgatory until the leader observes that the follower replicas replicated the message, at which point a response is sent to the client

**buffer.memory**

    This sets the amount of memory the producer will use to buffer messages waiting to be sent to brokers.
    If messages are sent by the application faster than they can be delivered to the server, the producer may run out of space and additional send() calls will either block or throw an exception, based on the block.on.buffer.full parameter (replaced with max.block.ms in release 0.9.0.0, which allows blocking for a certain time and then throwing an exception).

**batch.size**

    When multiple records are sent to the same partition, the producer will batch them together.
    This parameter controls the amount of memory in bytes (not messages!) that will be used for each batch.
    When the batch is full, all the messages in the batch will be sent. However, this does not mean that the producer will wait for the batch to become full.
    The producer will send half-full batches and even batches with just a single message in them.
    Therefore, setting the batch size too large will not cause delays in sending messages; it will just use more memory for the batches.
    Setting the batch size too small will add some overhead because the producer will need to send messages more frequently.

**linger.ms**

    linger.ms controls the amount of time to wait for additional messages before sending the current batch.
    KafkaProducer sends a batch of messages either when the current batch is full or when the linger.ms limit is reached.
    By default, the producer will send messages as soon as there is a sender thread available to send them, even if there’s just one message in the batch.
    By setting linger.ms higher than 0, we instruct the producer to wait a few milliseconds to add additional messages to the batch before sending it to the brokers.
    This increases latency but also increases throughput (because we send more messages at once, there is less overhead per message).

**compression.type**

    By default, messages are sent uncompressed
    supported compression types: snappy, gzip and lz4.
    snappy is recommended, with low CPU and good performance and decent compression ratio.
    Gzip use more CPU and time, but result in better compression ratio.

**max.in.flight.requests.per.connection**

    This controls how many messages the producer will send to the server without receiving responses.
    Setting this high can increase memory usage while improving throughput, but setting it too high can reduce throughput as batching becomes less efficient.
    Setting this to 1 will guarantee that messages will be written to the broker in the order in which they were sent, even when retries occur.

**Methods of sending messages**

    * Fire-and-forget :
        We may lose data in this situation. Possible cases of losing data: SerializationException when it fails to serialize the message, a BufferExhaustedException or TimeoutException if the buffer is full, or an InterruptException if the sending thread was interrupted.
        not recommended for production use.
    * Synchronous send
        We user Future.get() to wait for a reply from Kafka
    * Asynchronous send
        We call the send() method with a callback function, which gets triggered when it receives a response from the Kafka broker.

**Type of errors**

    Retriable: errors are those that can be resolved by sending the message again.
              For example, a connection error can be resolved because the connection may get reestablished.
              A “no leader” error can be resolved when a new leader is elected for the partition.
              KafkaProducer can be configured to retry those errors automatically.
    non-retriable: For example, “message size too large.” In those cases, KafkaProducer will not attempt a retry and will return the exception immediately.

**Explain bootstrap.servers**

    bootstrap.servers property so that the producer can find the Kafka cluster.

**Explain client.id**

    Although not required, you should always set a client.id since this allows you to easily correlate requests on the broker with the client instance which made it.


**Where can I find the full list of configs documentation?**

    https://docs.confluent.io/current/installation/configuration/producer-configs.html#cp-config-producer

**unclean.leader.election.enable**

    the default value is true, this will allow out of sync replicas to become leaders.
    This should be disabled in critical applications like banking system managing transactions.

**min.insync.replicas**

    This will ensure the minimum number of replicas are in sync.
    NotEnoughReplicasException will be thrown to the producer when the in-sync replicas are less then what is configured.
    
**Which errors are retriable from Kafka Producer?**
```
    LEADER_NOT_AVAILABLE, 
    NOT_LEADER_FOR_PARTITION, 
    UNKNOWN_TOPIC_OR_PARTITION
```  
**Kafka Producer throw error for following non-retriable errors:**
```
    OFFSET_OUT_OF_RANGE
    BROKER_NOT_AVAILABLE
    MESSAGE_TOO_LARGE
    INVALID_TOPIC_EXCEPTION
```  
**When produce to a topic which doesn't exist and auto.create.topic.enable=true** 
    then kafka creates the topic automatically with the broker/topic settings num.partition and default.replication.factor  
  
**What is a generic unique id which can be used for a message received from a consumer?**
    Topic + Partition + Offset      
    
**Safe Producer Configuration**    
```
min.insync.replicas=2 (set at broker or topic level)
retries=MAX_INT number of reties by producer in case of transient failure/exception. (default is 0)
max.in.flight.per.connection number=5 number of producer request can be made in parallel (default is 5)
acks=all
enable.idempotence=true producer send producerId with each message to identify for duplicate msg at kafka end. 
When kafka receives duplicate message with same producerId which it already committed. It do not commit it again and send ack to producer (default is false)
```
**High Throughput Producer using compression and batching**   
```
compression.type=snappy value can be none(default), gzip, lz4, snappy. 
    Compression is enabled at the producer level and doesn't require any config change in broker or consumer 
    Compression is more effective in case of bigger batch of messages being sent to kafka
linger.ms=20 Number of millisecond a producer is willing to wait before sending a batch out. 
    (default 0). Increase linger.ms value increase the chance of batching.
batch.size=32KB or 64KB Maximum number of bytes that will be included in a batch (default 16KB).
    Any message bigger than the batch size will not be batched.
```
**Message Key**  
```
Producer can choose to send a key with message.
If key = null, data is send in round robin
If key is sent, then all message for that key will always go to same partition. 
    This can be used to order the messages for a specific key since order is guaranteed in same partition.
Adding a partition to the topic will lose the guarantee of same key go to same partition.
Keys are hashed using murmur2 algorithm by default.
```

Setting the retries parameter to nonzero, and the
**max.in.flights.requests.per.session** (Note the word session not connection)
 m ax.in.flights.requests.per.session value to more than one means that it is possible that the broker will fail to write the first batch of
 messages, succeed to write the second (which was already inflight),
 and then retry the first batch and succeed, thereby reversing the order.
 so to guaranteeing order is critical, we recommend
 setting in.flight.requests.per.session=1 to make sure that
 while a batch of messages is retrying, additional messages will not
 be sent (because this has the potential to reverse the correct order).
 This will severely limit the throughput of the producer, so only use
 this when order is important.