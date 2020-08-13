<img align="right" src="./logo-small.png">


Lab. Performance Tuning and Monitoring
---------------------------------------


Performance is a critical requirement for many applications. Each
component in the communication flow between the components in a system
impacts performance, including the message broker. In this lab, we
will focus our attention on optimizing and monitoring the performance of
the RabbitMQ message broker and using various benchmarks to compare
RabbitMQ against other brokers.

The following topics will be covered in this lab:

-   Performance tuning of RabbitMQ instances
-   Monitoring RabbitMQ instances
-   Comparing RabbitMQ with other message brokers


Performance tuning of RabbitMQ instances
----------------------------------------



Tuning  the performance of a system is, in many
cases, a nontrivial process that is conducted gradually over time. This
also applies to the message broker itself. The RabbitMQ team has done a
pretty good job in optimizing the  various bits and
pieces of the broker over time. One such example is topic exchanges.
Version 2.4.0 significantly improved the performance of message routing
from topic exchanges using a tire data structure. Another one is the
significant improvement in performance predictability in version 2.8.1
during the heavy loading of the message broker due to improved memory
management. However, there are many scenarios that require the tuning of
the broker based on the usage patterns and properties of the system, as
we shall see in this lab.

To understand better how to tune the performance of our broker, let's
take a look at the standard three-tier broker setup:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_07_01.jpg)


We can consider  performance tuning at each level of
message passing as follows:


-   The sender may decide to optimize the way it establishes the
    connection to the broker (the number of channels created, usage of
    multiple threads for the creation of channels, and sending of
    messages), the size of the messages (whether the compression or
    batching of messages is proper), whether to use AMQP transactions or
    publisher confirms (which may hit message performance in terms of
    reliable delivery---reliability typically always implies a trade-off
    for performance), and message TTL (time to live).

-   The network link between the sender/consumer and broker might be an
    issue. While in systems where RabbitMQ is a component that provides
    loosely-coupled communication between the system components running
    on the same server or server cluster, network links may not be an
    issue, but if we use RabbitMQ to process messages being sent from a
    system on a remote network, this may be an issue. In this case, it
    is a shared responsibility between the AMQP client and server to
    tune the way channels are created in a connection or the size of
    messages processed in the channel. One possible solution would be to
    establish a dedicated line between the sender/consumer and broker.
    Network tuning may improve the communication link; network
    optimizations are out of scope for this lab.

-   Broker optimizations are for focus points when we discuss
    performance tuning in terms of RabbitMQ. This involves a number of
    aspects such as memory management, CPU utilization (in terms of
    multiple cores), storage of persistent and transient messages on the
    disk, faster execution of Erlang code from the RabbitMQ broker,
    impact of node synchronization and queue mirroring in a cluster, per
    queue message TTL customization, queue creation/deletion rates,
    message sending/consumption rates, and complexity of binding key
    patterns.

-   The consumer may use similar optimization techniques as the sender
    with the addition of broker subscription management (such as
    preventing excessive subscriptions to the broker).


To check the  performance load, you need to prepare
a maximum-sized volume of messages of the expected size to send for the
processing and measuring of latency and throughput. Let's kick off our
performance tuning guide by taking into account these considerations.





### Memory usage


Persistent  messages are always written to the disk
once they arrive on a queue, while transient messages will be written to
the disk under high memory consumption (based on the memory limit
specified for use by the RabbitMQ broker instance). Each disk operation
slows down the message processing. By default, RabbitMQ is configured to
use up to 40% of the physical RAM on the machine on which an instance
runs; although this is not guaranteed as it only implies a threshold at
which publishers are notified to slow down message sending (throttled).
Be careful to set the parameter properly in case multiple RabbitMQ
instances are running on the same physical/virtual machine. Assuming
that you have a single instance running per workstation, you can
increase the parameter so that RabbitMQ may consume more memory for its
queues. This can be done either in the RabbitMQ configuration file or
using the `rabbitmqctl` utility as follows:





```
rabbitmqctl set_vm_memory_high_watermark 0.7
```


You should see a message that tells you whether the memory threshold has
been set successfully:





```
Setting memory threshold on rabbit@DOMAIN to 0.7 ...
```


In this case, we are assuming that a single instance is running on the
workstation and there are no other applications running on the same
server. In case you run a cluster of three nodes on the same machine,
you may want to set the parameter to each of them to something less than
0.33 (for example, 0.25):





```
rabbitmqctl set_vm_memory_high_watermark 0.25
rabbitmqctl -n instance1 set_vm_memory_high_watermark 0.25
rabbitmqctl -n instance2 set_vm_memory_high_watermark 0.25
```


