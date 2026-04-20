## Goals: ##
1. Start a kafka broker as a docker container
2. Create a topic from the cli
3. Produce messages from the cli
4. Consume messages from the cli
5. View partition info from the cli
6. View consumer group info from the cli

## Steps: ##

### Step 1: Start a kafka broker as a docker container ###
```
docker run --restart unless-stopped -d -p 9092:9092 --name single-broker apache/kafka:latest​
```

### Step 2: View cli tools inside the container ###
```
docker exec -it single-broker /bin/bash
cd opt/kafka/bin/
ls
```

### Step 3: Create a topic from the cli ###
View the options for the tool kafka-topics.sh

```./kafka-topics.sh```

Create a topic called "test-topic" with default values for partitions and replications.

### Step 4: In a new terminal run a consumer ###
Repeat step 2 in a second command window

View the options for the tool kafka-console-consumer.sh

```./kafka-console-consumer.sh```

Start a console consumer process.

### Step 5: In a new terminal run a producer ###
Repeat step 2 in a third command window

View the options for the tool kafka-console-producer.sh

```./kafka-console-producer.sh```

Start a console producer process.

Verify that when you type messages in the producer window you can see them in the consumer window

### Optional Extras - 1 ###

View the options for the kafka-consumer-groups.sh tool.

If you restart your consumer process with a name, can you see details with the kafka-consumer-groups tool?

### Optional Extras - 2 ###

View the options for the kafka-topics.sh tool.

Can you use it to see the partitions for your topic?

Can you create a new topic with more than one partition?

####################
