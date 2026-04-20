## Goals: ##
1. Create a topic with multiple partitions on the 3-broker cluster
2. Use a script to continuously produce messages to the topic
3. Explore how Kafka distributes partitions across consumers in a group
4. Observe consumer group rebalancing as consumers join and leave
5. Use kafka-consumer-groups.sh to inspect group state, offsets and lag

## Steps: ##

### Step 1: Ensure the 3-broker cluster is running ###
If the cluster from Lab 3 is not already running, start it:
```
docker-compose up -d
```

Verify the brokers are running:
```
docker-compose top
```

### Step 2: Create a topic ###
Create a topic called `activity` with 6 partitions and a replication factor of 3:
```
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --partitions 6 \
  --replication-factor 3 \
  --topic activity
```

Verify the topic and note the partition/leader/replica layout:
```
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic activity
```

Note how Kafka has distributed partition leadership fairly across the 3 brokers.

### Step 3: Start a continuous producer script ###
In a terminal window, start the following script to produce a new message every 5 seconds:
```
while true; do
  KEY="user-$((RANDOM % 6 + 1))"
  VALUE="event-$((RANDOM % 1000))"
  echo "$KEY:$VALUE" | kafka-console-producer.sh \
    --broker-list localhost:9092 \
    --topic activity \
    --property "parse.key=true" \
    --property "key.separator=:"
  sleep 5
done
```

Leave this running in the background. Press `Ctrl-C` at any time to stop it.

The script cycles through user-1 to user-6 as keys, so messages spread across partitions in a predictable way.

### Step 4: Start a single consumer in a named group ###
Open a new terminal window and start a consumer in a group called `analytics`:
```
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic activity \
  --group analytics \
  --from-beginning \
  --property print.key=true \
  --property print.value=true \
  --property print.partition=true \
  --property key.separator=" : "
```

Observe that this single consumer is receiving messages from **all 6 partitions**.

### Step 5: Add a second consumer to the same group ###
Open a new terminal window and start a second consumer in the same `analytics` group:
```
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic activity \
  --group analytics \
  --property print.key=true \
  --property print.value=true \
  --property print.partition=true \
  --property key.separator=" : "
```

Watch both consumer windows. You should see Kafka **rebalance** — each consumer will now handle 3 partitions each. New messages will no longer all appear in the first consumer window.

### Step 6: Add a third consumer to the same group ###
Open a fourth terminal window and start a third consumer in the `analytics` group using the same command as Step 5.

Observe the rebalance again. With 6 partitions and 3 consumers, each consumer should now own **2 partitions**.

### Step 7: Inspect the consumer group with kafka-consumer-groups.sh ###
Open a new terminal window and explore the consumer groups tool:
```
kafka-consumer-groups.sh
```

List all consumer groups on the cluster:
```
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Describe the `analytics` group in detail:
```
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --group analytics
```

Examine the output carefully. For each partition note:
- **CONSUMER-ID** — which consumer instance owns this partition
- **CURRENT-OFFSET** — the last offset committed by the consumer
- **LOG-END-OFFSET** — the latest offset written to the partition
- **LAG** — the difference between the two; how far behind the consumer is

### Step 8: Explore the __consumer_offsets topic ###
Kafka stores all consumer group offset commits internally in a special topic called `__consumer_offsets`. This topic is normally hidden but can be consumed directly from the CLI.

By default this topic is not shown in topic listings. Verify it exists by listing all topics including internal ones:
```
kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list \
  --exclude-internal
```

Then run without the flag to see it appear:
```
kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Describe the topic to see how it is structured — note that it has 50 partitions by default:
```
kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --topic __consumer_offsets
```

Now consume from the topic using the built-in `kafka.coordinator.group.GroupMetadataManager$OffsetsMessageFormatter` to decode the binary content into a readable format:
```
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic __consumer_offsets \
  --from-beginning \
  --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" \
  --max-messages 20
```

Each line represents an offset commit from a consumer group. You should be able to see entries for your `analytics` group, showing which partition offset has been committed. The format is:

```
[group-id, topic, partition]::OffsetAndMetadata[offset=N, ...]
```

With your producer script still running, watch how new entries appear in `__consumer_offsets` as your consumers commit their offsets.

### Step 9: Observe a rebalance by stopping a consumer ###
Stop one of your consumer processes with `Ctrl-C`.

Wait a few seconds, then re-run the describe command from Step 7. Notice:
- The group has rebalanced — the stopped consumer's partitions have been redistributed
- The remaining consumers now own more partitions each
- The LAG for the reassigned partitions may have increased while the rebalance was in progress

### Step 10: Add a consumer to a second independent group ###
Open a new terminal window and start a consumer in a **different** group called `reporting`:
```
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic activity \
  --group reporting \
  --from-beginning \
  --property print.key=true \
  --property print.value=true \
  --property print.partition=true \
  --property key.separator=" : "
```

Observe that this consumer receives **all messages independently** of the `analytics` group — it has its own offset tracking and processes every record from the beginning, regardless of what `analytics` has already consumed.

List the groups again to see both:
```
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list
```

### Optional Extras ###

View the options for the kafka-consumer-groups.sh tool.

What happens to the LAG reported for the `analytics` group if you stop all consumers in the group and let the producer continue running for a minute? Start a consumer again — does it catch up?

What happens if you start more consumers than there are partitions? Start a 7th consumer in the `analytics` group and observe what the describe output shows for that consumer.

Can you use `kafka-consumer-groups.sh --reset-offsets` to rewind the `analytics` group back to the beginning of the topic and reprocess all messages?

####################