Before RabbitMQ hits the memory limit in order to start the persistence
of messages on the disk (persistent messages are already stored on the
disk as they are persisted upon arrival in the queue, but they need to
be removed from memory anyway), saving to the disk starts earlier (by
default, when 50% of the maximum memory limit is reached). To change
this threshold (let's say, to 80%), you need to set the
`vm_memory_high_watermark_paging_ratio` parameter per each
RabbitMQ node as follows:





```
rabbitmqctl eval "application:set_env(rabbit, vm_memory_high_watermark_paging_ratio, 0.8). "
```


You can also  set the parameter in the RabbitMQ
configuration file before the node is started. The memory consumption in
the broker is affected by the number of client connections, number of
queues and messages in each of them, enabled plugins and the amount of
memory that they use, in-memory Mnesia metadata and message store index,
and the additional amount of memory used by the Erlang VM.






### Faster runtime execution


Erlang  supports the  
[**HiPE**] ([**High Performance Erlang**]) compilation
for some platforms that improves the performance of message processing
by the RabbitMQ broker. (At the time of writing, this was still in the
experimental phase.) The HiPe compiler is pretty similar in comparison
to a server Java virtual machine---more native optimizations are done on
the startup of the server Java application resulting in an improved
runtime execution. In many scenarios, the start up time of the RabbitMQ
broker may not be critical so HiPe compilation may be a good
optimization. Behind the scenes, the Erlang VM precompiles the RabbitMQ
modules by passing the \[native\] parameter to the compiler that
triggers the HiPE compilation. On some platforms, however (such as
Windows at the time of writing), the HiPE compilation is not supported.
In order to enable the HiPE compilation for RabbitMQ, you can set the
`hipe_compile` parameter to `true` in the RabbitMQ
configuration file. In case the HiPE compilation is not enabled for the
particular platform where the RabbitMQ instances are running, you will
get a message in the instance logs that the HiPE compilation is not
performed.






### Message size


Smaller  messages can improve the latency (time to
process a single message) and throughput (message rate per period of
time). To reduce the message size, you can use a proper format for the
marshalling and unmarshalling of messages, for example, JSON instead of
XML. Try to avoid additional information as part of the message in order
to reduce the size of the message further.






### The maximum frame size of messages


A frame is a  basic unit of data transfer in the
AMQP protocol. There are different types of AMQP frames used to
establish the AMQP protocol life cycle. The transfer frame is
particularly used to transfer the message data between the RabbitMQ
broker and clients. The size of the message frame can affect the latency
and throughout. Typically, this value should not be changed but in case
you have messages bigger than 128 MB (the default maximum frame size),
then message fragmentation occurs---the message is split into multiple
frames. The more fragmentation there is, the less throughout there is
for the messages. The minimum size of frames in RabbitMQ is 4 KB.
Although the smaller maximum size of frames may degrade the throughput,
it may improve the latency, but you need to measure the performance of
your setup. To change the maximum frame size, you can set the
`frame_max` parameter to a particular value (in bytes) in the
RabbitMQ configuration file.






### The maximum number of channels


The  number of channels created from a connection to
the RabbitMQ server can affect the performance. An application can
achieve better throughput if more channels are used, and the application
uses a channel-per-thread approach to send messages. However, the more
channels there are in the RabbitMQ message broker, the more memory is
consumed. To set the maximum number of channels that an application can
use, use the `channel_max` parameter in the RabbitMQ
configuration file. The default value is zero meaning that there is no
limit for the number of channels that an application can create.






### Connection heartbeats


Connection heartbeats  provide a mechanism to detect
a dead TCP connection from the client (sender/consumer) and RabbitMQ
broker. The mechanism works by setting a heartbeat timeout from the
RabbitMQ client. (By default, it is set to 580 seconds, which can be a
pretty big timeout depending on your messaging use cases.) The RabbitMQ
server sends a heartbeat frame to the client and waits for a response.
If either side of the connection detects that more than two heartbeats
have been missed, then a TCP connection is detected that can be
typically handled by the client by catching a proper exception
(`MissedHeartbeatException` is thrown by the RabbitMQ Java
client). A heartbeat is sent every timeout/2 period of time. The
heartbeat timeout can be changed by either setting the heartbeat
parameter in the RabbitMQ configuration file or using a proper method in
the RabbitMQ client library to set a value for the heartbeat period
before creating a connection to the broker. Make sure that the heartbeat
is set to at least a few seconds as the performance can degrade
(especially in cases when the broker performs intense message
processing).






### Clustering and high availability


Clustering  can affect the performance of the broker
in terms of several different aspects. Heartbeats cannot be sent only
between the clients and RabbitMQ broker but also between nodes in a
RabbitMQ cluster in order to detect node availability. The
`net_ticktime` parameter specifies the frequency of sending
heartbeat messages between nodes in the cluster. The default value is 60
seconds, which means that a heartbeat is being sent roughly every 15
seconds (four times per `net_ticktime` period). Decreasing
this value to just a few seconds in a large cluster can have a slight
effect on the performance of the cluster. This applies to
`cluster_keepalive_interval` that is used to send keepalive
messages from a node to all the other nodes in the cluster and indicates
that the node is up (the default is 10,000 milliseconds). A much larger
value than 60 seconds imposes a risk of detecting a dead node too late
in time.

Another factor could  be the rate of exchange/queue
creation and deletion in a cluster. As every queue creates a new Erlang
process and the information about the queue must be synchronized with
all the nodes in the cluster, this can consume additional resources and
decrease the performance. Imagine that you have a large number of queues
and exchanges being created in a cluster, each one of them creates a
separate Erlang process on the cluster node on which it is created, and
information about each queue must propagate to each node in the cluster
using Erlang message passing. Each cluster node needs to persist the
information about the exchanges, queues, and other items in the cluster
on the disk (depending on the type of node). Now, imagine that you have
a large cluster and each queue being created/deleted is mirrored over
all the nodes in the cluster, then you have a recipe for performance
issues.

The following is a short list of guidelines considering the performance
in terms of clustering and high availability:


-   Try to minimize the number of exchanges and queues created and
    deleted in a RabbitMQ cluster.

-   If you have a large enough number of disk nodes and you want to
    scale, you can add RAM nodes instead of DISK nodes in order to
    improve the performance in terms of exchange/queue creation.

-   Mirror a queue on several other nodes in the cluster rather than all
    the nodes in the cluster. The replication factor depends on your
    reliability constraints, but replicating the queue contents over all
    the nodes in the cluster can hit the performance seriously,
    especially when you have a large RabbitMQ cluster.

-   Choose carefully which queues need to be mirrored and avoid the
    mirroring of queues that need to imply message reliability.

-   Last but not least, try to distribute the queues evenly among the
    nodes in a cluster.







### QoS prefetching


If you  have been sending messages to a queue and
one or a few consumers subscribe to this queue, the consumers may try to
fetch and buffer a large number of messages for consumption before
sending any acknowledgments, which can actually drain resources on the
consumer node and slow it down. To prevent this, you can use the
`basic.qos` operation during the channel creation (when
creating the channel from the client) to specify the maximum number of
messages that can be prefetched (buffered) by a consumer before they are
acknowledged. For example, using the Java client, you can set the
prefetch count to 50 per channel consumer using the following line of
code:





```
channel.basicQos(50);
```


A channel can have a prefetch count limit regardless of the number of
consumers:





```
channel.basicQos(100, true);
```


The general recommendation is to set a higher prefetch count (for
example, 40 or 50) in order to improve the performance. However, a large
prefetch count can prevent the event distribution of messages among the
consumers and so the value must be tuned with caution.






### Message persistence


Message persistence  in RabbitMQ also affects the
processing time for messages. We already discussed that transient and
persistent messages need to be persisted on the disk by RabbitMQ. The
persistence layer in RabbitMQ provides a message store to store messages
on the disk and also a queue index to keep information about the
location of a message in a queue and additional information (for
example, whether the message has been acknowledged or not) in memory.
When under memory pressure, the queue index may still preserve small
messages in-memory and flush only large messages to the message store.
The default size of messages that RabbitMQ tries to keep in-memory is 4
kilobytes and is specified by the
`queue_index_embed_msgs_below` parameter, which can be
modified in the RabbitMQ configuration file. Setting a larger value of
the parameter can allow you to store more messages in-memory, thus
reducing IO operations. However, as each queue index points to a segment
file held in-memory that can store 16, 384, increasing the value of the
`queue_index_embed_msgs_below` parameter even slightly may
increase the memory consumption drastically on the broker with regard to
improved performance. Another way that the performance might be affected
based on your scenario would be using a custom backing store that allows
you to store messages in a manner different from the default backing
store that writes them to the disk. This can either improve or decrease
the performance of your message broker.

For more  information about message persistence and
backing stores used in RabbitMQ, you can review the following posts from
the RabbitMQ documentation:


-   Check this link for persistence configuration:
    <https://www.rabbitmq.com/persistence-conf.html>

-   RabbitMQ backing stores:
    <http://www.rabbitmq.com/blog/2011/01/20/rabbitmq-backing-stores-databases-and-disks/>







### Mnesia transaction logs


The  Mnesia database used by RabbitMQ supports the
atomicity of operations via transactions. Each transaction log is stored
in the memory before being flushed to the disk (in the database itself)
and this is performed periodically by Mnesia. This can affect the
performance due to the number of disk writes. To reduce disk writes, you
can increase the size of the transaction log entries kept by Mnesia
in-memory by setting the `dump_log_write_threshold` parameter
in the configuration file (default value is 100).






### Acknowledgements, transactions and publisher confirms


In case  you release the reliability constraints,
you can improve the performance by avoiding the usage of message
acknowledgements, AMQP  transactions, and publisher
confirms. In case this is not acceptable, you can at least release
some  constraints. For publishers, you can use
publisher confirms for a batch of messages. For consumers, you can send
a single acknowledgment (using the `basic.ack` AMQL command)
for multiple messages by specifying a multiple flag set to
`true` and `delivery_tag` set to `0`
rather than sending an acknowledgement for each message separately.
Prefer publisher confirms instead of AMQP transactions for much better
performance.






### Message routing


The performance  can be hit not only by the
complexity of the binding key, but also by the type of exchange that you
use. Topic exchanges are slower than direct or fanout exchanges, and a
headers exchange can be slower than a direct exchange that is dependent
on the number of message keys used to determine where a message will be
routed by the headers exchange. A headers exchange can be slower than a
topic exchange. In case both types of exchanges are an option for your
messaging scenario, make sure that you measure the performance using
both types of exchanges.






### Queue creation/deletion


We already discussed that queue creation  and
deletion might be one of the factors that affects the performance in
terms of synchronization between nodes in a cluster. There are other
queue parameters that can affect the performance (both running a single
node and cluster). Queues can be created with the
`auto-delete` flag set to true. For example, using the Java
client and an already created channel, you can declare
`sample_queue` as `auto-delete`:





```
channel.queueDeclare("sample_queue", false, false, true, null);
```


If the queue  does not have any consumers, it is
never deleted. However, after the already existing subscribers are
removed (either unsubscribed from the queue or dropped due to a
connection failure), then the queue is automatically deleted and must be
created again. If you have a large number of such queues, then intensive
queue creation and deletion can affect message processing. You can also
achieve the same effect by setting a rather small value for the queue
  [**TTL**] ([**Time-to-live**]),
fro example, just a few milliseconds. In this case, after there are no
consumers and no operations to retrieve a message from the queue have
been performed for the specified TTL period of time, then the queue is
dropped. The following example sets a TTL of just five milliseconds on
the `sample_queue` queue when it is declared using the
`x-expires` parameter. Note that you can set it as a policy
for all the queues using the `rabbitmqctl` utility as well;
refer to the RabbitMQ documentation.





```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-expires", 5).
channel.queueDeclare("sample_queue", false, false, false, args)
```







### Queue message TTL


In order to avoid  the saturation of a queue, which
can slow down the processing of subsequent messages and increase the
risk of overconsumption when one or more consumers are present as we
already saw in QoS prefetching, we can set a per-queue message TTL. The
following example sets a message TTL for the  
`sample_queue` queue using the  
`x-message-ttl` parameter set to two minutes:





```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 120000).
channel.queueDeclare("sample_queue", false, false, false, args);
```


You can also set a  per-message TTL but this will
not solve the problem with queue saturation as messages stay in the
queue even after their TTL has expired and are dropped when they reach
the top of the queue (just before being consumed).






### Alarms


Alarms  are triggered by the RabbitMQ broker when
memory or disk size limits are exceeded. We already saw how to configure
memory usage using `set_vm_memory_high_watermark`. This
parameter also specifies when producer throttling (intentional slowing
down of message sending) takes place. Producer connection can also be
blocked entirely in case a memory goes critically high; the management
UI shows this condition in the [**Connections**] tab for the
blocked connections. Disk size can also be an issue for the performance.
By default, RabbitMQ requires at least 50 MB of free disk space on the
location of the RabbitMQ message store. If this threshold is hit, the
throttling of the producers and connection blocking starts taking place.
A general recommendation from the RabbitMQ documentation is to set the
minimum free disk size to the amount of memory installed on the machine.
To do this, you can set the `disk_free_limit` parameter in the
RabbitMQ configuration file. You can also set a value relative to the
amount of memory on the machine by setting `disk_free_limit`
to `{mem_relative, 1.0}`. You should, however, check the
RabbitMQ log files on the particular node to make sure that RabbitMQ has
managed to detect the size of the memory on the machine properly. For
example, on an 8 GB machine with a default setting of 40% for the
maximum memory limit for use by the broker, you can see something
similar to the following:





```
Memory limit set to 3241MB of 8104MB total.
```


You can also use the `rabbitmqctl` utility to check the
current setting of the `disk_free_limit` [ ]and[
] `set_vm_memory_high_watermark` parameters:





```
rabbitmqctl status
```


This outputs a lot of additional information such as the number of used
file descriptors, used Erlang processes, and so on:





```
{vm_memory_high_watermark,0.4},
 {vm_memory_limit,3399178649},
 {disk_free_limit,50000000},
 {disk_free,87735959552},
 {file_descriptors,
     [{total_limit,8092},{total_used,4},{sockets_limit,7280},{sockets_used,2}]},{processes,[{limit,1048576},{used,201}]},
```


If a memory  or disk alarm has been raised, this
will be displayed as part of the preceding output; if no alarms have
been triggered, the parameter is an empty list:





```
{alarms,[]}
```


Now, you can see that when a memory or disk alarm triggers, the
performance can slow down drastically. So, apart from a decent amount of
memory and large enough limit of maximum memory for use by the broker,
you also need a decent amount of disk space to store transient and
persistent messages along with a proper setting of the minimum disk free
space threshold taken into consideration by the message broker.






### Network tuning


The  RabbitMQ documentation mentions several network
improvements that can increase the message throughput with the most
significant one being the TCP buffer size. The operating system
typically allocates memory automatically for a TCP connection buffer,
but you can explicitly specify the size of the TCP buffer used by
RabbitMQ connections using the RabbitMQ configuration. Another factor is
Nagle's algorithm that provides you with more efficient handling of
really small TCP packets. However, the algorithm can typically be
disabled in case you don't send small-sized TCP packets as this can
even decrease the performance. The following configuration of the
`tcp_listen_options` parameter in the RabbitMQ configuration
sets the TCP buffers for the publisher/consumer connections to 256 KB
and disables the Nagle's algorithm explicitly (it is disabled by
default in the later versions of RabbitMQ clients but can be enabled
when creating a connection from the client). For example,
`ConnectionFactory` in the Java client uses a
`SocketConfigurator` instance to configure the TCP socket to
connect to the broker and disables the algorithm by default on the
socket with `socket.setTcpNoDelay(true)`:





