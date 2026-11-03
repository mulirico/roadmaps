# Connecting a service to Kafka: exercises

The hands-on tasks use the `docker-compose.yml` in this folder. You need Docker and
the .NET SDK.

## Conceptual

1. List the three things a service needs to connect to a Kafka topic. What extra does
   a producer need, and what extra does a consumer need?
2. In the end-to-end flow, do the producer service and consumer service talk to each
   other directly? What sits between them?
3. Why is the replication factor 1 in the local Docker setup, and what would it be in
   production?
4. Why should a consumer call `Close()` on shutdown instead of just exiting?
5. What is a dead-letter topic, and what problem does it solve in a consumer loop?
6. A message takes a long time to process inside the poll loop. What can go wrong with
   the consumer group, and which setting relates to it?

## Hands-on

7. Start the local broker with `docker compose up -d` and create a topic `orders`
   with 3 partitions. Write the exact commands.
8. Build the producer from the notes, run it, and confirm from its output which
   partitions and offsets the 5 messages landed on. Which messages shared a
   partition, and why?
9. Build the consumer from the notes with `group.id=order-processor`, run it, and
   confirm it reads all 5 messages. Stop it, run the producer again to send 5 more,
   then start the consumer again. Does it re-read the first 5? Explain.
10. Start a second instance of the consumer with the same `group.id` while the first
    is running. Observe how the 3 partitions are split between the two instances. Now
    start a third and a fourth instance. What happens to the fourth?
11. Change the consumer's `group.id` to a new value and start it with
    `AutoOffsetReset=Earliest`. What does it read, and why is this different from
    reusing the old group id?
12. Modify the consumer to commit before processing instead of after. Describe (or
    simulate by killing the process mid-batch) how this changes the delivery
    guarantee.
