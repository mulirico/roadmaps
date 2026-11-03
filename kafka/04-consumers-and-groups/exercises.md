# Consumers and consumer groups: exercises

Try these without looking at `answers.md`.

## Conceptual

1. Describe the consumer poll loop in four steps.
2. Does Kafka push messages to a consumer, or does the consumer pull? Why does this
   matter for a slow consumer?
3. How does a consumer acknowledge that it has processed messages? Where are these
   acknowledgments stored and what are they keyed by?
4. What does committing offset 100 for a partition mean, and what happens on restart?
5. Explain the difference between auto commit and manual commit. What risk comes with
   auto commit?
6. You process a message, then commit. The consumer crashes after processing but
   before the commit. What happens on restart, and which delivery guarantee is this?
7. You commit first, then process, and crash in between. What happens, and which
   guarantee is this?
8. What is idempotent processing, and why does at-least-once delivery make it
   important? Give an example of an idempotent operation.
9. A topic has 6 partitions. A consumer group has 3 consumers. How are partitions
   assigned? What happens if the group grows to 8 consumers?
10. Two separate consumer groups subscribe to the same topic. Does each group get all
    the messages, or do they share them? Why?
11. What is a rebalance, when does it happen, and why does committing after processing
    matter during one?

## Hands-on thinking

12. You run the minimal consumer, stop it, and start it again with the same
    `group.id`. Where does it resume from? Now you change the `group.id` to a brand
    new value and start again with `AutoOffsetReset=Earliest`. Where does it start,
    and why?
13. Sketch the processing loop for an at-least-once consumer whose job is to insert
    orders into a database. What must be true about the insert so that a re-delivered
    message does not create a duplicate row?