```
  {nodelay,   true}  
  {sndbuf,    262144},
  {recbuf,    262144}
```


In case you have a large number of connections, you can set this value
to a smaller value and also increase the number of file handles used by
the RabbitMQ instance. To do this, you can use the `ulimit`
command in Linux before starting up your Rabbit instance. The following
example sets the maximum open files handle to `65536`:





```
ulimit -n 65536
```


Another tuning option suggested by the RabbitMQ documentation is the
size of the Erlang thread pool used to handle IO operations. A general
recommendation is to use at least 12 threads  per
core. To set a value, you can set the following environment variable
prior to starting the broker (in this example, we set the value to 96
for an eight-core machine):





```
RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS="+A 96"
```


However, you don't have any guarantees that increasing the value will
improve the throughput; you need to do the proper measurements.






### Client tuning


You  can improve the publisher/consumer performance
in terms of message publishing or message consumption using more threads
to create channels to the message broker. In terms of consumers, you
must be careful when you share a channel among multiple threads (each
using a separate set of queues) and you have QoS enabled for the shared
channels. This can introduce unpredictable behavior among the consumers.
Another case is when you have multiple subscriptions from different
threads and you need to acknowledge multiple messages at once, this
requires proper coordination among consumer threads, which will increase
the complexity of your consumer.






