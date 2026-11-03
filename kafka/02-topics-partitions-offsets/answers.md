# Topics, partitions, and offsets: answers

## Conceptual

1. A topic is split into one or more partitions, and each partition is its own
   append-only log. So a topic is not a single log; it is a collection of partitions.

2. An offset is a message's position within a partition, starting at 0 and increasing
   by 1. Offset 42 is not enough on its own, because offsets are unique only inside a
   partition. You need the partition plus the offset (and the topic) to identify a
   message.

3. Kafka guarantees ordering only within a single partition. Messages in one
   partition are read in the order they were written. There is no ordering guarantee
   across different partitions of the same topic.

4. With a key, Kafka hashes the key to pick a partition, so the same key always maps
   to the same partition (while the partition count is unchanged). Without a key, the
   producer balances messages across partitions, giving throughput but no per-key
   ordering.

5. Produce those events with `customerId` as the message key. All messages with the
   same key go to the same partition, and within a partition order is guaranteed, so
   the customer's events stay ordered.

6. Replication factor 3 means each partition has 3 copies on 3 brokers. One copy is
   the leader and handles all reads and writes. The other two are followers that copy
   the leader. Followers that are fully caught up are the in-sync replicas (ISR).

7. One of the in-sync follower replicas is promoted to leader. Clients then send reads
   and writes to the new leader, so the partition stays available with its data
   intact.

8. Retention by time (or size) deletes old messages once they age past the limit, no
   matter their key. Log compaction instead keeps the latest message per key and
   removes older ones for that key. Compaction is useful when the topic represents
   current state, for example the latest known value for each entity (a changelog or
   a lookup table).

## Hands-on thinking

9. With no key, the 9 messages are spread roughly evenly across the 3 partitions
   (about 3 each). You cannot rely on reading all 9 in the exact send order, because
   ordering holds only within each partition, not across the three.

10. Customer A's 4 events all hash to one partition, and customer B's 2 events hash to
    one partition; these may be the same partition or different ones (at most 2
    partitions are involved, possibly just 1). What is guaranteed: customer A's 4
    events are in order relative to each other, because they share a key and therefore
    a partition.
