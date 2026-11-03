# Consumers and consumer groups

## What this covers

The read side, and the part you specifically asked about: how a service reads from a
topic and how it acknowledges the messages it has processed. In Kafka that
acknowledgment is called **committing an offset**. This topic also covers consumer
groups, rebalancing, and the delivery guarantees that follow from how you commit.

## How a consumer reads

A consumer is your service running a Kafka client. It subscribes to one or more
topics and then polls in a loop:

1. Call poll (or the client's equivalent) to fetch a batch of messages.
2. Process each message (write to a database, call an API, whatever the job is).
3. Commit the offset to record how far it has read.
4. Repeat.

The consumer pulls from Kafka. Kafka does not push to the consumer. This is why a
slow consumer does not get overwhelmed; it simply asks for more when it is ready.

## The offset is the acknowledgment

There is no per-message "ack" sent back to the broker like in some queues. Instead,
the consumer tracks its position with the **committed offset**: the offset up to
which it has processed messages in each partition. Committing offset 100 for a
partition means "I have handled everything through offset 99, start me at 100 next
time".

Committed offsets are stored by Kafka in an internal topic called
`__consumer_offsets`, keyed by the consumer group. So if the consumer restarts, it
resumes from the last committed offset instead of from the beginning.

This is the answer to "how does a service acknowledge changes": it commits the offset
after it has successfully processed the messages.

## Auto commit vs manual commit

- **Auto commit** (`enable.auto.commit=true`): the client periodically commits the
  latest fetched offset in the background (every `auto.commit.interval.ms`). Simple,
  but it can commit before you have actually finished processing, which risks losing
  messages on a crash. It can also re-deliver if a crash happens between a commit and
  finishing work.
- **Manual commit** (`enable.auto.commit=false`): you call commit yourself, after you
  have done the work. This gives you control over the delivery guarantee. The usual
  pattern for safe processing is: process the batch, then commit.

For anything beyond a demo, manual commit after processing is the common choice.

## Delivery guarantees, from the commit order

The guarantee you get depends on whether you commit before or after processing:

- **At least once** (most common): process first, then commit. If the consumer
  crashes after processing but before committing, those messages are processed again
  on restart. No message is lost, but you may see a message twice, so your processing
  should be idempotent (safe to run twice for the same message).
- **At most once**: commit first, then process. If it crashes after committing but
  before processing, those messages are skipped. No duplicates, but you can lose
  messages.
- **Exactly once**: achieved with Kafka transactions (the read-process-write pattern
  inside a transaction) or by making your writes idempotent so duplicates have no
  effect. Harder to set up; often "at least once plus idempotent processing" is the
  pragmatic target.

Idempotent processing means an operation produces the same result whether it runs
once or several times, for example an upsert keyed by an id, so a re-delivered
message does not cause a second charge or a duplicate row.

## Consumer groups: scaling out

A **consumer group** is a set of consumers that share a `group.id` and split the work
of reading a topic. Kafka assigns each partition to exactly one consumer in the group.

- If a topic has 6 partitions and the group has 3 consumers, each consumer reads 2
  partitions.
- Add a fourth consumer and each of four reads from the group's share; the partitions
  are redistributed.
- If the group has more consumers than partitions, the extra consumers sit idle,
  because a partition is never read by two consumers in the same group at once.

This is how you scale consumption: more partitions allow more consumers working in
parallel, up to one consumer per partition.

Different groups are independent. Two separate groups each get the full stream, which
is how you let, say, a billing service and an analytics service both read all
`orders` events.

## Rebalancing

When a consumer joins or leaves the group (startup, crash, scale-up), Kafka
**rebalances**: it reassigns partitions among the current members. During a rebalance
consumption pauses briefly. Committed offsets matter here: after a rebalance a
consumer may be handed a partition another consumer was reading, and it resumes from
the last committed offset, which is another reason to commit after processing.

## A minimal consumer (C#, Confluent.Kafka)

```csharp
using Confluent.Kafka;

var config = new ConsumerConfig
{
    BootstrapServers = "localhost:9092",
    GroupId = "order-processor",
    AutoOffsetReset = AutoOffsetReset.Earliest, // where to start with no committed offset
    EnableAutoCommit = false                    // we commit manually after processing
};

using var consumer = new ConsumerBuilder<string, string>(config).Build();
consumer.Subscribe("orders");

while (true)
{
    var result = consumer.Consume(TimeSpan.FromSeconds(1));
    if (result is null) continue;

    Process(result.Message.Value); // do the work first

    consumer.Commit(result);       // then acknowledge by committing the offset
}
```

`AutoOffsetReset` only applies when the group has no committed offset yet (first
run): `Earliest` starts at the beginning of the partition, `Latest` starts at new
messages only.

## Documentation

- Consumer API: https://kafka.apache.org/documentation/#consumerapi
- Consumer configuration (group.id, auto commit, offset reset):
  https://kafka.apache.org/documentation/#consumerconfigs
- Consumer groups and rebalancing:
  https://kafka.apache.org/documentation/#intro_consumers
- Message delivery semantics:
  https://kafka.apache.org/documentation/#semantics
