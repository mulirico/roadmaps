# Producers: exercises

Try these without looking at `answers.md`.

## Conceptual

1. What does a producer do, and what work does the client library handle for you when
   you send a message?
   The producer is the service that access and communicates with the broker, kafka topic. The client library usually handle the partition selection, open and close communication from and to the topic and do the serialization of the message. Is the kafka client. 

2. Why is producer sending asynchronous by default, and how do you find out whether a
   send succeeded?
   The sending is asynchronous  by default so the client can buffer the message, sent it and have a result from the operation, such as a callback or a task. You can make sure if the message succeeded by registering the acknowledges from the topic. 
   
   
3. Kafka stores bytes. What is a serializer, and why must the consumer agree with the
   producer on the format?
   A serializer is the client part/producer part that transform a message to a serialized version that can be handled by the broker in the key value pairs. It can be serialized to plain JSON or AVRO/Protobuf format.
   
4. Explain `acks=0`, `acks=1`, and `acks=all`. Rank them from weakest to strongest
   durability and say what can be lost in each.
   Acknowledges 0 is a fire-and-forget when no parts of the producer waits on response/task from the kafka topic. When the broker fails before storing it, a message is lost and there is no track of what happened.
   The acknowledge 1 is when the producer waits just for the leader partition to write the message. If the leader partition crashes, there is a lost of the message track.
   Acknowledge -1 is when you write the message on all partitions and wait for the response from all them. So all in-sync replicas has the log of the message consumed.
   
5. For important financial data, what combination of `acks`, replication factor, and
   `min.insync.replicas` is commonly recommended, and what does that combination
   guarantee?
   For important financial data you can choose reliable combination of acknowledge waiting for all replicas and increase the min.insync.replicas to at least three. This way, the source of truth turns out more consistent as you would have to crash all partitions to lose data. So this scenario is rare and you persist data that tracks messagem streams consistent.
   
6. How can a retry cause a duplicate message, and what does an idempotent producer do
   about it?
   
   If the idempotence is not set and you have a acknowledge of 1 and you also lose data from the partition that kept track of that message stream, you may get duplicated messages since the producer will retry again and the broker or specially the partition has no track of that message. The idempotent producer will permit the broker to detect and drop the duplicated retry.
   
7. Define the three delivery guarantees from the producer side (at most once, at least
   once, exactly once) and the settings that produce each.
   
   What differenciate the types of guarantees from the producer side is the acknowledge on each activity made by it. It could guarantee a at most once send on the message, which means no acknowledge of it, which could cause lost of track of the message but could never generate a duplicate as the message send just triggers once.
   If the producer guarantees the message send in at least once partitions, it will track the returned message that was registered at least in the main partition. However, if that partition crashes and you lose the records on it, the producer could think the message was not sent and send it again, producing some duplicated messages (if the idempotence config was not set). 
   At last, the producer could guarantee the message delivery in its side by configuring the idempotent and make sure the transaction is write in multiple partition (all partitions, ack=-1). This way, if the main or other partition crashes, there is another reliable source to track if a message was sent and completed. The idempotent config also make sure in the producer side that it does not send another message again.
   
8. Two messages are produced with the same key to a topic with multiple partitions.
   Can they end up in different partitions? What does that mean for their order?
   
   The key passed is what the producer will take to send to the broker, so it can allocate the sequenced message in the same partition. This way it will be organized and ordered inside that partition. If the key is sent on the message, the broker will order them in the partition so they can be processed subsequentially. 

## Hands-on thinking

9. You send a message with `acks=1`. The leader writes it and acknowledges, then the
   leader broker crashes before any follower copied the message. What happened to the
   message, and how would `acks=all` have changed the outcome?
   
   If you had sent the message with acknowledge 1, the producer will not keep track of the processing of the message. So, either if the message was indeed processed before the crash, it will just lost track of the whole process. This is a fire and forget model.
   However, if had sent the message to acknowledge for all partition, either if the leader broker crashes, the subsequent leader will have track of the message processing.
   
10. Sketch the configuration for a producer that must never lose an acknowledged
    message and must not write duplicates. List the settings you would turn on.
	
	acks=all or acks=-1
	enable.idempotence=true
	min,insync.replicas=2
	```csharp
	using Confluent.Kafka;
	
	var config = new ProducerConfig 
	{
		Bootstrapservers = "localhost:9092",
		Acks = Acks.All,
		EnableIdempotence = true
	}
	
	using var producer = new ProducerBuilder<string, string>(config).Build();
	
	var result = await producer.ProduceAsync(
		"orders",
			new Message<string, string> { Key = "customer-123", Value = "order placed"
		};)
	```
