<img align="right" src="./logo-small.png">


# Java code for RabbitMQ
To successfully use the examples you will need a RabbitMQ node running locally.

## Requirements

You'll need to download the following JAR files from Maven Central:

 * [RabbitMQ Java Client](https://repo1.maven.org/maven2/com/rabbitmq/amqp-client/5.6.0/amqp-client-5.6.0.jar)
 * [SLF4J API](https://repo1.maven.org/maven2/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar)
 * [SLF4J Simple](https://repo1.maven.org/maven2/org/slf4j/slf4j-simple/1.7.25/slf4j-simple-1.7.25.jar)

For example, with `wget`:


``` shell
wget https://repo1.maven.org/maven2/com/rabbitmq/amqp-client/5.6.0/amqp-client-5.6.0.jar
wget https://repo1.maven.org/maven2/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar
wget https://repo1.maven.org/maven2/org/slf4j/slf4j-simple/1.7.25/slf4j-simple-1.7.25.jar
```

**Note:** Above jars have already been downloaded. 

Copy those files in your working directory, along the tutorials Java files.

To compile you only need the Rabbitmq Java client jar on the classpath.
To run them you'll need all the dependencies, see examples below.

You can set an environment variable for the jar files on the classpath e.g.

```
export CP=.:amqp-client-5.6.0.jar:slf4j-api-1.7.25.jar:slf4j-simple-1.7.25.jar
java -cp $CP Send
```

On Windows, use a semicolon instead of a colon to separate items in the classpath:

```
set CP=.;amqp-client-5.6.0.jar;slf4j-api-1.7.25.jar;slf4j-simple-1.7.25.jar
java -cp %CP% Send
```

## Code

#### [Tutorial one: "Hello World!"] `<host-ip>:<port>/lab/workspaces/lab1_java`

```
javac -cp amqp-client-5.6.0.jar Send.java Recv.java

# terminal tab 1
java -cp .:amqp-client-5.6.0.jar:slf4j-api-1.7.25.jar:slf4j-simple-1.7.25.jar Recv

# terminal tab 2
java -cp .:amqp-client-5.6.0.jar:slf4j-api-1.7.25.jar:slf4j-simple-1.7.25.jar Send
```

#### [Tutorial two: Work Queues] `<host-ip>:<port>/lab/workspaces/lab2_java`

```
javac -cp amqp-client-5.6.0.jar NewTask.java Worker.java

# terminal tab 1
java -cp $CP NewTask

# terminal tab 2
java -cp $CP Worker
```

#### [Tutorial three: Publish/Subscribe] `<host-ip>:<port>/lab/workspaces/lab3_java`

``` shell
javac -cp amqp-client-5.6.0.jar EmitLog.java ReceiveLogs.java

# terminal tab 1
java -cp $CP ReceiveLogs

# terminal tab 2
java -cp $CP EmitLog
```

#### [Tutorial four: Routing] `<host-ip>:<port>/lab/workspaces/lab4_java`

```
javac -cp $CP ReceiveLogsDirect.java EmitLogDirect.java

# terminal tab 1
java -cp $CP ReceiveLogsDirect warning error

# terminal tab 2
java -cp $CP ReceiveLogsDirect info warning error

# terminal tab 3
java -cp $CP EmitLogDirect error "Run. Run. Or it will explode."
```

#### [Tutorial five: Topics] `<host-ip>:<port>/lab/workspaces/lab5_java`

```
# To compile:
javac -cp $CP ReceiveLogsTopic.java EmitLogTopic.java

# To receive all the logs:
java -cp $CP ReceiveLogsTopic "#"

# To receive all logs from the facility "kern":
java -cp $CP ReceiveLogsTopic "kern.*"

# Or if you want to hear only about "critical" logs:
java -cp $CP ReceiveLogsTopic "*.critical"

# You can create multiple bindings:
java -cp $CP ReceiveLogsTopic "kern.*" "*.critical"

# And to emit a log with a routing key "kern.critical" type:
java -cp $CP EmitLogTopic "kern.critical" "A critical kernel error"
```

#### [Tutorial six: RPC] `<host-ip>:<port>/lab/workspaces/lab6_java`

```
# Compile and set up the classpath as usual (see tutorial one):
javac -cp $CP RPCClient.java RPCServer.java

# Our RPC service is now ready. We can start the server:
java -cp $CP RPCServer

# To request a fibonacci number run the client:
java -cp $CP RPCClient
```

#### [Tutorial seven: Publisher Confirms] `<host-ip>:<port>/lab/workspaces/lab7_java`

```
javac -cp $CP PublisherConfirms.java
java -cp $CP PublisherConfirms
```
