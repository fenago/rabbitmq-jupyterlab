

[]{#ch12}Chapter 12. Managing RabbitMQ Error Conditions {#chapter-12.-managing-rabbitmq-error-conditions .title}
-------------------------------------------------------

</div>

</div>
:::

In this chapter, we will cover the following topics:

::: {.itemizedlist}
-   Monitoring RabbitMQ\'s behavior

-   Using RabbitMQ to troubleshoot itself

-   Tracing RabbitMQ\'s ongoing activity

-   Debugging RabbitMQ\'s messages

-   What to do when RabbitMQ fails to restart

-   Debugging using Wireshark



[]{#ch12lvl1sec89}Introduction {#introduction .title style="clear: both"}
------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Whenever we develop an application, it\'s a common practice to develop a
diagnostic infrastructure. This can be based on log files, SNMP traps,
and many others.

RabbitMQ provides both standard log files and a built-in messaging-based
troubleshooting solution.

We will see how to use these features in the first three recipes.

Sometimes, there are problems that prevent RabbitMQ from starting. In
this case, it\'s mandatory to fix the problem directly on the server
machine where the issue persists, and to reset the broker. We\'ll see
this in the [*What to do when RabbitMQ fails to restart*]{.emphasis}
recipe.

However, debugging messages is a part of application development too. In
this case, we need to know the exact information exchanged between
RabbitMQ and its clients. It is possible to use a proxy built-in tool,
part of the Java client API (see the [*Debugging RabbitMQ\'s
messages*]{.emphasis} recipe) or to use an advanced network monitor to
examine the traffic, as we will see in the [*Debugging using
Wireshark*]{.emphasis} recipe.



[]{#ch12lvl1sec90}Monitoring RabbitMQ\'s behavior {#monitoring-rabbitmqs-behavior .title style="clear: both"}
-------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In order to []{#id554 .indexterm}check the correct behavior of RabbitMQ,
it is useful to have a []{#id555 .indexterm}monitoring tool, especially
when dealing with a cluster.

There are many different tools, commercial and freeware, which help to
keep things under control in distributed systems such as Nagios[]{#id556
.indexterm} and []{#id557 .indexterm}Zabbix.

In this recipe, we will show how to configure the RabbitMQ plugin for
[]{#id558 .indexterm} [**Ganglia**]{.strong}
(<http://sourceforge.net/apps/trac/ganglia/wiki/ganglia_quick_start>).

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec309}Getting ready {#getting-ready .title}

</div>

</div>
:::

In order to run this recipe, you need to have RabbitMQ configured with
the management plugin enabled.

You need to install and configure Ganglia too. In this recipe, we have
used version 3.6.0.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec310}How to do it\... {#how-to-do-it... .title}

</div>

</div>
:::

In order to see RabbitMQ statistics within Ganglia monitoring graphs,
you need to perform the following steps:

::: {.orderedlist}
1.  Install and configure Ganglia using `yum`{.literal} or
    `apt-get`{.literal} depending on your Linux distribution. You will
    need the following packages:

    ::: {.itemizedlist}
    -   `ganglia-gmetad`{.literal}

    -   `ganglia-gmond`{.literal}

    -   `ganglia-gmond-python`{.literal}

    -   `ganglia-web`{.literal}

    -   `ganglia`{.literal}
    :::

2.  Copy the Python Ganglia monitor plugin into
    `/usr/lib64/ganglia/python_modules`{.literal} from
    <https://github.com/ganglia/gmond_python_modules/blob/master/rabbit/python_modules/rabbitmq.py>.

3.  Copy the Python Ganglia configuration file into
    `/etc/ganglia/conf.d`{.literal} from
    <https://github.com/ganglia/gmond_python_modules/blob/master/rabbit/conf.d/rabbitmq.pyconf>.

4.  Check the configuration file for the correct parameters. In
    particular, you probably need to fix the following entry by leaving
    only the default `vhost`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    paramvhost {
    value = "/"
        }
    ```
    :::

5.  If it is already running, restart `gmond`{.literal} using:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    service gmond restart
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec311}How it works\... {#how-it-works... .title}

</div>

</div>
:::

By going through this recipe, you will be able to monitor RabbitMQ from
within the Ganglia environment.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip66}Tip {#tip .title}

For basic troubleshooting, you can have a look at the log files that, by
default, are saved into `/var/log/rabbitmq`{.literal}.
:::

Once it\'s up and[]{#id559 .indexterm} running, you will be able to
access both system-wide[]{#id560 .indexterm} information and information
about RabbitMQ queues and nodes from the same web interface, as you can
see in the following screenshot:

::: {.mediaobject}
![](3_files/6501OS_12_01.jpg)
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec312}There\'s more\... {#theres-more... .title}

</div>

</div>
:::

Ganglia[]{#id561 .indexterm} is a widespread solution for cluster
monitoring, but not the only one.

Other typical solutions with RabbitMQ configurations that are available
are Nagios[]{#id562 .indexterm}
([www.nagios.org](http://www.nagios.org/){.ulink}), Zabbix[]{#id563
.indexterm} ([www.zabbix.com](http://www.zabbix.com/){.ulink}), and
Puppet[]{#id564 .indexterm}
([puppetlabs.com](http://puppetlabs.com/){.ulink}).



[]{#ch12lvl1sec91}Using RabbitMQ to troubleshoot itself {#using-rabbitmq-to-troubleshoot-itself .title style="clear: both"}
-------------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

As mentioned in the []{#id565 .indexterm}previous recipe, we can monitor
the RabbitMQ behavior by accessing its log files in quite a conventional
way.

It is also possible to access the same kind of information using
RabbitMQ itself, by informing a generic AMQP client of the broker
activity.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec313}Getting ready {#getting-ready .title}

</div>

</div>
:::

To run this recipe, we need RabbitMQ up and running and the Java client
library.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec314}How to do it\... {#how-to-do-it... .title}

</div>

</div>
:::

To consume log messages, you can execute the Java main function in
`Consumer.java`{.literal}. You can find this in the book source archive
in the directory `Chapter12/Recipe02/Java/src/rmqexample`{.literal}. In
the following, we highlight the main steps:

::: {.orderedlist}
1.  Create a temporary-anonymous queue and bind it to the AMQP log
    exchange:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    String tmpQueue = channel.queueDeclare().getQueue();
    channel.queueBind(tmpQueue,"amq.rabbitmq.log","#");
    ```
    :::

2.  In the consumer callback (`ActualConsumer.java`{.literal}), retrieve
    the message and the routing key of each message and print them:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    String routingKey = envelope.getRoutingKey();
    String message = new String(body);
    System.out.println(routingKey + ": " + message);
    ```
    :::

3.  At this point, you can run any RabbitMQ operation on the broker, and
    you will see it logged to the standard output.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec315}How it works\... {#how-it-works... .title}

</div>

</div>
:::

The RabbitMQ log exchange, `amq.rabbitmq.log`{.literal}, is a topic
[]{#id566 .indexterm}exchange to which RabbitMQ itself publishes its log
messages.

In our example code, we consume messages from all the topics using the
`#`{.literal} wildcard.

For example, by running another code that runs two connections to the
same broker and by aborting it, we get the following output:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
info: accepting AMQP connection <0.2737.0> (127.0.0.1:54698 -> 127.0.0.1:5672)
info: accepting AMQP connection <0.2753.0> (127.0.0.1:54699 -> 127.0.0.1:5672)
warning: closing AMQP connection <0.2737.0> (127.0.0.1:54698 -> 127.0.0.1:5672):
connection_closed_abruptly
warning: closing AMQP connection <0.2753.0> (127.0.0.1:54699 -> 127.0.0.1:5672):
connection_closed_abruptly
```
:::

It is worth to note []{#id567 .indexterm}that the information,
`info`{.literal} and `warning`{.literal} reported here is not a part of
the messages themselves, but are the routing keys that we are printing
at the beginning of each message (step 2 of the preceding steps).

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip67}Tip {#tip .title}

In case we just want to receive warning and error messages, we can
subscribe to the corresponding topics only.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec316}There\'s more\... {#theres-more... .title}

</div>

</div>
:::

By default, the log exchange, `amq.rabbitmq.log`{.literal}, is created
in the vhost `/`{.literal}. It\'s possible to customize its location by
defining `default_vhost`{.literal} in the RabbitMQ configuration file.



[]{#ch12lvl1sec92}Tracing RabbitMQ\'s ongoing activity {#tracing-rabbitmqs-ongoing-activity .title style="clear: both"}
------------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Sometimes, []{#id568 .indexterm}we need to trace all the messages that
are being received and delivered by RabbitMQ, to analyze and debug
unexpected application behavior.

RabbitMQ provides the so-called [**firehose**]{.strong} tracing
tool[]{#id569 .indexterm} to have such information available.

The tracing activity can be []{#id570 .indexterm}enabled and disabled at
runtime, and it[]{#id571 .indexterm} should be used just for debugging
since it imposes an overhead on the broker activity.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec317}Getting ready {#getting-ready .title}

</div>

</div>
:::

To run this recipe, we need RabbitMQ up and running and the Java client
library.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec318}How to do it\... {#how-to-do-it... .title}

</div>

</div>
:::

RabbitMQ sends trace messages using the same mechanism used for log
messages; so, the example code is very similar to the one of the
previous recipe.

To consume []{#id572 .indexterm}trace messages, you can execute the Java
main function []{#id573 .indexterm}in `Consumer.java`{.literal} that you
can find in the book source archive in the directory
`Chapter12/Recipe02/Java/src/rmqexample`{.literal}. Here, we highlight
the main steps:

::: {.orderedlist}
1.  Create a temporary queue and bind it to the AMQP log exchange:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    String tmpQueue = channel.queueDeclare().getQueue();
    channel.queueBind(tmpQueue,"amq.rabbitmq.trace","#");
    ```
    :::

2.  In the consumer callback (`ActualConsumer.java`{.literal}), retrieve
    the messages with some more information for each one and print them
    using the following code:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    String routingKey = envelope.getRoutingKey();
    String message = new String(body);
    Map<String,Object> headers = properties.getHeaders();
    LongStringexchange_name = (LongString) 
    headers.get("exchange_name");
    LongString node = (LongString) headers.get("node");
    ...
    ```
    :::

3.  Activate []{#id574 .indexterm}firehose by invoking it from the root
    user (Linux) or on the RabbitMQ command console (Windows). Use the
    following command to activate firehose:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbimqctl trace_on
    ```
    :::

4.  At this point, you can start producing and sending messages to the
    broker, and you will see them traced to the standard output.

5.  Deactivate firehose by invoking the command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbimqctl trace_off
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec319}How it works\... {#how-it-works... .title}

</div>

</div>
:::

The `amq.rabbit.trace`{.literal} topic exchange, by default, does not
receive any messages, but once firehose is activated (step 3 of the
previous steps), all the messages traveling through the broker will be
copied to it by following specific rules:

::: {.itemizedlist}
-   Messages entering the broker are published with the routing key
    `publish.exchange-name`{.literal}, where `exchange-name`{.literal}
    is the name of the exchange where the message was originally
    published to.

-   Messages leaving the broker are published with the routing key
    `deliver.queue-name`{.literal}, where `queue-name`{.literal} is the
    name of the queue where the message has been originally consumed
    from.

-   The body of the message is copied from the original one.

-   The metadata of the original messages are inserted in the header
    properties of the copied message. In step 2 of the previous numbered
    bullets, we have seen how to retrieve the[]{#id575 .indexterm}
    exchange name to which the message was originally delivered, but
    it\'s possible to get all the original information, that is, find
    all the available []{#id576 .indexterm}fields inserted into the
    message properties at the firehose official []{#id577
    .indexterm}documentation link at
    <http://www.rabbitmq.com/firehose.html>.



[]{#ch12lvl1sec93}Debugging RabbitMQ\'s messages {#debugging-rabbitmqs-messages .title style="clear: both"}
------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Sometimes, it is useful[]{#id578 .indexterm} to just have an idea of the
messages that are really[]{#id579 .indexterm} traveling through a broker
by just logging all of them to the standard output.

It is possible to trace those messages using a simple application
provided with the RabbitMQ Java client.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec320}Getting ready {#getting-ready .title}

</div>

</div>
:::

To run this recipe, you need to have RabbitMQ up and running on the
standard port `5672`{.literal} and the RabbitMQ Java client library.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec321}How to do it\... {#how-to-do-it... .title}

</div>

</div>
:::

RabbitMQ includes a tracing utility in the Java client library that you
can put in action by following these steps.

::: {.orderedlist}
1.  Download the latest version of the RabbitMQ Java client library from
    <http://www.rabbitmq.com/java-client.html>.

2.  Unpack it and enter its directory.

3.  Run the Java tracer by running:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    ./runjava.sh com.rabbitmq.tools.Tracer
    ```
    :::

4.  Run the Java client that is to be debugged and that connects to port
    `5673`{.literal}. For this recipe, we will use another Java tool
    included in the Java client library, by invoking:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    ./runjava.sh com.rabbitmq.examples.PerfTest -h amqp://localhost:5673 -C 1 -D 1
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec322}How it works\... {#how-it-works... .title}

</div>

</div>
:::

The Java tracing tool is a simple AMQP proxy; by default, it listens on
port `5673`{.literal} and forwards all the requests to the RabbitMQ
broker listening by default on `localhost`{.literal} at port
`5672`{.literal}.

All the messages produced or consumed, as well as the AMQP operations,
are all logged to a standard output.

To run the recipe at step 4 of the previous steps we have used another
tool that is included in the RabbitMQ Java []{#id580 .indexterm}client
library that can be used to perform a stress test with RabbitMQ.

In our case, we have []{#id581 .indexterm}just limited it to produce one
message (`-C 1`{.literal}) and consume it (`-D 1`{.literal}).

::: {.note style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#note37}Note {#note .title}

The tracing tool is available in the Java client API only.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec323}There\'s more\... {#theres-more... .title}

</div>

</div>
:::

It\'s possible to pass some more parameters to the Java tracing program
using the following code:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
./runjava.sh com.rabbitmq.tools.Tracer listenPort connectHost connectPort 
```
:::

In the preceding code `listenPort`{.literal} refers to the port where
the tracer is listening (default: `5673`{.literal}),
`connectHost`{.literal}/`connectPort`{.literal} (default:
`localhost/5672`{.literal}) are the host and the port where it connects
to and forwards the requests it receives.

You can find all `PerfTest`{.literal} available options using:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
./runjava.sh com.rabbitmq.examples.PerfTest --help
```
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec324}See also {#see-also .title}

</div>

</div>
:::

You can find the documentation of the Java tracing tool and PerfTest at
<http://www.rabbitmq.com/java-tools.html>.



[]{#ch12lvl1sec94}What to do when RabbitMQ fails to restart {#what-to-do-when-rabbitmq-fails-to-restart .title style="clear: both"}
-----------------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Occasionally, RabbitMQ fails to restart. This can be an important issue
in case the broker contains persistent data; otherwise, it\'s enough to
reset the broker persistent state.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec325}Getting ready {#getting-ready .title}

</div>

</div>
:::

To run this[]{#id582 .indexterm} recipe, you just need a test RabbitMQ
broker.

::: {.note style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#note38}Note {#note .title}

We are going to destroy all the previously defined data---avoid using a
production instance.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec326}How to do it\... {#how-to-do-it... .title}

</div>

</div>
:::

To clean-up[]{#id583 .indexterm} RabbitMQ, it\'s enough to follow these
simple steps:

::: {.orderedlist}
1.  Stop RabbitMQ if it is running.

2.  Locate the []{#id584 .indexterm} [**Mnesia**]{.strong} database
    directory. By default, it\'s `/var/lib/rabbitmq/mnesia`{.literal}
    (Linux) or `%APPDATA%\RabbitMQ\db`{.literal} (Windows).

3.  Delete it recursively.

4.  Restart RabbitMQ.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec327}How it works\... {#how-it-works... .title}

</div>

</div>
:::

The Mnesia database contains all the runtime definitions of RabbitMQ:
queues, exchanges, users, and so on.

By deleting it, (or renaming it in case we want to try to recover some
data, or to eventually fall back in case it\'s possible) RabbitMQ is
reset to the factory defaults; once started, it will create a new Mnesia
database and initialize it with default values.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec328}There\'s more\... {#theres-more... .title}

</div>

</div>
:::

In case the broker fails to start the first time, it is probable that
there is a file permission problem in one of the system directories:
either the Mnesia database directory or the log directory or some
temporary, or custom directories that are specified in the configuration
file.

You can find quite an exhaustive list of cases in the RabbitMQ
troubleshooting page (<http://www.rabbitmq.com/troubleshooting.html>).
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec329}See also {#see-also .title}

</div>

</div>
:::

You can find more information on how to hack Mnesia databases at the
Mnesia API documentation pages
(<http://www.erlang.org/doc/man/mnesia.html>).



[]{#ch12lvl1sec95}Debugging using Wireshark {#debugging-using-wireshark .title style="clear: both"}
-------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In the [*Debugging RabbitMQ\'s messages*]{.emphasis} recipe, we have
seen how to trace messages going to/from RabbitMQ.

However, it is not always possible, or desirable, to stop a running
client (or a RabbitMQ server), modify its connection port, and point it
to a different one; we just want to monitor the messages that are
passing in real-time, impacting the system activity as little as
possible.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip68}Tip {#tip .title}

However, it\'s possible to activate the firehose tracer as seen in the
recipe, [*Tracing RabbitMQ\'s ongoing activity*]{.emphasis}.
:::

Wireshark is a free network analysis tool that has the capability to
decode AMQP messages. This tool can be used either on the client side or
on the server side to monitor the AMQP traffic flow seamlessly.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec330}Getting ready {#getting-ready .title}

</div>

</div>
:::

To exercise this recipe, you need RabbitMQ up and running and the
RabbitMQ Java client library.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec331}How to do it\... {#how-to-do-it... .title}

</div>

</div>
:::

In the following steps, we are[]{#id585 .indexterm} going to see how to
use Wireshark to[]{#id586 .indexterm} trace the AMQP messages:

::: {.orderedlist}
1.  If not already available on your system, download and install
    Wireshark from <http://www.wireshark.org/>. You can also install it
    for your distribution if it is available, for example, with:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    yum install wireshark-gnome
    ```
    :::

2.  Start Wireshark on Linux from the `root`{.literal} user.

3.  Start to capture from the loopback interface.

4.  From a terminal from the Java client library path, run the command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    ./runjava.sh com.rabbitmq.examples.PerfTest -C 1 -D 1
    ```
    :::

5.  Stop the acquisition from the Wireshark GUI and analyze the captured
    AMQP traffic.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec332}How it works\... {#how-it-works... .title}

</div>

</div>
:::

Using Wireshark, it []{#id587 .indexterm}is possible to inspect the AMQP
[]{#id588 .indexterm}traffic exiting or entering a server that is
hosting a RabbitMQ server or client.

In our example, we have captured the network traffic running both the
client and the server on the same machine, thus connecting in
`localhost`{.literal}. That\'s why we were capturing the traffic from
the loopback interface (step 3 of the previous steps).

Otherwise, we should capture the traffic from the network interface,
usually eth0 or something similar.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip69}Tip {#tip-1 .title}

While on Linux, it\'s possible to capture traffic directed to
`localhost`{.literal}; the same does not apply to Windows. In this case,
the client and the server must be on two different machines, and the
capture must be activated on the network interface (either physical or
virtual), thus connecting them.
:::

So, in order to run the Wireshark graphical user interface, in case the
RabbitMQ client and the server run on the same node, you need to select
the loopback interface, as shown in the following screenshot:

::: {.mediaobject}
![](8_files/6501OS_12_02.jpg)
:::

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip70}Tip {#tip-2 .title}

On Linux, when you install the Wireshark package, you usually will have
the command line[]{#id589 .indexterm} interface only,
[**tshark**]{.strong}. To have Wireshark with the GUI installed, you
have to install the appropriate package. For example, on Fedora, you
have to install the `wireshark-gnome`{.literal} package.
:::

Once the AMQP traffic[]{#id590 .indexterm} has travelled through the
loopback interface, it has been captured by Wireshark.

The experiment run in[]{#id591 .indexterm} step 4 of the previous steps
actually starts both a producer and a consumer with two separated
connections.

In order to highlight it, find a packet described as
`Basic.PublishContent-Header`{.literal}, right-click on it, and select
`Follow TCP stream`{.literal}. You can then close the window showing the
payload dialogue between the client and the server. In the main window,
you can now see the network packets that are exchanged between the
client and the server, as shown in the following screenshot:

::: {.mediaobject}
![](8_files/6501OS_12_03.jpg)
:::

In the same way, []{#id592 .indexterm}you can select the traffic exiting
the RabbitMQ []{#id593 .indexterm}server, as shown in the following
screenshot:

::: {.mediaobject}
![](8_files/6501OS_12_04.jpg)
:::

In the previous two screenshots, we have highlighted the AMQP payload of
both messages, but you will find plenty of details in the AMQP traffic,
thanks to the fact that Wireshark includes a very complete AMQP
dissector.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec333}There\'s more... {#theres-more .title}

</div>

</div>
:::

In case RabbitMQ is configured to use SSL and you want to analyze the
encrypted traffic, this is possible under some given conditions by
properly configuring the SSL public/private keys in the Wireshark
configuration.

Find more information at <http://wiki.wireshark.org/SSL>.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch12lvl2sec334}See also {#see-also .title}

</div>

</div>
:::

You can find some references to the Wireshark AMQP dissector at
<http://wiki.wireshark.org/AMQP>.
