# Connecting a service to Kafka: answers

## Conceptual

1. A Kafka client library, the bootstrap servers (at least one broker address), and a
   topic. A producer additionally needs its durability setting (`acks`). A consumer
   additionally needs a `group.id` and an offset-commit strategy (auto or manual).

2. No, they do not talk directly. Kafka sits between them: the producer writes to a
   topic on the brokers, and the consumer reads from that topic. The two services are
   decoupled and do not need to know about each other.

3. There is only one broker in the local setup, so there is nowhere to place extra
   replicas; replication factor can only be 1. In production you run at least 3
   brokers and use replication factor 3 so a partition survives a broker failure.

4. `Close()` makes the consumer leave the group cleanly and commit as configured.
   Without it, the group waits for a session timeout before deciding the consumer is
   gone, which delays the rebalance and the handover of its partitions to other
   consumers.

5. A dead-letter topic is a separate topic where a consumer sends messages it cannot
   process (for example a message that fails to deserialize or that repeatedly
   errors). It keeps one bad message from blocking the loop forever, while preserving
   the message for later inspection or reprocessing.

6. If processing takes longer than `max.poll.interval.ms`, the broker considers the
   consumer dead and triggers a rebalance, reassigning its partitions. The work may
   then be redone elsewhere and the consumer can be kicked from the group. Either
   speed up processing, raise `max.poll.interval.ms`, or hand slow work off the poll
   thread.

## Hands-on

7. Commands:

```bash
docker compose up -d
docker compose exec kafka \
  /opt/kafka/bin/kafka-topics.sh --create \
  --topic orders --partitions 3 --replication-factor 1 \
  --bootstrap-server localhost:9092
```

8. The producer prints a line per message with its `TopicPartitionOffset`. The keys
   alternate `customer-0` and `customer-1` (from `i % 2`), so all `customer-0`
   messages share one partition and all `customer-1` messages share another. Messages
   with the same key land on the same partition because Kafka hashes the key.

9. On the second run the consumer does not re-read the first 5. With the same
   `group.id` it has committed offsets for those messages, so it resumes after them
   and reads only the 5 new ones. Progress is remembered per group in
   `__consumer_offsets`.

10. With two consumers in the group, the 3 partitions are split, for example 2 to one
    consumer and 1 to the other. With three consumers, each reads 1 partition. With a
    fourth consumer there are only 3 partitions, so the fourth is assigned nothing and
    sits idle until a partition becomes free (for example if another consumer leaves).

11. A new `group.id` has no committed offsets, so with `AutoOffsetReset=Earliest` it
    reads the topic from the beginning and sees all messages ever produced (still
    within retention). This differs from reusing the old group id, which would resume
    from the old group's committed position and skip what it already processed.

12. Committing before processing turns it into at-most-once. If the process is killed
    after the commit but before the work is done, those messages are marked as
    handled and are skipped on restart, so they are lost. The trade-off is no
    duplicates but possible message loss, the opposite of the at-least-once default.