### Performance testing


We already  discussed a variety of tuning options
and we can use this knowledge to create a proper strategy for the
performance tuning of our RabbitMQ instance/cluster. The process can be
divided roughly into two phases executed iteratively:


-   Perform a RabbitMQ optimization as suggested in the previous
    sections, such as changing a configuration parameter, policy, or a
    routing pattern, reducing the message size, or increasing system
    resources such as RAM or disk space (along with tuning of the proper
    RabbitMQ parameters).

-   Measure the performance of your broker's setup and see if the
    performance improves. Always consider conducting performance tests
    on the maximum performance limits in non-peak hours even with the
    risk of crashing your system.


An ideal scenario would be if you have a test environment that mimics
your production environment as closely as possible, and you can measure
the performance over this setup and apply settings to the real
environment without disrupting users or, even better, have a load
balancer that would allow you to measure and tune the performance on
only one node/cluster while the other nodes/clusters continue to operate
normally. Unfortunately, this is not always the case, so you may need to
do performance measurements and load testing directly on your production
environment---better finding a bottleneck sooner than discovering it
later the hard way. When conducting performance testing, you can
consider the following  basic factors and do proper
combinations on any of them (based on your use cases):


-   The size of messages

-   The number of messages

-   The type of messages (transient/persistent)

-   The number of connections

