# messaging-overview

Comparison of managed messaging and streaming solutions available on AWS platform.

| Parameter                        | SQS / SQS FIFO                      | SNS / SNS FIFO                                      | EventBridge                                     | Kinesis Data Streams                    | IoT Core Broker                                         | MSK/Kafka (2.2.1-2.7.0)                           | Amazon MQ (RabbitMQ 3.8.6)                            | Amazon MQ (ActiveMQ 5.15 Classic)                                                                  | ElastiCache (Redis 6.x)                                   |
| -------------------------------- | ----------------------------------- | --------------------------------------------------- | ----------------------------------------------- | --------------------------------------- | ------------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Infrastructure                   | Serverless                          | Serverless                                          | Serverless                                      | Serverless                              | Serverless                                              | Managed (EC2)                                     | Managed (EC2)                                         | Managed (EC2)                                                                                      | Managed (EC2)                                             |
| HA                               | Regional                            | Regional                                            | Regional                                        | Regional                                | Regional                                                | Multi-AZ                                          | Multi-AZ                                              | Multi-AZ                                                                                           | Multi-AZ                                                  |
| SLA                              | 99.90%                              | 99.90%                                              | 99.99%                                          | 99.90%                                  | 99.90%                                                  | 99.90%                                            | 99.90%                                                | 99.90%                                                                                             | 99.90%                                                    |
| Order                            | Best-effort/FIFO                    | best-effort/FIFO(1)                                 | best effort (no order) (3)                      | Ordered (within a shard)                | N (9)                                                   | Y                                                 | Y                                                     | Y                                                                                                  | Y (Streams) / N (Pub/Sub)                                 |
| Delivery                         | At least once / exactly once (FIFO) | At least once / exactly once (FIFO) (4)             | At least once                                   | At least once                           | QoS0 (can drop message), QoS1 (at least once) (9)       | At least once, At most once, Exactly once (55,56) | At least once, At most once (54)                      | Depends on protocol (exactly once is possible with MQTT/QoS2)(14)                                  | At least once (Streams) / At most once (Pub/Sub) (15)     |
| Routing                          | N                                   | Y (via filtering) (62)                              | Y (63)                                          | Y (b/w shards using partition key) (64) | Y (65)                                                  | N (66)                                            | Y (67)                                                | Y (with Apache Camel) (68, 69)                                                                     | Y (Pub/Sub pattern matching) (70) / N (Streams)           |
| Max message size                 | 256Kb                               | 256 Kb                                              | 256 Kb                                          | 1MB                                     | 128 Kb                                                  | 1 MB (default)                                    | 128Mb (in v3.8.0+)/2Gb (up to v3.7.0) (61)            | some Gb (limited by RAM, disk size) (59, 60)                                                       | 32 Mb/min* (Pub/Sub) (75) / 512 Mb (76)                    |
| Max rate, req/s                  | 3000msg/sec (batch mode)            | 3000msg/sec or 10MB/sec (batch mode)                | 2400 put msg/s; 4500 invoc/s (FRA, soft limit), | 1000 msg/sec/shard (max 1Mb/s) (24)     | 100 req/sec, 512Kb/s (31)                               | nM msg/sec (11, 12)                               | 1M msg/sec (10)                                       | 22K msg/sec (58)                                                                                   | 1M msg/sec (16)                                           |
| Retention, max                   | 14 days                             | 6h (SMTP, SMS, mobile push) /23d (SQS, Lambda) (19) | 24h (5)                                         | 1Y                                      | 1h (retry/QoS1) (31), 7d (QoS1 persistent sessions, 42) | forever (limited by disk)                         | Limited by disk? (13)                                 | forever (limited by disk)                                                                          | forever (Streams, limited by disk) (72) / N (PubSub) (71) |
| Replay                           | N (2)                               | N (2)                                               | Y (3)                                           | Y                                       | N                                                       | Y                                                 | N (25)                                                | N (27)                                                                                             | Y (Streams) / N (Pub/Sub)                                 |
| P2P vs Pub/Sub                   | P2P                                 | Pub/Sub                                             | Pub/Sub                                         | Pub/Sub                                 | Pub/Sub                                                 | Pub/Sub                                           | Pub/Sub                                               | Pub/Sub                                                                                            | *                                                         |
| Push vs Pull                     | Pull                                | Push                                                | Push                                            | Pull                                    | Push                                                    | Pull                                              | Both                                                  | Both                                                                                               | Pull (Streams) (74) / Pull (Pub/Sub) (73)                 |
| Encryption in transit            | Y                                   | Y                                                   | Y                                               | Y                                       | Y                                                       | Y                                                 | Y                                                     | Y                                                                                                  | Y (18)                                                    |
| Encryption at rest               | Y                                   | Y                                                   | Y                                               | Y                                       | ?                                                       | Y                                                 | Y                                                     | Y                                                                                                  | Y (17)                                                    |
| DLQ                              | Y                                   | Y (7)                                               | Y (23)                                          | N (21, 22)                              | N                                                       | Y (via Connect/emulation) (30)                    | Y (28)                                                | Y (26)                                                                                             | Y (29)                                                    |
| Message Schema (on AWS)          | N                                   | N                                                   | Y (33)                                          | Y (with KPL/KCL) (34)                   | N                                                       | Y (32,35)                                         | N                                                     | N                                                                                                  | N                                                         |
| Standards, Protocols, Transports | JMS1.1, REST (36)                   | REST                                                | REST                                            | REST                                    | REST, WebSockets, MQTT* (37)                            | Proprietary                                       | AMQP, STOMP, MQTT, HTTP, WebSockets (38), JMS1.1 (39) | AMQP, AUTO, MQTT, OpenWire, REST, RSS/Atom, STOMP, WSIF, WS Notification, XMPP, JMS 1.1 (40,41,57) | Proprietary                                               |
| Data type                        | Text, XML, JSON (46)                | Text, JSON (45)                                     | JSON (47)                                       | Base64-ed blob (44)                     | JSON, Binary (43)                                       | Binary (48)                                       | *                                                     | *                                                                                                  | Binary-safe K/V (49)                                      |
| Processing Latency               | 0.13-1.3sec (6,53)                  | seconds (51)                                        | 0.5 sec (4)                                     | 200ms-1s (50)                           | ?                                                       | 5-48ms (52,53)                                    | 1-120ms (52,53)                                       | 49ms (53)                                                                                          | 1ms (16)                                                  |

