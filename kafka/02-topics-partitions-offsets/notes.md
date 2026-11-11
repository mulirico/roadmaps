# Topics, partitions, and offsets

## What this covers

How Kafka actually organizes the data inside a topic. This is the part beginners most
often get confused by, and it is where most interview questions about ordering,
scaling, and parallelism come from.

## A topic is split into partitions

A topic is not a single log. It is split into one or more **partitions**, and each
partition is its own append-only log. When you create a topic you choose how many
partitions it has, for example `orders` with 6 partitions.

Each partition:

- Is an ordered sequence of messages.
- Lives on a broker (and is replicated to others for safety).
- Is read and written independently of the other partitions.

Partitions are the unit of parallelism in Kafka. More partitions means more
consumers can read the topic at the same time (more on that in the consumers topic).

## Offsets

Within a partition, every message has an **offset**: a number that marks its position,
starting at 0 and increasing by 1 for each new message. The offset is unique only
inside its partition, so "offset 42" only means something when you also say which
partition.

A consumer's progress is just the offset it has reached in each partition. Reading is
moving that offset forward.

## Ordering guarantee

This is a common interview point. Kafka guarantees ordering **only within a single
partition**, not across the whole topic. Messages in partition 0 are read in the
order they were written, but there is no global order across partitions 0, 1, and 2.

So if the order of certain messages matters (for example all events for one customer),
you must make sure those messages land in the same partition.

## How a message is assigned to a partition

When a producer sends a message, the partition is chosen like this:

- If the message has a **key**, Kafka hashes the key and uses it to pick a partition.
  The same key always goes to the same partition (as long as the partition count does
  not change). This is how you keep related messages ordered together.
- If there is **no key**, the producer spreads messages across partitions (balanced),
  so you get throughput but no per-key ordering.

Example: using `customerId` as the key means all events for a given customer go to one
partition and stay in order, while different customers spread across partitions.

## Replication, leaders, and followers

Each partition is replicated across several brokers for fault tolerance. The
**replication factor** says how many copies exist (for example 3).

- One replica is the **leader**. All reads and writes for that partition go through
  the leader.
- The others are **followers**, which copy the leader's data.
- Followers that are caught up are called the **in-sync replicas (ISR)**.
- If the leader broker fails, one of the in-sync followers is promoted to leader, so
  the partition stays available.

## Retention

A topic keeps messages for a configured retention, by time (for example 7 days) or by
size. After that, old segments are deleted. There is also **log compaction**, which
keeps only the latest message per key instead of deleting by age, useful when the
topic represents the current state of things rather than a stream of events.

## Documentation

- Topics and logs: https://kafka.apache.org/documentation/#intro_concepts_and_terms
- Partitioning and the producer: https://kafka.apache.org/documentation/#producerconfigs
- Replication: https://kafka.apache.org/documentation/#replication
- Log compaction: https://kafka.apache.org/documentation/#compaction