-   The number of channels

-   The number of producers and consumers

-   The ratio of the number of producers and consumers

-   The number of pre-existing messages in a queue or set of queues


Typically, you try to use a tool that suits your own needs in terms of
performance testing or use an already existing one. We will first
briefly cover the PerfTest Java utility that comes with the RabbitMQ
Java client and see how to use it in order to conduct performance
measurements of our RabbitMQ message broker setup. Then, we will see how
to build our own tool on top of PerfTest in order to execute performance
tests against our current message broker setup in a loosely coupled
manner (independent of the message broker implementation) and see later
how to extend this tool with support for additional message brokers.

You can download the RabbitMQ Java client by cloning the
`rabbitmq-codegen` and `rabbitmq-java-client` GitHub
repository. You also need to install Python 2.x and the latest version
of Ant in order to build the Java client (Python 3.x is not supported at
the time of writing this book). To download and build the project after
you have installed Python and Ant, execute the following:





```
git clone https://github.com/rabbitmq/rabbitmq-codegen
git clone https://github.com/rabbitmq/rabbitmq-java-client
cd rabbitmq-java-client
ant dist
```


You can then either include the `rabbitmq-java-VERSION JAR` in
the build path of your project (and use it with a testing library such
as JUnit or TestNG to build your performance test suite or build a
custom tool on top of it) or execute the PerfTest utility directly from
the command line and observe statistics. The following example shows the
available options for the PerfTest utility in Windows (in a Linux
distribution, you can use the `runjava.sh` script
alternatively):





```
cd build/dist
runjava.sh com.rabbitmq.examples.PerfTest –help
```


As you can see, it takes into account many of the factors that can
affect the performance and we already covered this in this section. In
addition, it allows you to set different criteria to conduct performance
measurements including the prefilling of queues with messages. It lacks
features for the testing of the performance in a cluster, such as
setting up mirroring policies or precreating multiple queues with proper
distribution over the cluster nodes. However, you 
can easily build your own tool on top of PerfTest that does
that for you. Let's assume that we have our three-node RabbitMQ local
cluster up and running. The tool performs the following functions:


-   It starts up a number of consumers in separate consumer threads;
    only one consumer is started by default

-   It starts up a number of producers in separate producer threads;
    only one producer is started by default

-   It starts sending messages from the producers and consuming them
    from the consumers

-   It displays the collected statistics for the time period (starting
    with one second) and the number of sent and consumed messages for
    this period along with the minimum, average, and maximum latency for
    a message


Before running the tool, you must take into account several important
facts:


-   If you specify the number of messages to the publisher, be sure to
    specify the same or smaller number of messages to be consumed from
    the consumers; otherwise, the tool will hang and will not display
    any statistics (at least one consumer will still be waiting for
    messages). For example, if you have one producer and you want to
    send 10,000 messages to two consumers, you must specify a value of
    50,000 or less for the consumer message count.

-   If you specify the number of messages to the producer, be sure to
    specify a large enough amount of messages (that will require more
    than a second of processing) in order to get accurate statistics;
    for a small amount of messages, PerfTest will not give you accurate
    statistics.


