
<img align="right" src="./logo-small.png">

Lab. Performance Tuning for RabbitMQ
----------------------------------------


In this lab we will cover:


-   Multithreading and queues
-   System tuning
-   Improving bandwidth
-   Using different distribution tools


#### Pre-reqs:
- Google Chrome (Recommended)

**Note:** 
- Terminal is already running. You can also open new terminal by clicking:
`File` > `New` > `Terminal`.
- To copy and paste: use **Control-C** and to paste inside of a terminal, use **Control-V**

Run following command in the terminal and move into java files directory:

`cd /home/jovyan/work/rabbitmq-jupyterlab/CookBook/Lab08`


Introduction
--------------


There are no standard RabbitMQ tuning guidelines because different
applications should be optimized in different ways.

Very often the application needs to be optimized on the client side:


-   CPU-intensive applications can be optimized by running one thread
    for each CPU core
-   I/O-intensive applications can be optimized by running many threads
    per core in order to hide implicit latencies


In both cases  messaging is a perfect fit. In order
to optimize the network transfer rate, the AMQP standard prescribes that
messages are transferred in bunches, and then consumed one by one by the
client.

RabbitMQ allows multithreaded applications to consume messages
efficiently; this is covered in the [*Multithreading and
queues*] recipe.

Another frequent use case is when RabbitMQ is at the foundation of a
distributed application serving a large number of clients. In this case,
it's more realistic that the bottleneck is the broker and not the
client application.

In this case, it's important that our broker has one characteristic,
that is, scalability.

When the number of clients outgrow the current maximum capacity, it's
sufficient to add one or more nodes to the RabbitMQ cluster to
distribute the load and improve the total throughput.

Why to optimize then? Well, to reduce cost. The cost of the hardware,
electric power, cooling, or cloud computing resources.

In this lab, we will talk about the RabbitMQ performance, showing
some tips to improve the performance on both the client side and
eventually modifying the broker parameters.



System tuning
-------------

In this recipe, we will  appreciate some steps
useful to obtain the maximum performance from RabbitMQ. We will cover
the following topics:


-   The `vm_memory_high_watermark` configuration
    (<http://www.rabbitmq.com/memory.html>)

-   Erlang [**High Performance Erlang**] ([**HiPE**])
    (<http://erlang.org/doc/apps/hipe/>)


The `vm_memory_high_watermark` configuration
 is the maximum percentage of the system memory used to cache
messages before they are consumed or cached to the disk.

Before the limit is reached, by default, at fifty percent of
`vm_memory_high_watermark`, (or properly setting the
`vm_memory_high_watermark_paging_ratio` parameter, set to
`0.5` by default), RabbitMQ will start to move messages from
memory to on-disk paging space.

If neither this paging mechanism, nor the consumers are able to keep
pace with the producers, the limit will be reached, and then RabbitMQ
will block the producers.

In some cases, it is possible to enlarge these parameters, in order to
avoid starting to page messages to the disk too early. In this recipe,
we will see how to do it in conjunction with HiPE. There are two
different aspects, but the steps needed to accomplish them are very
similar.

You can use the code from the book repository in the directory
`Lab08/Recipe02`.





### Getting ready


To try this recipe, you need to start with RabbitMQ and have the
management plugin installed. Then, you need Java 1.7 or higher and
Apache maven.






### How to do it...


In order to obtain the  maximum performance from
RabbitMQ, you can perform the following steps:


1.  Configure the watermark using:



    ```
    rabbitmqctl set_vm_memory_high_watermark 0.6 
    ```
    

    [**Or directly in the rabbitmq.config file using:**]



    ```
    [{rabbit, [{vm_memory_high_watermark, 0.6}]}].
    ```
    

2.  Change the Linux `ulimit` parameter modifying the
    `/etc/default/rabbitmq-server` file. Then, you can improve
    RabbitMQ itself by using HiPE.

3.  Install the latest version of Erlang from
    <http://www.erlang.org/download.html>.

4.  Install HiPE in your system.

5.  Check that HiPE is correctly activated; if not, you need to install
    Erlang from the sources and activate it.

6.  Activate Erlang HiPE in the RabbitMQ configuration file. Create the
    `rabbitmq.config` file with this option or add it if the
    file already exists:



    ```
    [
      {rabbit, [{hipe_compile, true}]}
    ].
    ```
    

7.  Restart RabbitMQ.

8.  Check that in the RabbitMQ log file there is not a warning showing
    that HiPE has not been activated:



    ```
    =WARNING REPORT==== 6-Oct-2013::00:38:23 ===Not HiPE compiling: HiPE not found in this Erlang installation.
    ```
    







### How it works...


The watermark is the maximum memory used by RabbitMQ, by default it's
0.4 which means 40 percent of the installed physical memory. When the
memory reaches the watermark the broker stops accepting new connections
and messages. The watermark value is approximate; in some cases it could
be overcome by the default 40 percent. Anyway, when the server has lots
of RAM, you can increase the value, for example, to 60 percent, just to
tolerate the spikes. With `rabbitmqctl` the change is
temporary; when you modify the `rabbitmq.config` file, the
option is set permanently.

The `ulimit` parameter  by default is
`1024`. Increase the value to increase the number of files and
of sockets available to RabbitMQ.


### Tip

A too high value could impact negatively the system. Read about the
`ulimit` parameter at <https://wiki.debian.org/Limits>.


Using Erlang  HiPE is currently considered
experimental. If it works, we can use it. In case the system is
unstable, you need to disable it.

However, using it you can obtain a consistent 40 percent of CPU usage
improvement of the RabbitMQ server, in case this is your bottleneck. For
example, in the following screenshot, you can see the behavior of the
broker in a standard configuration with a producer and a 
consumer on localhost:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_08.jpg)


In this example, we have run both the producer and the consumer on the
localhost, sending 32 byte messages for 300 seconds, letting the
consumer consume all the messages in real time.

After HiPE has been activated, as shown with details in the following
screenshot, the same test behaves considerably better:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_09.jpg)


Before you activate HiPE in the RabbitMQ configuration file, you can
check if your local Erlang installation has it by just invoking the
`erl` command as follows:





```
# erlErlang R15B03 (erts-5.9.3.1) [source] [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]

EshellV5.9.3.1  (abort with ^G)1>
```


In case HiPE is  present, you will see
`[hipe]` among the options shown at startup.


### Tip

In Debian wheeze systems, once the latest Erlang version from the
Erlang-site is downloaded, you can add the HiPE module by installing:





```
apt-get install erlang-base-hipe
```


The other distributions have similar packages available as well.


Otherwise, you need to install it from an external package or from the
Erlang source code by downloading it from
<http://www.erlang.org/download.html> and installing it; remember to
specify the `--enable-hipe` option at the
`configure` step.

Once RabbitMQ has been configured too (in step 6), you will notice that
the restart of the server will take a long time; typically several
minutes.


### Tip

In most of the Linux distributions, the default RabbitMQ configuration
file is placed at `/etc/rabbitmq/rabbitmq.config`.


At this point, the RabbitMQ broker is HiPE-activated. The most demanding
parts are not interpreted anymore but compiled at startup into native
machine code.

You can further check  that in the log file you
don't see any message as follows:





```
=WARNING REPORT==== 6-Oct-2013::00:38:23 ===Not HiPE compiling: HiPE not found in this Erlang installation.
```



### Tip

By default, on Linux RabbitMQ, log files are placed in
`/var/log/rabbitmq`.


### There's more...


Since HiPE is an experimental option, we discourage its usage from the
beginning, given that usually the optimization effort needs to address
the application side optimization and scalability.

However, by enabling it, you can reduce the CPU usage and power
consumption of your servers; so this is an option you can consider when
optimizing your architecture.



Improving bandwidth
-------------------------------------



