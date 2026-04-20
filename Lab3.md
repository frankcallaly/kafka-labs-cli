## Goals: ##
1. Start a kafka cluster with 3 brokers as docker containers
2. Create a topic from the cli with multiple partitions and replications
3. Produce messages from the cli
4. Consume messages from the cli
5. View partition info from the cli
6. View consumer group info from the cli

## Steps: ##

### Step 1: Start a kafka cluster with docker-compose ###
Download the docker-compose.yaml file to your machine.

Explore the contents of the file and try to understand what it's doing.

In the same directory as the yaml file start the containers:
```
docker-compose up -d
```

Verify that the containers are running
```
docker-compose top
```

### Step 2: View cli tools inside the container ###
```
docker exec -it broker1 /bin/bash
cd opt/kafka/bin/
ls
```

### Step 3: Create a topic from the cli ###
View the options for the tool kafka-topics.sh

```./kafka-topics.sh```

Create a topic called "test-topic-2" with 2 partitions and 2 replications of each partition

### Step 4: In a new terminal run a consumer ###
Repeat step 2 in a second command window

View the options for the tool kafka-console-consumer.sh

```./kafka-console-consumer.sh```

Start a console consumer process, ensure that you give your consumer group a name.

Ensure that you specify that keys will be given and use the : character as a key/value separator.
Hint: Properties "parse.key=true" and "key.separator=:"

### Step 5: In a new terminal run a SECOND consumer ###
Repeat step 2 in a third command window

Start another console consumer process, ensure that you give your consumer group the same name as the previous consumer.

Ensure that you specify that keys will be given and use the : character as a key/value separator.
Hint: Properties "parse.key=true" and "key.separator=:" 

### Step 5: In a new terminal run a producer ###
Repeat step 2 in a fourth command window

View the options for the tool kafka-console-producer.sh

```./kafka-console-producer.sh```

Start a console producer process, ensure that you will specify the keys and use the : character as a key/value separator.
Hint: Properties "parse.key=true" and "key.separator=:" 

Verify that when you type messages in the producer window you can see them in the consumer window(s)


### Optional Extras - 1 ###

View the options for the kafka-topics.sh tool.

Can you use it to see the partitions for your topic?

What happens if you stop and start a broker using docker-compose and then continue to send messages?

### Optional Extras - 2 ###

View the options for the kafka-consumer-groups.sh tool.

If you stop a consumer process and send new messages, what happens and why?

If you start a new consumer process and send new messages, what happens and why?

