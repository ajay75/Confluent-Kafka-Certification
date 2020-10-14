**Is there a way to purge the topic in kafka?**

Not sure but sounds like it should work
kafka-delete-records --bootstrap-server <kafka_server:port> --offset-json-file delete.json

The structure of the delete.json file should be following:

{ "partitions": [ { "topic": "foo", "partition": 1, "offset": -1 } ], "version": 1 }

where offset :-1 will delete all the records (This command has been tested with kafka 2.0.1

An accepted answer was: 
kafka-configs.sh --zookeeper <zkhost>:2181 --entity-type topics --alter --entity-name <topic name> --add-config retention.ms=1000
then wait for the purge to take effect (about one minute). Once purged, restore the previous retention.ms value.

**How can I send large messages with Kafka (over 15MB)?**
https://stackoverflow.com/questions/21020347/how-can-i-send-large-messages-with-kafka-over-15mb
You need to adjust following properties:
1. fetch.message.max.bytes -- this to set the size of message that can be fetched by the consumer
2. replica.fetch.max.bytes -- allow the replicas to replicate the message correctly, because if it is too
   smaller than  message will never be replicated therefore consumer will never see this message.
3. message.max.bytes -- the largest size of message that broker can receive from the producer.
4. max.message.bytes -- this is the largest size of message the broker will allow to append to the topic.
   The size is validated pre-compression ( Default to message.max.bytes)
   
** Kafka delivery semantics**
https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e 
At-most-once: Acks =0::: (fire and forget approach with no retris) the message is either delivered or not delivered.
  That means you can lose some message, it is useful for high throughput since 
  there is no overhead for an acknowledgement from brokers.
At-least-once: Acks =1, message can be delivered and consumed several times, so message
   does not get lost. But there is little concern with duplication of data.
Exactly-once: Acks =all (enable,idempotence = true) message are delivered and read once. Very important in financial 
    translations. acks=all must be used in conjunction with 
    min.insync.replicas which can be set at a broker or topic level.
   #### A kafka topic with replication.factor=3, acks=all, min.insync.replicas=2 can only tolerate 1 broker going down, otherwise the producer will receive an exception NOT_ENOUGH_REPLICAS on send.
   #### A kafka topic with replication.factor=3, acks=all, min.insync.replicas=1, can tolerate maximum number of 2 brokers going down, so that a producer can still produce to the topic.
No guarantee: enable.auto.commit =true ( this is default) . The frequency of these
    commits depends on the configuration of auto.commit.interval.ms 
