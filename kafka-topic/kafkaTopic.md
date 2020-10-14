Using Per-Topic Overrides
```
log.retention.hours.per.topic,
log.retention.bytes.per.topic,  
log.segment.bytes.per.topic. These parameters are no longer supported,
and overrides must be specified using the administrative tools.
```
auto.create.topics.enable=true
```
num.partitions parameter determines how many partitions a new topic is created
with, primarily when automatic topic creation is enabled. Number of
partitions for a topic can only be increased, never decreased

When a producer starts writing messages to the topic
When a consumer starts reading messages from the topic
When any client requests metadata for the topic
```
log.retention.bytes 
```
it is applied per-partition. This means that if you have a topic with 8 partitions, and
log.retention.bytes is set to 1 GB, the amount of data retained for the topic will be
8 GB at most.Note that all retention is performed for individual partitions, not the
topic.
```
log.segment.bytes
```
Adjusting the size of the log segments can be important if topics have a low produce
rate. For example, if a topic receives only 100 megabytes per day of messages, and
log.segment.bytes is set to the default (1 GB), it will take 10 days to fill one segment. As
messages cannot be expired until the log segment is closed, if log.retention.ms is
set to 604800000 (1 week), there will actually be up to 17 days of messages retained
until the closed log segment expires. This is because once the log segment is closed
with the current 10 days of messages, that log segment must be retained for 7 days
before it expires based on the time policy (as the segment cannot be removed until
the last message in the segment can be expired).
Another way to control when log segments are closed is by using the log.segment.ms
parameter, which specifies the amount of time after which a log segment should be
closed. As with the log.retention.bytes and log.retention.ms parameters,
log.segment.bytes and log.segment.ms are not mutually exclusive properties.
Kafka will close a log segment either when the size limit is reached or when the time
limit is reached, whichever comes first

```
message.max.bytes/fetch.message.max.bytes/replica.fetch.max.bytes  
```
message.max.bytes defaults to 1 MB. If producer that tries to send a message larger than this will receive an error back from
the broker, and the message will not be accepted.The message size configured on the Kafka broker must be coordinated with the 
fetch.message.max.bytes configuration on consumer clients. If this value is smaller than message.max.bytes,
then consumers that encounter larger messages will fail to fetch those messages, resulting in a situation where the consumer gets
stuck and cannot proceed. The same rule applies to the **replica.fetch.max.bytes** configuration on the brokers when configured
in a cluster.