The following example runs the tool with auto-acknowledgment by sending
messages from a single producer and binding a single consumer:





```
cd build/dist
runjava.bat com.rabbitmq.examples.PerfTest -a
```


We can observe the following result:





```
starting consumer #0
starting producer #0
time: 1.000s, sent: 23959 msg/s, received: 20884 msg/s, min/avg/max latency: 210
/65998/93740 microseconds
time: 2.000s, sent: 51274 msg/s, received: 51371 msg/s, min/avg/max latency: 495
19/59427/94140 microseconds
time: 3.000s, sent: 53224 msg/s, received: 52846 msg/s, min/avg/max latency: 487
12/57278/68175 microseconds
time: 4.000s, sent: 53228 msg/s, received: 53752 msg/s, min/avg/max latency: 477
22/56663/65392 microseconds
time: 5.000s, sent: 53878 msg/s, received: 53533 msg/s, min/avg/max latency: 487
26/57483/70630 microseconds
…
```


You can see that  after the first second, we produce
and consume roughly about 52,000 messages per second with
auto-acknowledgement enabled. Now, let's execute the same test with the
acknowledgment of each message from the consumer:





```
runjava.bat com.rabbitmq.examples.PerfTest
```


We can observe the following result:





```
starting consumer #0
starting producer #0
time: 1.000s, sent: 15088 msg/s, received: 11151 msg/s, min/avg/max latency: 262
6/133696/214058 microseconds
time: 2.001s, sent: 25932 msg/s, received: 23126 msg/s, min/avg/max latency: 137
341/213911/272298 microseconds
time: 3.001s, sent: 26605 msg/s, received: 22065 msg/s, min/avg/max latency: 249
500/333672/455356 microseconds
time: 4.002s, sent: 22690 msg/s, received: 19948 msg/s, min/avg/max latency: 444
164/570170/643165 microseconds
time: 5.002s, sent: 24013 msg/s, received: 20410 msg/s, min/avg/max latency: 562
357/654099/717019 microseconds
…
```


You can see now that the performance drops more than twice (roughly
about 21,000 messages per second) with acknowledgments from the
consumer, which is a significant performance hit. Let's also make
messages persistent before running the performance measurement:





```
runjava.bat com.rabbitmq.examples.PerfTest -f persistent
```


We can observe the following result:





```
starting consumer #0
starting producer #0
time: 1.004s, sent: 11297 msg/s, received: 6623 msg/s, min/avg/max latency: 3168
/227397/373579 microseconds
time: 2.006s, sent: 15388 msg/s, received: 11577 msg/s, min/avg/max latency: 338
389/456810/586714 microseconds
time: 3.006s, sent: 13493 msg/s, received: 10476 msg/s, min/avg/max latency: 570
519/711663/886369 microseconds
time: 4.006s, sent: 12850 msg/s, received: 9844 msg/s, min/avg/max latency: 8203
60/1052631/1172428 microseconds
time: 5.010s, sent: 14719 msg/s, received: 11384 msg/s, min/avg/max latency: 113
1484/1183177/1235015 microseconds
```


This is even  worse: about 10,000 messages per
second when message persistence takes place. You can specify further
options such as publisher confirms, number of consumers/producers,
messages, and others depending on your setup and messaging requirements.

The following example allows you to predict what would be the relative
time to produce and consume 1,000,000 messages of size 4 KB using a
single producer and consumer without acknowledgments:





```
runjava.bat com.rabbitmq.examples.PerfTest -a -C 1000000 -D 1000000 -s 4096
```


On the sample three-node RabbitMQ cluster, it took about 20 seconds to
process all the messages.



Monitoring of RabbitMQ instances
--------------------------------------------------



We have  been discussing various performance tuning
tips that would allow us to create a more scalable broker setup.
However, in order to be able to observe how our setup behaves in various
scenarios, it is not sufficient to do only partial performance
measurements using PerfTest, a custom performance tool, or even a
third-party performance-testing solution. In a production environment,
we would typically want to have a real-time monitoring solution that
would allow us to observe how our broker behaves at any point in time
enabling us to take measures as fast as possible when something goes
wrong with our RabbitMQ instances.

The RabbitMQ management plugin provides you with a good real-time
overview of the resource utilization of the instances of a cluster and
the message rates per queue or exchange. However, we may want to have a
central monitoring infrastructure that monitors all the parts of our
infrastructure, including the message broker. Moreover, we may want to
make use  of advanced features provided by a typical
monitoring solution such as the ability to receive notifications
(e-mail, SMS, and so on) when something wrong happens with the broker
such as a failed RabbitMQ instance or an exceeded memory / CPU / free
disk threshold. For this reason, we can leverage a monitoring solution
to do the job.

We will briefly discuss the capabilities provided by the management
plugin, and then we will see how to monitor RabbitMQ using Nagios,
Monit, or Munin assuming that we are running our RabbitMQ instances in a
Linux environment.





### The management UI


When  you navigate to the [**Overview**]
tab of the RabbitMQ management web interface and click on a node, you
can observe the resource consumption by this node in real time under the
[**Statistics**] section:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_07_02.jpg)


You can  observe the number of file descriptors or
socket descriptors that are used, Erlang processes, and memory currently
used by the message broker along with the current free disk space. On
the same page, you can observe more information about the distribution
of memory among the different components of the message broker under the
[**Memory details**] section by clicking on the
[**Update**] button first in order to take a memory snapshot:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_07_03.jpg)


