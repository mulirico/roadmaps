# Topics, partitions, and offsets: exercises

Try these without looking at `answers.md`.

## Conceptual

1. What is the relationship between a topic and a partition? Is a topic a single log?
2. What is an offset? Is offset 42 enough to identify a message, or do you need more?
3. State Kafka's ordering guarantee precisely. Where is order guaranteed and where is
   it not?
4. How does a producer decide which partition a message goes to when the message has
   a key? And when it has no key?
5. You need all events for a given `customerId` to be processed in order. How do you
   achieve that, and what do you use as the message key?
6. What is a replication factor of 3? Describe the roles of leader, follower, and
   in-sync replica.
7. The leader broker for a partition crashes. What happens so the partition stays
   available?
8. What is the difference between deleting messages by retention time and log
   compaction? Give a use case for compaction.

## Hands-on thinking

9. A topic `orders` has 3 partitions. A producer sends 9 messages with no key. Roughly
   how are they distributed, and can you rely on reading all 9 in the exact order they
   were sent? Explain.
10. The same topic now receives messages keyed by `customerId`. Customer A sends 4
    events, customer B sends 2. How many partitions could these land on, and what is
    guaranteed about the order of customer A's events?
