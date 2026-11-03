# Kafka core concepts

## What this covers

The mental model you need before anything else: what Kafka is, the pieces that make
up a Kafka system (broker, cluster, topic, producer, consumer), the idea of the log,
and where Kafka fits compared to a traditional message queue.

## The log is the core idea

Kafka is built around an append-only log. A log here is just an ordered sequence of
records where new records are added only at the end, and each record gets a position
number. Think of a notebook where you only ever write on the next empty line and
never erase.

This one idea explains most of Kafka's behavior:

- Writes are fast because they only append.
- Reads do not remove data. A consumer reads by moving a pointer forward through the
  log.
- Many consumers can read the same log independently, each at its own position.
- Messages stay available for a configured retention period (for example 7 days)
  even after they have been read.

This is the big difference from a classic queue, where a message is typically removed
once it is delivered to one consumer. In Kafka the data stays, and each reader tracks
where it is.

## The main pieces

- **Message (record)**: the unit of data. It has a key (optional), a value (the
  payload, just bytes to Kafka), a timestamp, and optional headers.
- **Topic**: a named log that messages are written to and read from, for example
  `orders` or `user-signups`. This is the main thing a service connects to.
- **Producer**: a client that writes messages to a topic.
- **Consumer**: a client that reads messages from a topic.
- **Broker**: a single Kafka server. It stores data and serves producer and consumer
  requests.
- **Cluster**: a group of brokers working together. Data is spread and replicated
  across brokers for scale and fault tolerance.

A producer and a consumer are usually your own services, running a Kafka client
library. They talk to the brokers over the network.

## How a service finds the cluster

A client is configured with a list of **bootstrap servers**, which is one or more
broker addresses such as `localhost:9092`. The client connects to one of them, learns
the full layout of the cluster (all brokers, which broker holds what), and from then
on talks to the right brokers directly. You do not need to list every broker, just
enough to make the first contact.

## What coordinates the cluster

Older Kafka versions used a separate system called ZooKeeper to track cluster
metadata (which brokers exist, who is the controller, and so on). Modern Kafka uses a
built-in mechanism called KRaft (Kafka Raft) and no longer needs ZooKeeper. For a
beginner the takeaway is: the cluster needs a way to agree on its own state, and newer
Kafka does this on its own.

## Where Kafka fits

Kafka is a good fit when you want to:

- Decouple services: a producer does not know or care who reads the data.
- Let multiple independent consumers read the same events for different purposes
  (one updates a database, another sends emails, another feeds analytics).
- Buffer bursts of traffic and let consumers catch up at their own speed.
- Replay history by resetting a consumer back to an earlier position.

It is less suited to request and response style communication where you need an
immediate answer, which is what HTTP or gRPC are for.

## Documentation

- Introduction: https://kafka.apache.org/intro
- Main concepts and terminology: https://kafka.apache.org/documentation/#intro_concepts_and_terms
- Design, the log: https://kafka.apache.org/documentation/#design
- KRaft overview: https://kafka.apache.org/documentation/#kraft
