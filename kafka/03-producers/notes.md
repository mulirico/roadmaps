# Producers

## What this covers

How a service writes messages to Kafka: what a producer does, the role of the key,
serialization, and the acknowledgment setting (`acks`) that controls how sure you are
that a message was safely stored. This is the write side of "how a service connects
and acknowledges".

## What a producer does

A producer is your service running a Kafka client. It builds a message (optional key,
a value, optional headers), picks a topic, and sends it. The client library handles
the networking: finding the partition leader, batching messages, and retrying on
transient failures.

Sending is asynchronous by default. The client buffers the message, sends it in
batches for efficiency, and gives you back a way to know the result (a callback, a
future, or a `Task` depending on the language).

## The key, again

As covered in the partitions topic, the key decides the partition. From the
producer's point of view:

- Use a key when related messages must stay ordered together (same key, same
  partition, ordered).
- Leave the key empty when you only care about throughput and order does not matter.

## Serialization

Kafka stores bytes. The producer must turn the key and value into bytes with a
**serializer**. Common choices: plain strings (UTF-8), JSON, or a schema-based format
like Avro or Protobuf (often paired with a Schema Registry that enforces a shared
schema between producers and consumers). The consumer must use a matching
deserializer.

## acks: the producer acknowledgment

This is the key producer-side setting for durability. `acks` controls how many broker
replicas must confirm a write before the producer considers it successful.

- `acks=0`: the producer does not wait for any confirmation. Fastest, but a message
  can be lost if the broker fails before storing it. Fire and forget.
- `acks=1`: the producer waits for the partition leader to write the message. Safe if
  the leader survives, but a message can be lost if the leader crashes before
  followers copy it.
- `acks=all` (also written `acks=-1`): the producer waits until all in-sync replicas
  have the message. Strongest durability; no acknowledged message is lost as long as
  at least one in-sync replica survives.

For important data the common choice is `acks=all` combined with a replication factor
of 3 and `min.insync.replicas=2`, which means a write is only acknowledged when at
least 2 replicas have it.

## Retries and idempotence

Networks fail, so producers retry. A naive retry can create duplicates: the message
was actually stored, the acknowledgment was lost, the producer resends. An
**idempotent producer** (enabled with `enable.idempotence=true`, the default in
modern clients) makes the broker detect and drop these duplicate retries, so each
message is written exactly once per partition even with retries. This is a building
block for exactly-once processing.

## Delivery guarantees from the producer side

- At most once: `acks=0` or no retries. You may lose messages, never duplicate.
- At least once: retries on, idempotence off. You never lose an acknowledged message,
  but you may get duplicates.
- Exactly once: idempotent producer, and transactions when writing to multiple
  partitions or topics atomically. No loss, no duplicates.

## A minimal producer (C#, Confluent.Kafka)

```csharp
using Confluent.Kafka;

var config = new ProducerConfig
{
    BootstrapServers = "localhost:9092",
    Acks = Acks.All,
    EnableIdempotence = true
};

using var producer = new ProducerBuilder<string, string>(config).Build();

var result = await producer.ProduceAsync(
    "orders",
    new Message<string, string> { Key = "customer-42", Value = "order placed" });

Console.WriteLine($"wrote to {result.TopicPartitionOffset}");
```

`ProduceAsync` completes when the broker has acknowledged the write according to the
`Acks` setting, and `TopicPartitionOffset` tells you exactly where it landed.

## Documentation

- Producer API: https://kafka.apache.org/documentation/#producerapi
- Producer configuration (acks, retries, idempotence):
  https://kafka.apache.org/documentation/#producerconfigs
- Message delivery semantics:
  https://kafka.apache.org/documentation/#semantics
- Confluent .NET client: https://docs.confluent.io/kafka-clients/dotnet/current/overview.html
