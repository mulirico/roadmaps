# Connecting a service to Kafka: exercises

The hands-on tasks use the `docker-compose.yml` in this folder. You need Docker and
the .NET SDK.

## Conceptual

1. List the three things a service needs to connect to a Kafka topic. What extra does
   a producer need, and what extra does a consumer need?

	A client library, at least one bootstrap server and a topic to read form or to write to. 
	If this service is a producer, it will need an durability setting, i.e., the acknowledgment.
	If the service is a consumer, it will need a group.id and the offset-commit policy.

2. In the end-to-end flow, do the producer service and consumer service talk to each
   other directly? What sits between them?
   
   The broker sits between the producer and the consumer services, as the broker is only part that communicates with the producer, receiving the messages and sending the acknowledgment back to the procucer, and the only part that communicates directly with the consumer that pulls those messages from the broker itself to process and register them.
   
   
3. Why is the replication factor 1 in the local Docker setup, and what would it be in
   production?
   
   The broker that is ran in the Docker setup is a single-broker, that is the reason the replication factor is 1. Whenever in production, you will need more brokers (at least three of them), i.e., a bigger replication factor. 
   
4. Why should a consumer call `Close()` on shutdown instead of just exiting?

	Because you will need to leave the group that the consumer is part of cleanly and trigger the final commit of the offset so no message tracking is lost. The commiting is the case when the processing is set to 'at-least-once' and idempotent.
	

5. What is a dead-letter topic, and what problem does it solve in a consumer loop?

	The problem that writing to a dead-letter topic resolves is failures handling serialization. To write in this specific topic removes logic like a bad message breaking the whole loop forever when a message is being processed.

6. A message takes a long time to process inside the poll loop. What can go wrong with
   the consumer group, and which setting relates to it?

	If the consumer takes a lot of time to process a message, the broker may consider the consumer dead and will rebalance its work and does not send more messages to that consumer. If you need to work upon slow processing, it is important to set the 'max.poll.interval.ms' setting. 


## Hands-on

7. Start the local broker with `docker compose up -d` and create a topic `orders`
   with 3 partitions. Write the exact commands.
   
   `docker compose up -d
	   
    docker compose exec kafka \opt\kafka\bin\kafka-topics.sh --create \
		--topic orders --partitions 3 --replication-factor 1 \
		--bootstra[-server localhost:9092`
   
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
