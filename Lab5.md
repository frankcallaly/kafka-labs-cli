## Goals: ##
1. Use kafka-get-offsets.sh to inspect partition offsets across topics
2. Use kafka-configs.sh to view and dynamically alter broker and topic configuration
3. Use kafka-log-dirs.sh to inspect partition log sizes and distribution across brokers

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

If the `activity` topic from Lab 4 is not present, create it now:
```
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --partitions 6 \
  --replication-factor 3 \
  --topic activity
```

Produce some messages to the topic so there is data to inspect:
```
for i in $(seq 1 30); do
  echo "user-$((RANDOM % 6 + 1)):event-$i" | kafka-console-producer.sh \
    --broker-list localhost:9092 \
    --topic activity \
    --property "parse.key=true" \
    --property "key.separator=:"
done
```

---

### Step 2: Explore kafka-get-offsets.sh ###
View the available options for the tool:
```
kafka-get-offsets.sh
```

Note the key flags:
- `--bootstrap-server` — the broker to connect to
- `--topic` — the topic to query
- `--topic-partitions` — query specific topic/partition combinations
- `--time` — `latest` (default), `earliest`, or a specific timestamp
- `--exclude-internal-topics` — exclude internal topics such as `__consumer_offsets`

Get the latest (end) offsets for all partitions of the `activity` topic:
```
kafka-get-offsets.sh \
  --bootstrap-server localhost:9092 \
  --topic activity
```

The output format is `topic:partition:offset`. Note how many messages have been written to each partition — they may not be evenly distributed depending on the keys used.

Get the earliest (start) offsets to see where each partition log begins:
```
kafka-get-offsets.sh \
  --bootstrap-server localhost:9092 \
  --topic activity \
  --time earliest
```

Query offsets for a specific partition only:
```
kafka-get-offsets.sh \
  --bootstrap-server localhost:9092 \
  --topic-partitions activity:0,activity:1
```

Get offsets across all topics on the cluster (excluding internal topics):
```
kafka-get-offsets.sh \
  --bootstrap-server localhost:9092 \
  --exclude-internal-topics
```

---

### Step 3: Explore kafka-configs.sh ###
View the available options for the tool:
```
kafka-configs.sh
```

Note the key flags:
- `--describe` — view current configuration
- `--alter` — change configuration dynamically without restarting
- `--entity-type` — one of `brokers`, `topics`, `clients`, `users`
- `--entity-name` — the name of the entity to describe or alter
- `--add-config` — configuration property to add or update
- `--delete-config` — configuration property to remove (revert to default)

**Viewing topic configuration**

Describe the current configuration for the `activity` topic:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --entity-type topics \
  --entity-name activity
```

At this point the output will likely show no dynamic overrides — the topic is using broker defaults for all settings.

**Altering topic configuration**

Set a custom retention period of 1 hour (3,600,000 ms) on the `activity` topic:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --alter \
  --entity-type topics \
  --entity-name activity \
  --add-config retention.ms=3600000
```

Describe the topic configuration again to confirm the override has been applied:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --entity-type topics \
  --entity-name activity
```

You should now see `retention.ms=3600000` listed as a dynamic topic config.

Set a maximum message size of 512KB on the topic:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --alter \
  --entity-type topics \
  --entity-name activity \
  --add-config max.message.bytes=524288
```

**Viewing broker configuration**

Describe the configuration for broker 1:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --entity-type brokers \
  --entity-name 1
```

This shows all dynamic configuration overrides applied to the broker. Note that static configuration set in `server.properties` at startup does not appear here — only properties that have been altered dynamically are shown.

**Reverting a configuration override**

Remove the custom retention period and revert to the broker default:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --alter \
  --entity-type topics \
  --entity-name activity \
  --delete-config retention.ms
```

Verify the override has been removed:
```
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --entity-type topics \
  --entity-name activity
```

---

### Step 4: Explore kafka-log-dirs.sh ###
View the available options for the tool:
```
kafka-log-dirs.sh
```

Note the key flags:
- `--bootstrap-server` — the broker to connect to
- `--broker-list` — comma-separated list of broker IDs to query
- `--topic-list` — comma-separated list of topics to query

Get the log directory information for all brokers across all topics:
```
kafka-log-dirs.sh \
  --bootstrap-server localhost:9092 \
  --broker-list 1,2,3 \
  --topic-list activity
```

The output is JSON. For each broker you will see:
- The log directory path
- Each partition replica stored on that broker
- The `size` in bytes of the partition log
- The `offsetLag` — how far the replica is behind the leader

Look at how the 6 partitions of the `activity` topic are distributed across the 3 brokers. With a replication factor of 3, each broker should hold replicas for all 6 partitions — but the leader for each partition will differ.

Produce more messages to the topic and re-run the command to see the partition sizes grow:
```
for i in $(seq 1 50); do
  echo "user-$((RANDOM % 6 + 1)):event-$i" | kafka-console-producer.sh \
    --broker-list localhost:9092 \
    --topic activity \
    --property "parse.key=true" \
    --property "key.separator=:"
done
```

```
kafka-log-dirs.sh \
  --bootstrap-server localhost:9092 \
  --broker-list 1,2,3 \
  --topic-list activity
```

### Optional Extras ###

Can you use `kafka-get-offsets.sh` with the `--time` flag and a Unix timestamp in milliseconds to find the offset of the first message written after a specific point in time?

Can you use `kafka-configs.sh` to set a retention policy by size rather than time? Look for the `retention.bytes` configuration property.

Can you identify from the `kafka-log-dirs.sh` output which broker is the leader for each partition, and cross-reference this with the output of `kafka-topics.sh --describe`?

####################