Using [**noAck**] flag  and
 managing the [**prefetch**] parameter
 is another client-side way to improve the performance and
the bandwidth. Both noAck and prefetch are used by the consumers.

In this example, we are going to create one producer and one consumer
using these parameters. You can find the source code at
`Lab08/Recipe03`.


Code for this lab is available in `/home/jovyan/work/rabbitmq-jupyterlab/CookBook/Lab08/Recipe03`

### Getting ready


You need Java 1.7 or higher and Apache maven.


### How to do it...


We skip the producer code because it's the same one shown in the
[*Multithreading and queues*] recipe. We still use the
`ReliableClient` class as the base class. Let's see the
consumer by performing the following steps:


1.  Create a maven project and add the RabbitMQ client dependency.

2.  Create a consumer main class, which reads from `args[]` to
    manage the consumer with the following four parameters:



    ```
    threadNumber = Integer.valueOf(args[0]);
    prefetchcount = Integer.valueOf(args[1]);
    autoAck = (Integer.valueOf(args[2]) != 0);
    print_thread_consumer
    = (Integer.valueOf(args[3]) != 0);
    ```
    

3.  Create a consumer that extends the `ReliableClient` class,
    and then set the `prefetch` and `noAck`
    parameters:



    ```
    internalChannel.basicQos(prefetch_count);
    internalChannel.basicConsume(Constants.queue, autoAck..
    ```
    







### How it works...


The prefetch-size is ignored if the `noAck` option is set, so
we divide the recipe in the following two sections:


-   Prefetch

-   noAck


The aim is to  understand how to manage the
client-side parameters to improve the performance and the bandwidth.





#### Prefetch


To set the  prefetch, use
`basicQos(prefetch_count)` (refer to step 3).

The prefetch count is the maximum number of unacknowledged messages: a
large value will let the client prefetch many messages in advance
without waiting for the acks of the messages being processed.

As stated at the beginning of the lab, there is not a one-for-all
rule when optimizing. In fact, improving the prefetch count can be
counterproductive when the per message processing time is important, and
we need to distribute and balance the processing.

Well firstly, maven will compile using the following command:



```
cd /home/jovyan/work/rabbitmq-jupyterlab/CookBook/Lab08/Recipe03

mvn clean compile assembly:single
```

**Note:** Jar is created inside target directory.
`cd target && ls -ltr`

Then, maven will create the `rmqAckTest.jar` package.

You can now try the  example, by changing the
parameters to see how the message rates change. We made a test using a
MacBook pro Dual Core, 4 GB RAM using the following parameters:


-   For the producer, we can run the following command:

    ```
    java -cp rmqAckTest.jar rmqexample.ProducerMain 1 100000 64000
    ```

-   For the consumer, we can run the following two tests:

    ```
    java -cp rmqAckTest.jar rmqexample.ConsumerMain 2 50 0 0
    java -cp rmqAckTest.jar rmqexample.ConsumerMain 2 1 0 0
    ```
    


The producer  uses `1` thread for 100
seconds and 64000 bytes as the message size.

The consumer uses `2` threads, the prefetch count in
`Test1` is `50` and in `Test2` is
`1`, `autoAck` is set to `false`, and
without printing the thread number on the console.

The results for these two tests were as follows:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_02.jpg)

Test1

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_03.jpg)

Test2


As you can see, the  prefetch count can make the
difference, especially when you have more consumers bound to the same
queue.



#### NoAck


To set noAck,  use
`basicConsume(Constants.queue, true)`. The parameter is useful
when you have a stream data or when it doesn't matter to send the acks
manually.


### Tip

In the Java API, the name of the `ack` parameter of
`Channel.basicConsume` as a Boolean, `autoack`, is
quite misleading; setting `autoack=true` actually means that
we are setting the `noAck` option.


When noAck is set, you must not call the following method:





```
internalChannel.basicAck(envelope.getDeliveryTag(), false);
```


Try to execute the test with the third parameter set to `1`
(the second one is ignored) as follows:





```
java -cp rmqAckTest.jar rmqexample.ConsumerMain 1 1 1 0
```








### There's more...


When optimizing messaging operations, you can obtain performance gains
by acting on application-side optimizations. This is feasible both for
\"small\" and \"large\" messages:


-   If the size of your messages is too small, you can aggregate them
    manually before sending them and unpack them at the receiver side

-   If the size of the messages is too large, you can try to compress
    the message before sending it and decompress it at the consumer side







### See also


Read the article at
<http://www.rabbitmq.com/blog/2012/04/17/rabbitmq-performance-measurements-part-1/>
and
<http://www.rabbitmq.com/blog/2012/04/25/rabbitmq-performance-measurements-part-2/>
to understand how important every single parameter is.



Using different distribution tools
----------------------------------------------------



When the application  needs performance, you have to
choose the right distribution tool. In this example, we will show the
differences between publishing a message between a mirrored queue and
non-mirrored one.





### Getting ready


You need Java 1.7 or higher and Apache Maven.






### How to do it...


You can use the source code from the [*Improving bandwidth*]
recipe, then you have to create a RabbitMQ cluster with two nodes.






### How it works...


A cluster using HA mirrored queues is slower than a single broker.
Higher the number of mirroring servers, slower the application will be
because the producer can send more messages only after the message being
sent is stored to all the mirrors.


### Note

That's not as bad as it might seem. On one side, the distribution
toward the nodes of the cluster is performed in parallel, so the
overhead does not grow linearly with the number of nodes. On the other
side, the replication is usually limited to two or three replicas at the
most.

We performed a test using the following environment:


-   <https://www.digitalocean.com/> as cloud

-   Two Debian machines in a RabbitMQ cluster, as shown in the following
    screenshot:

    
    ![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_04.jpg)
    

-   One Debian  machine with the same
    characteristics as the Java client


The tests performed are as follows:


-   [**Test 1**]: Create a mirror using the
    configuration, as shown in the following screenshot:

    ![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_05.jpg)
    

    So. the cluster will mirror all queues with the `perf_`
    prefix.

    The producer is run with the following command:



    ```
    java -cp rmqAckTest.jar rmqexample.ProducerMain 1 100000 640
    ```
    

    The consumer is run with the following command:



    ```
    java -cp rmqAckTest.jar rmqexample.ConsumerMain 1 0 0 0
    ```
    

    The clients exchange messages through the
    `perf_queue_08/03` queue, on which the observed
    performance is as follows:

    
    ![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_06.jpg)
    

-   [**Test 2**]: We removed the HA policy and tried again. The
    result, in this case, was similar to the following screenshot:

    
    ![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_08_07.jpg)
    

-   [**Conclusion**]: By using small-sized messages, we have
    amplified the differences. For larger messages, they are much less
    evident. In Test 2, we have observed about 2.000 mgs/s more than
    Test 1, but as you can see, the rate dropped because the producer is
    faster than the consumer.



### Tip

As a general rule, high availability has a negative impact on
performance. So, whenever it's not mandatory, it's better to leave it
off.


In this example, we  have gone to try the highest
performance and in this context, we have seen the impact of queue
mirroring. If we need a level of replication, but without the strict
requirements of mirroring, it is possible to use a shovel plugin or
simply publish the message to two independent brokers in parallel. In
this example, the messages aren't persistent and don't use the
[**tx-transaction**].


### Tip

tx-transactions kill the performance, especially when you try to commit
each single message, because it has to wait the disk flush time for each
message.







### There's more...


Finding the right compromise between performance and reliability is very
hard because there are lots of variables inside a distributed
application. A typical error is trying to optimize each single
application flow losing scalability or eventually high-availability
benefits. This lab presents extreme situations, but as we have seen,
there are margins for improvement.






### See also


Performance is a hot topic in the RabbitMQ mailing list. You can search
and find a lot of useful information in the archives at
<http://rabbitmq.markmail.org/>.

