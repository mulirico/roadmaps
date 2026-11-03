# Kafka core concepts: answers

## Conceptual

1. The log is an ordered, append-only sequence of records: new records go only at the
   end, and each gets a position number. Two consequences: writes are fast because
   they only append, and reading does not delete data, so many consumers can read the
   same log independently at their own positions.

2. A classic queue usually removes a message once it has been delivered to a
   consumer. Kafka keeps the message for a configured retention period regardless of
   who has read it; each consumer just tracks its own position in the log.

3. Broker: a single Kafka server that stores data and serves requests. Cluster: a
   group of brokers working together. Topic: a named log that messages are written
   to and read from. Producer: a client that writes messages. Consumer: a client
   that reads messages.

4. Bootstrap servers are one or more broker addresses the client uses to make first
   contact. After connecting to one, the client learns the full cluster layout and
   then talks to the right brokers directly, so you only need enough addresses to
   reach the cluster once.

5. The value is the payload. To Kafka the value (and the key) are just bytes; Kafka
   does not interpret the contents. Serialization and deserialization are the
   client's responsibility.

6. ZooKeeper tracked cluster metadata such as which brokers exist and which one is
   the controller. Modern Kafka replaces it with the built-in KRaft (Kafka Raft)
   mechanism, so a separate ZooKeeper is no longer needed.

7. Good fits (any two): decoupling producers from consumers, letting multiple
   independent consumers read the same events, buffering traffic bursts, replaying
   history by resetting a consumer. Better with HTTP or gRPC: when you need an
   immediate response to a request, such as a synchronous lookup or a command that
   must return a result to the caller.

8. Because reads do not consume the data and each consumer tracks its own position,
   all three services can read the full `orders` stream independently. One can update
   a database, another can send emails, another can feed analytics, without
   interfering with each other.

## Hands-on thinking

9. A reasonable sketch: the two producers both send messages to the topic
   `payments`. The topic's data physically lives on the brokers in the cluster
   (spread and replicated across the three brokers). The two consumers each read from
   the brokers, each keeping its own position. The key point to get right: messages
   are stored on the brokers, not on the producers or consumers.

10. No, reading does not remove the message. With 7-day retention the message stays
    available to any consumer until it ages past 7 days (or until a size or compaction
    policy removes it), independent of whether it has been read.
