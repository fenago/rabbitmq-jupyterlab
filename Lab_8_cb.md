

[]{#ch08}Chapter 8. Performance Tuning for RabbitMQ {#chapter-8.-performance-tuning-for-rabbitmq .title}
---------------------------------------------------

</div>

</div>
:::

In this chapter we will cover:

::: {.itemizedlist}
-   Multithreading and queues

-   System tuning

-   Improving bandwidth

-   Using different distribution tools



[]{#ch08lvl1sec68}Introduction {#introduction .title style="clear: both"}
------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

There are no standard RabbitMQ tuning guidelines because different
applications should be optimized in different ways.

Very often the application needs to be optimized on the client side:

::: {.itemizedlist}
-   CPU-intensive applications can be optimized by running one thread
    for each CPU core

-   I/O-intensive applications can be optimized by running many threads
    per core in order to hide implicit latencies
:::

In both cases[]{#id385 .indexterm} messaging is a perfect fit. In order
to optimize the network transfer rate, the AMQP standard prescribes that
messages are transferred in bunches, and then consumed one by one by the
client (refer to [Chapter
1](https://subscription.packtpub.com/book/application_development/9781849516501/1){.link},
[*Working with AMQP*]{.emphasis}).

RabbitMQ allows multithreaded applications to consume messages
efficiently; this is covered in the [*Multithreading and
queues*]{.emphasis} recipe.

Another frequent use case is when RabbitMQ is at the foundation of a
distributed application serving a large number of clients. In this case,
it\'s more realistic that the bottleneck is the broker and not the
client application.

In this case, it\'s important that our broker has one characteristic,
that is, scalability.

When the number of clients outgrow the current maximum capacity, it\'s
sufficient to add one or more nodes to the RabbitMQ cluster to
distribute the load and improve the total throughput.

Why to optimize then? Well, to reduce cost. The cost of the hardware,
electric power, cooling, or cloud computing resources.

In this chapter, we will talk about the RabbitMQ performance, showing
some tips to improve the performance on both the client side and
eventually modifying the broker parameters.



[]{#ch08lvl1sec70}System tuning {#system-tuning .title style="clear: both"}
-------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In this recipe, we will []{#id395 .indexterm}appreciate some steps
useful to obtain the maximum performance from RabbitMQ. We will cover
the following topics:

::: {.itemizedlist}
-   The `vm_memory_high_watermark`{.literal} configuration
    (<http://www.rabbitmq.com/memory.html>)

-   Erlang [**High Performance Erlang**]{.strong} ([**HiPE**]{.strong})
    (<http://erlang.org/doc/apps/hipe/>)
:::

The `vm_memory_high_watermark`{.literal} configuration[]{#id396
.indexterm} is the maximum percentage of the system memory used to cache
messages before they are consumed or cached to the disk.

Before the limit is reached, by default, at fifty percent of
`vm_memory_high_watermark`{.literal}, (or properly setting the
`vm_memory_high_watermark_paging_ratio`{.literal} parameter, set to
`0.5`{.literal} by default), RabbitMQ will start to move messages from
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
`Chapter08/Recipe02`{.literal}.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec239}Getting ready {#getting-ready .title}

</div>

</div>
:::

To try this recipe, you need to start with RabbitMQ and have the
management plugin installed. Then, you need Java 1.7 or higher and
Apache maven.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec240}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

In order to obtain the []{#id397 .indexterm}maximum performance from
RabbitMQ, you can perform the following steps:

::: {.orderedlist}
1.  Configure the watermark using:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl set_vm_memory_high_watermark 0.6 
    ```
    :::

    [**Or directly in the rabbitmq.config file using:**]{.strong}

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    [{rabbit, [{vm_memory_high_watermark, 0.6}]}].
    ```
    :::

2.  Change the Linux `ulimit`{.literal} parameter modifying the
    `/etc/default/rabbitmq-server`{.literal} file. Then, you can improve
    RabbitMQ itself by using HiPE.

3.  Install the latest version of Erlang from
    <http://www.erlang.org/download.html>.

4.  Install HiPE in your system.

5.  Check that HiPE is correctly activated; if not, you need to install
    Erlang from the sources and activate it.

6.  Activate Erlang HiPE in the RabbitMQ configuration file. Create the
    `rabbitmq.config`{.literal} file with this option or add it if the
    file already exists:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    [
      {rabbit, [{hipe_compile, true}]}
    ].
    ```
    :::

7.  Restart RabbitMQ.

8.  Check that in the RabbitMQ log file there is not a warning showing
    that HiPE has not been activated:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    =WARNING REPORT==== 6-Oct-2013::00:38:23 ===Not HiPE compiling: HiPE not found in this Erlang installation.
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec241}How it works... {#how-it-works .title}

</div>

</div>
:::

The watermark is the maximum memory used by RabbitMQ, by default it\'s
0.4 which means 40 percent of the installed physical memory. When the
memory reaches the watermark the broker stops accepting new connections
and messages. The watermark value is approximate; in some cases it could
be overcome by the default 40 percent. Anyway, when the server has lots
of RAM, you can increase the value, for example, to 60 percent, just to
tolerate the spikes. With `rabbitmqctl`{.literal} the change is
temporary; when you modify the `rabbitmq.config`{.literal} file, the
option is set permanently.

The `ulimit`{.literal} parameter[]{#id398 .indexterm} by default is
`1024`{.literal}. Increase the value to increase the number of files and
of sockets available to RabbitMQ.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip46}Tip {#tip .title}

A too high value could impact negatively the system. Read about the
`ulimit`{.literal} parameter at <https://wiki.debian.org/Limits>.
:::

Using Erlang []{#id399 .indexterm}HiPE is currently considered
experimental. If it works, we can use it. In case the system is
unstable, you need to disable it.

However, using it you can obtain a consistent 40 percent of CPU usage
improvement of the RabbitMQ server, in case this is your bottleneck. For
example, in the following screenshot, you can see the behavior of the
broker in a standard configuration with a producer and a []{#id400
.indexterm}consumer on localhost:

::: {.mediaobject}
![](4_files/6501OS_08_08.jpg)
:::

In this example, we have run both the producer and the consumer on the
localhost, sending 32 byte messages for 300 seconds, letting the
consumer consume all the messages in real time.

After HiPE has been activated, as shown with details in the following
screenshot, the same test behaves considerably better:

::: {.mediaobject}
![](4_files/6501OS_08_09.jpg)
:::

Before you activate HiPE in the RabbitMQ configuration file, you can
check if your local Erlang installation has it by just invoking the
`erl`{.literal} command as follows:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
# erlErlang R15B03 (erts-5.9.3.1) [source] [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]

EshellV5.9.3.1  (abort with ^G)1>
```
:::

In case HiPE is[]{#id401 .indexterm} present, you will see
`[hipe]`{.literal} among the options shown at startup.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip47}Tip {#tip-1 .title}

In Debian wheeze systems, once the latest Erlang version from the
Erlang-site is downloaded, you can add the HiPE module by installing:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
apt-get install erlang-base-hipe
```
:::

The other distributions have similar packages available as well.
:::

Otherwise, you need to install it from an external package or from the
Erlang source code by downloading it from
<http://www.erlang.org/download.html> and installing it; remember to
specify the `--enable-hipe`{.literal} option at the
`configure`{.literal} step.

Once RabbitMQ has been configured too (in step 6), you will notice that
the restart of the server will take a long time; typically several
minutes.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip49}Tip {#tip-2 .title}

In most of the Linux distributions, the default RabbitMQ configuration
file is placed at `/etc/rabbitmq/rabbitmq.config`{.literal}.
:::

At this point, the RabbitMQ broker is HiPE-activated. The most demanding
parts are not interpreted anymore but compiled at startup into native
machine code.

You can further check []{#id402 .indexterm}that in the log file you
don\'t see any message as follows:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
=WARNING REPORT==== 6-Oct-2013::00:38:23 ===Not HiPE compiling: HiPE not found in this Erlang installation.
```
:::

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip50}Tip {#tip-3 .title}

By default, on Linux RabbitMQ, log files are placed in
`/var/log/rabbitmq`{.literal}. You can find more information in [Chapter
12](https://subscription.packtpub.com/book/application_development/9781849516501/12){.link},
[*Managing RabbitMQ Error Conditions*]{.emphasis}.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec242}There\'s more... {#theres-more .title}

</div>

</div>
:::

Since HiPE is an experimental option, we discourage its usage from the
beginning, given that usually the optimization effort needs to address
the application side optimization and scalability.

However, by enabling it, you can reduce the CPU usage and power
consumption of your servers; so this is an option you can consider when
optimizing your architecture.



[]{#ch08lvl1sec71}Improving bandwidth {#improving-bandwidth .title style="clear: both"}
-------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Using [**noAck**]{.strong} flag[]{#id403 .indexterm} and[]{#id404
.indexterm} managing the [**prefetch**]{.strong} parameter[]{#id405
.indexterm} is another client-side way to improve the performance and
the bandwidth. Both noAck and prefetch are used by the consumers.

In this example, we are going to create one producer and one consumer
using these parameters. You can find the source code at
`Chapter08/Recipe03`{.literal}.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec243}Getting ready {#getting-ready .title}

</div>

</div>
:::

You need Java 1.7 or higher and Apache maven.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec244}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

We skip the producer code because it\'s the same one shown in the
[*Multithreading and queues*]{.emphasis} recipe. We still use the
`ReliableClient`{.literal} class as the base class. Let\'s see the
consumer by performing the following steps:

::: {.orderedlist}
1.  Create a maven project and add the RabbitMQ client dependency.

2.  Create a consumer main class, which reads from `args[]`{.literal} to
    manage the consumer with the following four parameters:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    threadNumber = Integer.valueOf(args[0]);
    prefetchcount = Integer.valueOf(args[1]);
    autoAck = (Integer.valueOf(args[2]) != 0);
    print_thread_consumer
    = (Integer.valueOf(args[3]) != 0);
    ```
    :::

3.  Create a consumer that extends the `ReliableClient`{.literal} class,
    and then set the `prefetch`{.literal} and `noAck`{.literal}
    parameters:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    internalChannel.basicQos(prefetch_count);
    internalChannel.basicConsume(Constants.queue, autoAck..
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec245}How it works... {#how-it-works .title}

</div>

</div>
:::

The prefetch-size is ignored if the `noAck`{.literal} option is set, so
we divide the recipe in the following two sections:

::: {.itemizedlist}
-   Prefetch

-   noAck
:::

The aim is to []{#id406 .indexterm}understand how to manage the
client-side parameters to improve the performance and the bandwidth.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

#### []{#ch08lvl3sec08}Prefetch {#prefetch .title}

</div>

</div>
:::

To set the[]{#id407 .indexterm} prefetch, use
`basicQos(prefetch_count)`{.literal} (refer to step 3).

We have already seen the `channel QoS`{.literal} parameter in [Chapter
1](https://subscription.packtpub.com/book/application_development/9781849516501/1){.link},
Working with AMQP, [*Distributing Messages to Many
Consumers*]{.emphasis}, where the messages are acknowledged one by one,
in order to correctly load balance the messages.

The prefetch count is the maximum number of unacknowledged messages: a
large value will let the client prefetch many messages in advance
without waiting for the acks of the messages being processed.

As stated at the beginning of the chapter, there is not a one-for-all
rule when optimizing. In fact, improving the prefetch count can be
counterproductive when the per message processing time is important, and
we need to distribute and balance the processing.

Well firstly, maven will compile using the following command:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
mvn clean compile assembly:single
```
:::

Then, maven will create the `rmqAckTest.jar`{.literal} package.

You can now try the[]{#id408 .indexterm} example, by changing the
parameters to see how the message rates change. We made a test using a
MacBook pro Dual Core, 4 GB RAM using the following parameters:

::: {.itemizedlist}
-   For the producer, we can run the following command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    java -cp rmqAckTest.jar rmqexample.ProducerMain 1 100000 64000
    ```
    :::

-   For the consumer, we can run the following two tests:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    java -cp rmqAckTest.jar rmqexample.ConsumerMain 2 50 0 0
    java -cp rmqAckTest.jar rmqexample.ConsumerMain 2 1 0 0
    ```
    :::
:::

The producer[]{#id409 .indexterm} uses `1`{.literal} thread for 100
seconds and 64000 bytes as the message size.

The consumer uses `2`{.literal} threads, the prefetch count in
`Test1`{.literal} is `50`{.literal} and in `Test2`{.literal} is
`1`{.literal}, `autoAck`{.literal} is set to `false`{.literal}, and
without printing the thread number on the console.

The results for these two tests were as follows:

::: {.mediaobject}
![](5_files/6501OS_08_02.jpg)

::: {.caption}
Test1
:::
:::

::: {.mediaobject}
![](5_files/6501OS_08_03.jpg)

::: {.caption}
Test2
:::
:::

As you can see, the []{#id410 .indexterm}prefetch count can make the
difference, especially when you have more consumers bound to the same
queue.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

#### []{#ch08lvl3sec09}NoAck {#noack .title}

</div>

</div>
:::

To set noAck, []{#id411 .indexterm}use
`basicConsume(Constants.queue, true)`{.literal}. The parameter is useful
when you have a stream data or when it doesn\'t matter to send the acks
manually.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip51}Tip {#tip .title}

In the Java API, the name of the `ack`{.literal} parameter of
`Channel.basicConsume`{.literal} as a Boolean, `autoack`{.literal}, is
quite misleading; setting `autoack=true`{.literal} actually means that
we are setting the `noAck`{.literal} option.
:::

When noAck is set, you must not call the following method:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
internalChannel.basicAck(envelope.getDeliveryTag(), false);
```
:::

Try to execute the test with the third parameter set to `1`{.literal}
(the second one is ignored) as follows:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
java -cp rmqAckTest.jar rmqexample.ConsumerMain 1 1 1 0
```
:::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec246}There\'s more... {#theres-more .title}

</div>

</div>
:::

When optimizing messaging operations, you can obtain performance gains
by acting on application-side optimizations. This is feasible both for
\"small\" and \"large\" messages:

::: {.itemizedlist}
-   If the size of your messages is too small, you can aggregate them
    manually before sending them and unpack them at the receiver side

-   If the size of the messages is too large, you can try to compress
    the message before sending it and decompress it at the consumer side
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec247}See also {#see-also .title}

</div>

</div>
:::

Read the article at
<http://www.rabbitmq.com/blog/2012/04/17/rabbitmq-performance-measurements-part-1/>
and
<http://www.rabbitmq.com/blog/2012/04/25/rabbitmq-performance-measurements-part-2/>
to understand how important every single parameter is.



[]{#ch08lvl1sec72}Using different distribution tools {#using-different-distribution-tools .title style="clear: both"}
----------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

When the application []{#id412 .indexterm}needs performance, you have to
choose the right distribution tool. In this example, we will show the
differences between publishing a message between a mirrored queue and
non-mirrored one.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec248}Getting ready {#getting-ready .title}

</div>

</div>
:::

You need Java 1.7 or higher and Apache Maven.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec249}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

You can use the source code from the [*Improving bandwidth*]{.emphasis}
recipe, then you have to create a RabbitMQ cluster with two nodes.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec250}How it works... {#how-it-works .title}

</div>

</div>
:::

A cluster using HA mirrored queues is slower than a single broker.
Higher the number of mirroring servers, slower the application will be
because the producer can send more messages only after the message being
sent is stored to all the mirrors.

::: {.note style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#note22}Note {#note .title}

That\'s not as bad as it might seem. On one side, the distribution
toward the nodes of the cluster is performed in parallel, so the
overhead does not grow linearly with the number of nodes. On the other
side, the replication is usually limited to two or three replicas at the
most, as we saw in [Chapter
7](https://subscription.packtpub.com/book/application_development/9781849516501/7){.link},
[*Developing High-availability Applications*]{.emphasis}.
:::

We performed a test using the following environment:

::: {.itemizedlist}
-   <https://www.digitalocean.com/> as cloud

-   Two Debian machines in a RabbitMQ cluster, as shown in the following
    screenshot:

    ::: {.mediaobject}
    ![](6_files/6501OS_08_04.jpg)
    :::

-   One Debian[]{#id413 .indexterm} machine with the same
    characteristics as the Java client
:::

The tests performed are as follows:

::: {.itemizedlist}
-   [**Test 1**]{.strong}: Create a mirror (as we saw in [Chapter
    7](https://subscription.packtpub.com/book/application_development/9781849516501/7){.link},
    [*Developing High-availability Applications*]{.emphasis}) using the
    configuration, as shown in the following screenshot:

    ::: {.mediaobject}
    ![](6_files/6501OS_08_05.jpg)
    :::

    So. the cluster will mirror all queues with the `perf_`{.literal}
    prefix.

    The producer is run with the following command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    java -cp rmqAckTest.jar rmqexample.ProducerMain 1 100000 640
    ```
    :::

    The consumer is run with the following command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    java -cp rmqAckTest.jar rmqexample.ConsumerMain 1 0 0 0
    ```
    :::

    The clients exchange messages through the
    `perf_queue_08/03`{.literal} queue, on which the observed
    performance is as follows:

    ::: {.mediaobject}
    ![](6_files/6501OS_08_06.jpg)
    :::

-   [**Test 2**]{.strong}: We removed the HA policy and tried again. The
    result, in this case, was similar to the following screenshot:

    ::: {.mediaobject}
    ![](6_files/6501OS_08_07.jpg)
    :::

-   [**Conclusion**]{.strong}: By using small-sized messages, we have
    amplified the differences. For larger messages, they are much less
    evident. In Test 2, we have observed about 2.000 mgs/s more than
    Test 1, but as you can see, the rate dropped because the producer is
    faster than the consumer.
:::

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip52}Tip {#tip .title}

As a general rule, high availability has a negative impact on
performance. So, whenever it\'s not mandatory, it\'s better to leave it
off.
:::

In this example, we []{#id414 .indexterm}have gone to try the highest
performance and in this context, we have seen the impact of queue
mirroring. If we need a level of replication, but without the strict
requirements of mirroring, it is possible to use a shovel plugin or
simply publish the message to two independent brokers in parallel. In
this example, the messages aren\'t persistent and don\'t use the
[**tx-transaction**]{.strong}.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip53}Tip {#tip-1 .title}

tx-transactions kill the performance, especially when you try to commit
each single message, because it has to wait the disk flush time for each
message.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec251}There\'s more... {#theres-more .title}

</div>

</div>
:::

Finding the right compromise between performance and reliability is very
hard because there are lots of variables inside a distributed
application. A typical error is trying to optimize each single
application flow losing scalability or eventually high-availability
benefits. This chapter presents extreme situations, but as we have seen,
there are margins for improvement.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch08lvl2sec252}See also {#see-also .title}

</div>

</div>
:::

Performance is a hot topic in the RabbitMQ mailing list. You can search
and find a lot of useful information in the archives at
<http://rabbitmq.markmail.org/>.

