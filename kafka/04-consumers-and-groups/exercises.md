# Consumers and consumer groups: exercises

Try these without looking at `answers.md`.

## Conceptual

1. Describe the consumer poll loop in four steps.
   
   Call poll to fetch a batch of messages. Then, process each message. After, it commits the offset to keep track of its readings. And repeat this whole process.
   
2. Does Kafka push messages to a consumer, or does the consumer pull? Why does this
   matter for a slow consumer?
   
   In Kafka, only the consumer pulls batch of messages from the Kafka topic. This way, a slow consumer is not affect by an uncapable amount of message, as it pulls whenever it's ready to process.
   
3. How does a consumer acknowledge that it has processed messages? Where are these
   acknowledgments stored and what are they keyed by?
   
   The acknowledgments are registered as commits of the offset that is created inside an internot topic in kafka service, this is how a Kafka topic keeps in track to what was processed or not. 
   
4. What does committing offset 100 for a partition mean, and what happens on restart?
   
   

5. Explain the difference between auto commit and manual commit. What risk comes with
   auto commit?
   
   With auto commit, the topic writes the commit of the offset by a time rule set (auto.commit.interval.ms), which is risk because if the message processing was not completed and the commit already wrote its register in the internal database, that message could be lost, if the consumer crashs before the completion of the processing step. However, nothing is registered in the offset commiting with manual commits, without the processing being really completed, as it usually orders the steps in the following way: consumer fetch message batches, process each message and commits directly after processing the message to keep track of the offset.
   
6. You process a message, then commit. The consumer crashes after processing but
   before the commit. What happens on restart, and which delivery guarantee is this?
   
   If the idempotent spec is set, the consumer will pulls the message again but no duplicated will be generated as the message handling is idempotent. 
   However, if operations is not idempotent, a duplicate of the processing could be generated. 
   This is the at least once delivery guarantee.
   
7. You commit first, then process, and crash in between. What happens, and which
   guarantee is this?
   
   If you commit and then process and there is a crash between these process, you could lose track of a message. No duplicates will be generated but you could lose the effectively processing of a message. This is the at most once delivery guarantee.
   
8. What is idempotent processing, and why does at-least-once delivery make it
   important? Give an example of an idempotent operation.
   
   Idempotent processing is to make sure the activity is done exactly once, even if the command to process it more than once is emitted. For example, to place an order: if a consumer pulls the message to process the place of an order and it crashes before registering the commit that keeps track of that activity, it will try to execute again that placing. If the activity is not idempotent, it will place the order again and cause a problem of two items being placed for the same order, for example.
   
9. A topic has 6 partitions. A consumer group has 3 consumers. How are partitions
   assigned? What happens if the group grows to 8 consumers?
   
   In this scenario, each consumer will read 2 messages as they are equally distributed. If you add 8 consumers, each consumer will read a message and there will be two obsolete consumers that will stand on hold, as two consumers of the group does not consume the same message twice.
   
10. Two separate consumer groups subscribe to the same topic. Does each group get all
    the messages, or do they share them? Why?
	
	The consumer groups will receive the same group of messages (full messages stream) as they are independent. 
	
11. What is a rebalance, when does it happen, and why does committing after processing
    matter during one?
	
	Rebalance happens whenever a consumer enters or leaves the consumer group. This way, the consumers pauses the pull from the kafka topic and continues after the rebalancing upon the new amount of consumers of that group. It is important commiting so the consumer group can easily return to the same spot it paused its activities.

## Hands-on thinking

12. You run the minimal consumer, stop it, and start it again with the same
    `group.id`. Where does it resume from? Now you change the `group.id` to a brand
    new value and start again with `AutoOffsetReset=Earliest`. Where does it start,
    and why?
	
	It start from the latest commited message now on. When you change the offset reset, it will start the processing from the most early message on that kafka topic.
	
13. Sketch the processing loop for an at-least-once consumer whose job is to insert
    orders into a database. What must be true about the insert so that a re-delivered
    message does not create a duplicate row?
	
	It is important to make sure the activity of the database changes are idempotent. This way, the at least once delivery guarantee does not produces duplicates registers on the database even if the process crash after processing and before commiting the changes.
	
