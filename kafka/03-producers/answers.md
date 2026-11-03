# Producers: answers

## Conceptual

1. A producer builds a message (optional key, a value, optional headers), chooses a
   topic, and sends it. The client library finds the partition leader, batches
   messages for efficiency, serializes the data, and retries on transient failures.

2. Sending is asynchronous so the client can batch messages and keep throughput high
   instead of blocking on every single send. You learn the result through a callback,
   future, or awaited `Task` (for example `ProduceAsync` in the .NET client), which
   reports success with the topic, partition, and offset, or an error.

3. A serializer turns the key and value into bytes, because Kafka stores and transmits
   only bytes and does not understand the content. The consumer must deserialize with
   a matching format, otherwise it cannot reconstruct the original data.

4. `acks=0`: no confirmation, fastest, a message can be lost if the broker fails
   before storing it. `acks=1`: wait for the leader to write it; lost only if the
   leader crashes before followers copy it. `acks=all`: wait for all in-sync replicas;
   strongest, no acknowledged message lost while at least one in-sync replica
   survives. Weakest to strongest: `acks=0`, `acks=1`, `acks=all`.

5. `acks=all` with replication factor 3 and `min.insync.replicas=2`. A write is only
   acknowledged when at least 2 replicas have stored it, so an acknowledged message
   survives the loss of any single broker.

6. If the broker stored the message but its acknowledgment was lost on the network,
   the producer retries and the message is written twice. An idempotent producer
   tags messages so the broker detects and drops the duplicate retry, writing each
   message exactly once per partition.

7. At most once: `acks=0` or retries disabled; messages may be lost, never
   duplicated. At least once: retries on, idempotence off; no acknowledged message
   lost, but duplicates possible. Exactly once: idempotent producer, plus
   transactions when writing atomically across partitions or topics; no loss, no
   duplicates.

8. Only if the partition count changed between the two sends (the key-to-partition
   mapping depends on the current partition count). With a stable partition count the
   same key always maps to the same partition, keeping the two messages ordered. If
   the mapping did change and they split across partitions, their relative order is no
   longer guaranteed.

## Hands-on thinking

9. With `acks=1` the producer already got a success, but the message existed only on
   the leader. When the leader crashes before replication, that message is lost and a
   newly elected leader has no copy of it. With `acks=all` and enough in-sync
   replicas, the producer would not have been acknowledged until the followers also
   had the message, so the new leader would still have it.

10. Turn on: `acks=all` (wait for all in-sync replicas), `enable.idempotence=true` (no
    duplicate retries), retries enabled (the default, so transient failures recover),
    replication factor 3, and `min.insync.replicas=2`. For atomic writes across
    multiple partitions or topics, also use transactions.
