# Connecting a service to Kafka

## What this covers

The practical end-to-end picture: what you need to connect a service to a Kafka
topic, how to run Kafka locally, and a worked example where one service produces
messages and another consumes and acknowledges them. This ties together the producer
and consumer topics.

## What a service needs to connect

Three things:

1. A **Kafka client library** for your language. For .NET it is the `Confluent.Kafka`
   NuGet package. There are official or community clients for Java, Python, Go,
   Node.js, and more.
2. The **bootstrap servers**: at least one broker address, for example
   `localhost:9092`.
3. A **topic** to read from or write to. Topics can be created ahead of time, or
   auto-created on first use if the broker allows it (better to create them
   explicitly in real systems).

A producer also needs its durability settings (`acks`), and a consumer also needs a
`group.id` and an offset-commit strategy. Everything else is tuning.

## Running Kafka locally with Docker

The file `docker-compose.yml` in this folder starts a single-broker Kafka using
KRaft (no ZooKeeper). Start it with:

```bash
docker compose up -d
```

The broker is then reachable at `localhost:9092`. Stop it with:

```bash
docker compose down
```

Create a topic once the broker is up (using the tools inside the container):

```bash
docker compose exec kafka \
  /opt/kafka/bin/kafka-topics.sh --create \
  --topic orders --partitions 3 --replication-factor 1 \
  --bootstrap-server localhost:9092
```

Note: replication factor is 1 here because there is only one broker. In production
you would use at least 3 brokers and replication factor 3.

## The end-to-end flow

```
  producer service                 Kafka cluster                consumer service
  ----------------                 -------------                ----------------
  build message        --->   topic "orders" (3 partitions)  --->  poll messages
  send with acks=all          stores + replicates                  process each one
  wait for ack          <---                                       commit offset
```

The producer writes and waits for the acknowledgment it asked for (`acks`). The
broker stores the message in a partition. The consumer, in its own group, polls the
partition, processes each message, and commits the offset to acknowledge progress.
The two services never talk to each other directly; Kafka sits between them.

## Worked example in C#

Install the client in both the producer and consumer projects:

```bash
dotnet add package Confluent.Kafka
```

Producer (writes orders):

```csharp
using Confluent.Kafka;

var config = new ProducerConfig
{
    BootstrapServers = "localhost:9092",
    Acks = Acks.All,
    EnableIdempotence = true
};

using var producer = new ProducerBuilder<string, string>(config).Build();

for (int i = 1; i <= 5; i++)
{
    var msg = new Message<string, string>
    {
        Key = $"customer-{i % 2}",   // same customer key -> same partition
        Value = $"order {i}"
    };
    var result = await producer.ProduceAsync("orders", msg);
    Console.WriteLine($"sent '{msg.Value}' to {result.TopicPartitionOffset}");
}

producer.Flush(TimeSpan.FromSeconds(5)); // make sure buffered messages are sent
```

Consumer (reads and acknowledges orders):

```csharp
using Confluent.Kafka;

var config = new ConsumerConfig
{
    BootstrapServers = "localhost:9092",
    GroupId = "order-processor",
    AutoOffsetReset = AutoOffsetReset.Earliest,
    EnableAutoCommit = false
};

using var consumer = new ConsumerBuilder<string, string>(config).Build();
consumer.Subscribe("orders");

var cancel = new CancellationTokenSource();
Console.CancelKeyPress += (_, e) => { e.Cancel = true; cancel.Cancel(); };

try
{
    while (!cancel.IsCancellationRequested)
    {
        var result = consumer.Consume(cancel.Token);
        if (result is null) continue;

        // do the real work first
        Console.WriteLine(
            $"processing '{result.Message.Value}' " +
            $"from partition {result.Partition.Value} offset {result.Offset.Value}");

        // then acknowledge by committing the offset
        consumer.Commit(result);
    }
}
catch (OperationCanceledException) { }
finally
{
    consumer.Close(); // leaves the group cleanly and triggers a final commit/rebalance
}
```

## Things to get right in a real service

- Always `Close()` the consumer on shutdown so it leaves the group cleanly and the
  group rebalances quickly instead of waiting for a timeout.
- Commit after processing for at-least-once, and make the processing idempotent.
- Handle deserialization errors: a bad message should not crash the whole loop
  forever. A common pattern is to route un-processable messages to a dead-letter
  topic.
- Match the number of consumers in a group to the partition count when you need more
  throughput; extra consumers beyond the partition count stay idle.
- Keep the processing in the loop fast, or the broker may consider the consumer dead
  and rebalance. For slow work, tune `max.poll.interval.ms` or hand work off.

## Documentation

- Kafka quickstart: https://kafka.apache.org/quickstart
- Confluent .NET client overview: https://docs.confluent.io/kafka-clients/dotnet/current/overview.html
- Confluent .NET producer and consumer examples:
  https://github.com/confluentinc/confluent-kafka-dotnet/tree/master/examples
- Running Kafka in KRaft mode: https://kafka.apache.org/documentation/#kraft