(1) https://aws.amazon.com/blogs/compute/building-event-driven-architectures-with-amazon-sns-fifo/ and https://aws.amazon.com/about-aws/whats-new/2020/10/amazon-sns-introduces-fifo-topics-with-strict-ordering-and-deduplication-of-messages/

(2) Replay implementation with SNS + 2 x SQS https://docs.aws.amazon.com/sns/latest/dg/sns-fork-pipeline-as-subscriber.html#sns-fork-event-replay-pipeline

(3) https://aws.amazon.com/blogs/aws/new-archive-and-replay-events-with-amazon-eventbridge/

(4) https://docs.aws.amazon.com/sns/latest/dg/fifo-message-dedup.html and SNS vs EventBridge https://cloudonaut.io/eventbridge-vs-sns/

(5) https://docs.aws.amazon.com/eventbridge/latest/userguide/create-eventbridge-scheduled-rule.html

(6) https://softwaremill.com/amazon-sqs-performance-latency/

(7) DLQ https://aws.amazon.com/about-aws/whats-new/2019/11/amazon-sns-adds-support-for-dead-letter-queues-dlq/ and https://aws.amazon.com/blogs/compute/designing-durable-serverless-apps-with-dlqs-for-amazon-sns-amazon-sqs-aws-lambda/

(8) https://aws.highspot.com/items/5f5fe9b260e9cc1120781e22?lfrm=srp.0#31 - comparison of different solutions

(9) https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html

(10) https://tanzu.vmware.com/content/blog/rabbitmq-hits-one-million-messages-per-second-on-google-compute-engine

(11) https://events.static.linuxfound.org/sites/events/files/slides/Kafka%20At%20Scale.pdf

(12) https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines and 15K msg/sec https://aiven.io/blog/benchmarking-kafka-write-throughput-performance-2019-update

(13) https://www.rabbitmq.com/ttl.html

(14) https://activemq.apache.org/components/artemis/documentation/2.3.0/protocols-interoperability.html

(15) https://redislabs.com/solutions/use-cases/messaging/

(16) https://devopedia.org/redis-streams

(17) https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/at-rest-encryption.html

(18) https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/in-transit-encryption.html

(19) https://docs.aws.amazon.com/sns/latest/dg/sns-dead-letter-queues.html

(20) https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/data-protection.html

(21) https://acloudguru.com/blog/engineering/aws-lambda-3-pro-tips-for-working-with-kinesis-streams?utm_source=medium_blog&utm_medium=redirect&utm_campaign=medium_blog

(22) https://acloudguru.com/blog/engineering/use-sns-to-retry-failed-kinesis-events

(23) https://docs.aws.amazon.com/eventbridge/latest/userguide/rule-dlq.html

(24) https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html

(25) https://www.cloudkarafka.com/blog/which-service-rabbitmq-vs-apache-kafka.html

(26) https://activemq.apache.org/message-redelivery-and-dlq-handling

