## Goals: ##
1. Explore the parameters available for kafka-topics.sh
2. Create a topic with multiple partitions
3. Produce messages with keys using the console producer
4. Consume messages using the console consumer
5. Delete a topic from the cli

## Steps: ##

### Step 1: Start a kafka broker (if not already running) ###
```
docker run --restart unless-stopped -d -p 9092:9092 --name single-broker apache/kafka:latest
```

If the container is already running from Lab 1, skip this step.

### Step 2: Explore the kafka-topics.sh parameters ###
Run the tool with no arguments to see the full list of available options:
```
kafka-topics.sh
```

Take a moment to read through the options. Note the flags for:
- `--create` — creates a new topic
- `--describe` — shows partition and replication details for a topic
- `--list` — lists all topics on the broker
- `--delete` — deletes a topic
- `--partitions` — sets the number of partitions
- `--replication-factor` — sets the replication factor
- `--bootstrap-server` — identifies the broker to connect to

### Step 3: Create a new topic ###
Create a topic called `orders` with 3 partitions and a replication factor of 1:
```
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1 \
  --topic orders
```

Verify the topic was created and inspect its partition layout:
```
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic orders
```

Note which broker is the leader for each partition, and the partition offsets.

### Step 4: In a new terminal, start a console consumer ###
Open a new terminal window.

View the options for the consumer tool:
```
kafka-console-consumer.sh
```

Start a consumer on the `orders` topic, reading from the beginning and printing both keys and values:
```
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --from-beginning \
  --property print.key=true \
  --property print.value=true \
  --property key.separator=" : "
```

Leave this running — you will see messages appear here as you produce them in the next step.

### Step 5: In a new terminal, start a console producer ###
Open a third terminal window.

View the options for the producer tool:
```
kafka-console-producer.sh
```

Start a producer that sends key/value messages to the `orders` topic:
```
kafka-console-producer.sh \
  --broker-list localhost:9092 \
  --topic orders \
  --property "parse.key=true" \
  --property "key.separator=:"
```

Type a few messages in the format `key:value`, pressing Enter after each one:
```
customer-1:order placed
customer-2:order placed
customer-1:order shipped
customer-3:order placed
customer-2:order shipped
```

Verify that the messages appear in your consumer window. Note that messages with the same key (e.g. `customer-1`) always arrive in the same order — this is because they are routed to the same partition.

Press `Ctrl-C` to stop the producer when done.

### Step 6: Delete the topic ###
In your first terminal window, delete the `orders` topic:
```
kafka-topics.sh --delete \
  --bootstrap-server localhost:9092 \
  --topic orders
```

Confirm the topic has been removed by listing all topics:
```
kafka-topics.sh --list \
  --bootstrap-server localhost:9092
```

The `orders` topic should no longer appear in the list.

### Optional Extras ###

Can you produce messages **without** a key and observe how Kafka distributes them across partitions?

Remove the `parse.key` and `key.separator` properties from the producer command and send several messages. Use the `--property print.partition=true` flag on the consumer to see which partition each message lands on. Is the distribution what you expected?

####################