You can also check the message rates from [**Queues**] and
[**Exchanges**] [ ]by clicking on a particular queue
 or exchange.






### Nagios


Nagios  is an open source system monitoring
application that provides a number of plugins to extend its capabilities
along with various types of integrations with different network
protocols and applications. In order to install Nagios in Ubuntu, you
can use the following command:





```
sudo apt-get update
sudo apt-get install nagios3 nagios-nrpe-plugin
```


When prompted  during the installation, specify a
proper password for the Nagios administrative panel. To check whether
the Nagios service is running, execute the following command:





```
sudo service nagios3 status
```


You should now be able to log in to the Nagios administrative interface
from `http://localhost/nagios3` and provide the
`nagiosadmin` user along with the password that you specified
during the installation. The next thing to do is to install some Nagios
health checks (or write your own if the installed ones are not proper):





```
git clone https://github.com/jamesc/nagios-plugins-rabbitmq
sudo chown -R nagios:nagios nagios-plugins-rabbitmq/
mv nagios-plugins-rabbitmq /usr/lib/nagios/plugins/
sudo apt-get install libnagios-plugin-perl
sudo apt-get install libnagios-object-perl
apt-get install perl-Nagios-Plugin 
apt-get install libreadonly-xs-perl
sudp perl -MCPAN -e 'install Bundle::LWP'
perl -MCPAN -e 'install Monitoring::Plugin'
sudo cp -R /usr/share/perl/5.14.2/CPAN/LWP/ /etc/perl/
sudo cpan install JSON
```


In short, the process described in the preceding commands is as follows:


1.  We download the sources of the health checks from the 
     [**nagios-plugins-rabbitmq**] GitHub
    repository. You can see the available checks (provided as Perl
    scripts) under the `nagios-plugins-rabbitmq/scripts`
    directory; they use the RabbitMQ management REST API.

2.  We change the permissions of the sources and move them to the Nagios
    plugins directory.

3.  We install the Monitoring:Plugin Perl plugin along with the
    additional dependencies that is needed in order to write plugins for
    Nagios under Perl; this is required as the RabbitMQ health checks
    that we downloaded are provided as Perl scripts that depend on this
    library.


To verify that  you can run a check, you can execute
the following:





```
cd nagios-plugins-rabbitmq/scripts
./check_rabbitmq_server
```


If you are prompted to provide a hostname, then check whether your
compiles are fine. You need to define a particular command using this
script in the `commands.cfg` configuration file of Nagios:





```
sudo vim /etc/nagios3/commands.cfg
define command {
 command_name check_rabbitmq_server
 command_line /usr/lib/nagios/plugins/nagios-plugins-rabbitmq/scripts/check_rabbitmq_server -H localhost --port=15672 -u guest -p guest
}
```


You can now restart the service with the following:





```
sudo service nagios3 restart 
```


When you navigate to the [**Configuration**] menu under the
[**System**] section, select [**Commands**] from the
dropdown and click on [**view**]; you should see the
`you check-rabbitmq-server `command in the list. In a similar
way, you can define the other already provided RabbitMQ checks if you
need them to monitor.

You can create a service definition that uses the command and allows you
to specify which groups you would like to notify, for example, in case
the RabbitMQ server goes down. You can do this with the other RabbitMQ
health checks as well. You can also write your own health checks for
RabbitMQ, for example, using Java and RabbitMQ management REST API or
the `rabbitmqctl` utility.






### Monit


Monit  is a Unix utility to monitor processes. You
can also use it to monitor the RabbitMQ instance process in a pretty
straightforward manner. Monit requires a `pid` file that
stores the process ID for the currently running process. In the
 earlier versions of the rabbitmq-server script
`init script under /etc/init.d`, you had to add the creation
and deletion of this `pid` file manually upon the service
startup/shutdown. However, the later versions of RabbitMQ store a
`pid` file for the RabbitMQ Erlang process under the
`/var/run/rabbitmq/pid` directory.

In order to install  Monit, execute the following
command:





```
sudo apt-get install monit 
```


You can then add the following configuration to the
`/etc/monit/monitrc` file in order to monitor the RabbitMQ
process from the localhost:





```
set httpd port 2812 and
use address localhost
allow localhost
allow @monit 
allow @users readonly

CHECK PROCESS rabbitmq-server WITH PIDFILE /var/run/rabbitmq/pid
  GROUP rabbitmq
  START PROGRAM "/usr/sbin/service rabbitmq-server start"
  STOP PROGRAM "/usr/sbin/service rabbitmq-server stop"
  IF DOES NOT EXIST FOR 3 CYCLES THEN RESTART
  IF FAILED PORT 5672 4 TIMES WITHIN 6 CYCLES THEN RESTART
```


You can  start monit in the background with the
following command:





```
sudo service monit start
sudo monit
```


You can then check the status of the monited processes (including the
Erlang process of RabbitMQ) using the following command:





```
sudo monit status
```


You should see an output similar to the following:





