# Producers: exercises

Try these without looking at `answers.md`.

## Conceptual

1. What does a producer do, and what work does the client library handle for you when
   you send a message?
2. Why is producer sending asynchronous by default, and how do you find out whether a
   send succeeded?
3. Kafka stores bytes. What is a serializer, and why must the consumer agree with the
   producer on the format?
4. Explain `acks=0`, `acks=1`, and `acks=all`. Rank them from weakest to strongest
   durability and say what can be lost in each.
5. For important financial data, what combination of `acks`, replication factor, and
   `min.insync.replicas` is commonly recommended, and what does that combination
   guarantee?
6. How can a retry cause a duplicate message, and what does an idempotent producer do
   about it?
7. Define the three delivery guarantees from the producer side (at most once, at least
   once, exactly once) and the settings that produce each.
8. Two messages are produced with the same key to a topic with multiple partitions.
   Can they end up in different partitions? What does that mean for their order?

## Hands-on thinking

9. You send a message with `acks=1`. The leader writes it and acknowledges, then the
   leader broker crashes before any follower copied the message. What happened to the
   message, and how would `acks=all` have changed the outcome?
10. Sketch the configuration for a producer that must never lose an acknowledged
    message and must not write duplicates. List the settings you would turn on.
