# Consumers and consumer groups: answers

## Conceptual

1. Poll for a batch of messages; process each message; commit the offset to record
   progress; repeat.

2. The consumer pulls: it asks Kafka for messages when it is ready. A slow consumer
   is not overwhelmed, because it simply fetches more only when it has capacity,
   instead of having messages pushed at it faster than it can handle.

3. It acknowledges by committing an offset: the position up to which it has processed
   each partition. Committed offsets are stored by Kafka in the internal topic
   `__consumer_offsets`, keyed by the consumer group (and topic and partition).

4. It means the consumer has handled everything through offset 99 and should resume at
   offset 100. On restart the consumer reads its committed offset and continues from
   there instead of from the beginning.

5. Auto commit lets the client commit the latest fetched offset periodically in the
   background. Manual commit means you call commit yourself after processing. The risk
   with auto commit is that it may commit before you finished (or even started)
   processing, so a crash can skip messages, or it can re-process depending on timing.

6. On restart the consumer resumes from the last committed offset, which is before
   those messages, so it processes them again. This is at-least-once delivery: no
   loss, but possible duplicates.

7. The messages are marked as committed but never actually processed, so on restart
   the consumer skips past them. This is at-most-once delivery: no duplicates, but
   possible loss.

8. Idempotent processing means running the same operation once or several times
   produces the same result. It matters because at-least-once delivery can hand you
   the same message more than once, and idempotent processing makes those duplicates
   harmless. Example: an upsert keyed by order id, so reprocessing the same order
   updates the same row instead of inserting a second one.

9. Each of the 3 consumers is assigned 2 of the 6 partitions (one partition is read by
   exactly one consumer in the group). If the group grows to 8 consumers, only 6 can
   be assigned a partition; the other 2 sit idle, because a partition is never shared
   by two consumers in the same group.

10. Each group gets all the messages. Consumer groups are independent, so the topic's
    full stream is delivered to each group separately. This is how different services
    (for example billing and analytics) can each consume every event.

11. A rebalance is Kafka reassigning partitions among the group's members. It happens
    when a consumer joins or leaves (startup, crash, scaling). Consumption pauses
    briefly. Committing after processing matters because a partition may move to
    another consumer, which resumes from the last committed offset; if you had
    committed before processing, work could be skipped.

## Hands-on thinking

12. With the same `group.id` it resumes from the last committed offset, continuing
    where it left off. With a brand new `group.id` there is no committed offset, so
    `AutoOffsetReset=Earliest` makes it start from the beginning of each partition.
    `AutoOffsetReset` only applies when no committed offset exists for the group.

13. Loop: poll a batch, insert each order into the database, then commit the offset.
    For re-delivery to be safe the insert must be idempotent, for example an upsert
    (insert or update) keyed by the order id, or an insert guarded by a unique
    constraint on the order id so a duplicate is ignored rather than creating a second
    row. Committing only after the successful insert keeps it at-least-once.