```
Process 'rabbitmq-server'
  status                            Running
  monitoring status                 Monitored
  pid                               1046
  parent pid                        1039
  uptime                            6d 12h 6m 
  children                          2
  memory kilobytes                  13680
  memory kilobytes total            14484
  memory percent                    0.6%
  memory percent total              0.7%
  cpu percent                       0.0%
  cpu percent total                 0.0%
  port response time                0.000s to localhost:5672 [DEFAULT via TCP]
  data collected                    Mon, 31 Aug 2015 01:47:59
```







### Munin


You can  use Munin as a nice alternative to Nagios
 for the monitoring. The following command installs
Munin in Ubuntu (note that the Apache HTTP server must also be
installed):





```
sudo apt-get install apache2
sudo apt-get install munin
```


You must then edit the Munin configuration:





```
vim /etc/munin/munin.conf
```


Uncomment the following and change the value of the 
 `htmldir` attribute to `/var/www/munin`:





```
dbdir  /var/lib/munin
htmldir /var /www/munin
logdir /var/log/munin
rundir  /var/run/munin

tmpldir /etc/munin/templates
```


Add the following to the Munin configuration file in order to enable
monitoring on the localhost:





```
[MuninMonitor]
    address 127.0.0.1
    use_node_name yes
```


Open the Munin Apache configuration and change the alias to allow
external connections:





```
sudo vim /etc/munin/apache.conf
Alias /munin /var/www/munin
<Directory /var/www/munin>
    Order allow,deny
    #Allow from localhost 127.0.0.0/8  ::1
    Allow from all
    Options None
```


Create the `/var/www/munin` directory, change permissions to
the munin user and group, and finally restart the apache2 and
`munin-node` services:





```
sudo mkdir /var/www/munin
sudo chown munin:munin /var/www/munin
sudo service munin-node restart
sudo service apache2 restart
```


If you navigate to `http://localhost/munin/`, you should be
able to see the Munin administrative interface:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_07_04.jpg)


Now, we  need to install the Munin RabbitMQ set of
 plugins. To do so, execute the following commands
in order to download the Munin plugins directly to the Munin plugins
directory:





```
cd /etc/munin/plugins/
sudo git clone https://github.com/ask/rabbitmq-munin
sudo cp rabbitmq-munin/* .
```


Add the following configuration to the
`/etc/munin/plugin-conf.d/munin-node` file:





```
sudo vim /etc/munin/plugin-conf.d/munin-node
[rabbitmq_connections]
user root

[rabbitmq_consumers]
user root

[rabbitmq_messages]
user root

[rabbitmq_messages_unacknowledged]
user root

[rabbitmq_messages_uncommitted]
user root

[rabbitmq_queue_memory]
user root
```


Finally, restart  the `munin-node` service
and check whether  you have the munin plugins
available from the administrative interface:





```
sudo service munin-node restart
```



![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_07_05.jpg)



Comparing RabbitMQ with other message brokers
---------------------------------------------------------------



It is not  uncommon that when it comes to choosing a
message broker for your system, you may not choose RabbitMQ as a proper
solution without any comparison with other message brokers. Although
RabbitMQ is a great technology, it can turn out that there is a better
message broker (either in turns of features or performance) based on
your requirements. For this reason, you can benchmark RabbitMQ against
other message brokers such as Qpid, ActiveMQ, ZeroMQ, HornetMQ, and
Kafka, just to name a few. For this, you can follow the approach
provided by the PerfTest tool and build a wrapper utility (that can
abstract PerfTest for RabbitMQ) that allows you to produce and consume
messages of different numbers and sizes on each of the brokers that you
would like to benchmark along with RabbitMQ.



Case Study : Performance tuning and monitoring of RabbitMQ instances in CSN
---------------------------------------------------------------------------------------------



The CSN team  decided to scale the system both
vertically and horizontally in terms of the RabbitMQ message broker by
introducing more RAM and disk space on each of the RabbitMQ instance
servers and change the RabbitMQ configuration parameters accordingly so
that the broker can use more memory  and disk space,
if needed. The team also decided to introduce more RAM nodes for the
chat queues along with a deployment of Nagios to monitor all the parts
of the system (including the message broker) and send notifications to
the team in case of issues with resource utilization based on defined
thresholds.



Summary
-------------------------



In this lab, we provided a list of performance tuning tips that can
be used to build a proper approach for the tuning of the performance of
the RabbitMQ message broker. We discussed how to measure the performance
using the [**PerfTest**] utility provided by the RabbitMQ Java
client and monitor the performance in real time using either the
management interface or third-party monitoring solution, such as Nagios,
Monit, or Munin. At the end, we discussed how we can compare the
performance of RabbitMQ against a few other message brokers that are
widely used in practice and compete against RabbitMQ.


Exercises
---------------------------




1.  How can you optimize the performance of a single RabbitMQ instance?

2.  How can you optimize the performance of a single RabbitMQ cluster?

3.  How do acknowledgments and publisher confirms affect the
    performance?

4.  What tool can you use to measure the performance of a RabbitMQ
    instance?

5.  How can you set memory and disk free limits per RabbitMQ instance?

6.  What is QoS prefetching and how does it affect the performance?

7.  How do message persistence and message TTL affect the performance?

8.  How can you monitor memory, disk, and CPU consumption of a RabbitMQ
    instance?

9.  How can you evaluate RabbitMQ against other message brokers in terms
    of performance?

10. Is RabbitMQ better than ActiveMQ or ZeroMQ in terms of performance?