(27) History can be emulated via Mirrored queues http://activemq.apache.org/mirrored-queues.html

(28) https://www.rabbitmq.com/dlx.html

(29) https://redis.io/topics/streams-intro

(30) https://www.confluent.io/blog/kafka-connect-deep-dive-error-handling-dead-letter-queues/ and https://towardsdatascience.com/dead-letter-queue-dlq-in-kafka-29418e0ec6cf

(31) https://docs.aws.amazon.com/general/latest/gr/iot-core.html

(32) https://medium.com/@stephane.maarek/introduction-to-schemas-in-apache-kafka-with-the-confluent-schema-registry-3bf55e401321

(33) https://aws.amazon.com/blogs/compute/introducing-amazon-eventbridge-schema-registry-and-discovery-in-preview/

(34) https://docs.aws.amazon.com/streams/latest/dev/kpl-with-schemaregistry.html

(35) https://docs.aws.amazon.com/glue/latest/dg/schema-registry-integrations.html#schema-registry-integrations-amazon-msk

(36) https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-java-message-service-jms-client.html

(37) https://www.hivemq.com/blog/hivemq-cloud-vs-aws-iot/

(38) https://www.rabbitmq.com/protocols.html

(39) https://www.rabbitmq.com/jms-client.html

(40) https://activemq.apache.org/protocols

(41) https://activemq.apache.org/components/artemis/documentation/1.0.0/using-jms.html

(42) https://docs.aws.amazon.com/general/latest/gr/iot-core.html#message-broker-limits

(43) https://docs.aws.amazon.com/iot/latest/developerguide/binary-payloads.html

(44) https://docs.aws.amazon.com/kinesis/latest/APIReference/API_Record.html https://docs.aws.amazon.com/streams/latest/dev/fundamental-stream.html

(45) https://docs.aws.amazon.com/sns/latest/api/API_Publish.html

(46) https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html

(47) https://docs.aws.amazon.com/eventbridge/latest/APIReference/API_PutEvents.html

(48) https://kafka.apache.org/protocol#protocol_types

(49) https://redis.io/commands/XADD; https://redis.io/topics/data-types-intro

(50) https://docs.aws.amazon.com/streams/latest/dev/kinesis-low-latency.html

(51) https://www.pubnub.com/blog/amazon-sns-pubnub-differences-pubsub/

(52) https://www.confluent.io/blog/kafka-fastest-messaging-system/

(53) https://softwaremill.com/mqperf/

(54) https://www.rabbitmq.com/reliability.html

(55) https://kafka.apache.org/0110/documentation/streams/core-concepts

(56) https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e

(57) https://activemq.apache.org/

(58) https://activemq.apache.org/performance.html

(59) https://activemq.apache.org/configuring-wire-formats.html see `maxFrameSize = MAX_LONG` - theoretical maximum

(60) https://stackoverflow.com/questions/48645631/is-there-a-max-size-to-activemqs-or-generic-jms-message-size - 1Gb

(61) https://www.cloudamqp.com/blog/what-is-the-message-size-limit-in-rabbitmq.html

(62) https://docs.aws.amazon.com/sns/latest/dg/fifo-message-filtering.html

(63) https://aws.amazon.com/blogs/compute/reducing-custom-code-by-using-advanced-rules-in-amazon-eventbridge/

(64) https://www.amazonaws.cn/en/kinesis/data-streams/features/

(65) https://aws.amazon.com/iot-core/faqs/

(66) https://www.cloudamqp.com/blog/when-to-use-rabbitmq-or-apache-kafka.html

(67) https://www.cloudamqp.com/blog/part4-rabbitmq-for-beginners-exchanges-routing-keys-bindings.html

(68) https://aws.amazon.com/blogs/compute/integrating-amazon-mq-with-other-aws-services-via-apache-camel/ 

(69) https://camel.apache.org/manual/latest/faq/how-does-camel-work-with-activemq.html https://camel.apache.org/components/3.8.x/activemq-component.html

(70) https://redis.io/topics/pubsub

(71) https://stackoverflow.com/questions/59540563/what-are-the-main-differences-between-redis-pub-sub-and-redis-stream

(72) https://stackoverflow.com/questions/60019790/redis-streams-with-persistent-storage

(73) https://redislabs.com/ebook/part-2-core-concepts/chapter-3-commands-in-redis/3-6-publishsubscribe/ 

(74) https://redis.io/commands/xread

(75) https://redis.io/topics/clients - for the given case the max message size needs to be calculate backwards from the 32Mb/min limit.

(76) Assuming an argument of XADD is a string https://redis.io/commands/xadd, infer a max string size from https://redis.io/topics/data-types