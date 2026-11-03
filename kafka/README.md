# Apache Kafka study notes

Personal study material for Apache Kafka, written for a complete beginner and
oriented toward the concepts and questions that come up in interviews. The focus is
on understanding how a service connects to Kafka, reads from and writes to topics,
and acknowledges the messages it processes.

Each topic is a folder with three files:

- `notes.md`: a short summary of the concepts and links to the official
  documentation.
- `exercises.md`: practice questions and small hands-on tasks.
- `answers.md`: the answer key for that topic, kept separate so you can test
  yourself first.

## What Kafka is, in one paragraph

Apache Kafka is a distributed event streaming platform. At its core it is an
append-only log: producers write messages to the end of a topic, and consumers read
those messages at their own pace by tracking their position in the log. Kafka keeps
messages for a configured time even after they are read, so many independent services
can read the same stream without interfering with each other. This makes it a common
backbone for decoupling services, event-driven systems, and data pipelines.

## Topics

1. [Core concepts](01-core-concepts/notes.md): brokers, clusters, the log, where
   Kafka fits.
2. [Topics, partitions, and offsets](02-topics-partitions-offsets/notes.md): how
   data is organized and ordered.
3. [Producers](03-producers/notes.md): writing messages, keys, and delivery
   acknowledgments (`acks`).
4. [Consumers and consumer groups](04-consumers-and-groups/notes.md): reading
   messages, committing offsets, delivery guarantees, rebalancing.
5. [Connecting a service to Kafka](05-connecting-a-service/notes.md): a practical
   walkthrough with runnable examples.

## Suggested study flow

1. Read `notes.md` for the topic.
2. Open the documentation links and read the parts that are new to you.
3. Do the exercises in `exercises.md` without looking at the answers.
4. Check your work against `answers.md` and review anything you missed.

## Running Kafka locally

Topic 5 includes a `docker compose` setup so you can run a single-broker Kafka on
your machine and try producing and consuming messages. You need Docker installed.

## Official entry points

- Apache Kafka documentation: https://kafka.apache.org/documentation/
- Kafka introduction: https://kafka.apache.org/intro
- Kafka design: https://kafka.apache.org/documentation/#design
