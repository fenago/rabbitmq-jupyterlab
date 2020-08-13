
<img align="right" src="./logo-small.png">

Lab. Managing RabbitMQ Error Conditions
-------------------------------------------


In this lab, we will cover the following topics:


-   Monitoring RabbitMQ\'s behavior

-   Using RabbitMQ to troubleshoot itself

-   Tracing RabbitMQ\'s ongoing activity

-   Debugging RabbitMQ\'s messages

-   What to do when RabbitMQ fails to restart

-   Debugging using Wireshark



Introduction
------------------------------



Whenever we develop an application, it\'s a common practice to develop a
diagnostic infrastructure. This can be based on log files, SNMP traps,
and many others.

RabbitMQ provides both standard log files and a built-in messaging-based
troubleshooting solution.

We will see how to use these features in the first three recipes.

Sometimes, there are problems that prevent RabbitMQ from starting. In
this case, it\'s mandatory to fix the problem directly on the server
machine where the issue persists, and to reset the broker. We\'ll see
this in the [*What to do when RabbitMQ fails to restart*]
recipe.

However, debugging messages is a part of application development too. In
this case, we need to know the exact information exchanged between
RabbitMQ and its clients. It is possible to use a proxy built-in tool,
part of the Java client API (see the [*Debugging RabbitMQ\'s
messages*] recipe) or to use an advanced network monitor to
examine the traffic, as we will see in the [*Debugging using
Wireshark*] recipe.



Monitoring RabbitMQ\'s behavior
-------------------------------------------------



In order to  check the correct behavior of RabbitMQ,
it is useful to have a  monitoring tool, especially
when dealing with a cluster.

There are many different tools, commercial and freeware, which help to
keep things under control in distributed systems such as Nagios
 and  Zabbix.

In this recipe, we will show how to configure the RabbitMQ plugin for
  [**Ganglia**]
(<http://sourceforge.net/apps/trac/ganglia/wiki/ganglia_quick_start>).





### Getting ready


In order to run this recipe, you need to have RabbitMQ configured with
the management plugin enabled.

You need to install and configure Ganglia too. In this recipe, we have
used version 3.6.0.






### How to do it\...


In order to see RabbitMQ statistics within Ganglia monitoring graphs,
you need to perform the following steps:


1.  Install and configure Ganglia using `yum` or
    `apt-get` depending on your Linux distribution. You will
    need the following packages:

    
    -   `ganglia-gmetad`

    -   `ganglia-gmond`

    -   `ganglia-gmond-python`

    -   `ganglia-web`

    -   `ganglia`
    

2.  Copy the Python Ganglia monitor plugin into
    `/usr/lib64/ganglia/python_modules` from
    <https://github.com/ganglia/gmond_python_modules/blob/master/rabbit/python_modules/rabbitmq.py>.

3.  Copy the Python Ganglia configuration file into
    `/etc/ganglia/conf.d` from
    <https://github.com/ganglia/gmond_python_modules/blob/master/rabbit/conf.d/rabbitmq.pyconf>.

4.  Check the configuration file for the correct parameters. In
    particular, you probably need to fix the following entry by leaving
    only the default `vhost`:



    ```
    paramvhost {
    value = "/"
        }
    ```
    

5.  If it is already running, restart `gmond` using:



    ```
    service gmond restart
    ```
    







### How it works\...


By going through this recipe, you will be able to monitor RabbitMQ from
within the Ganglia environment.


### Tip

For basic troubleshooting, you can have a look at the log files that, by
default, are saved into `/var/log/rabbitmq`.


Once it\'s up and  running, you will be able to
access both system-wide  information and information
about RabbitMQ queues and nodes from the same web interface, as you can
see in the following screenshot:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_12_01.jpg)







### There\'s more\...


Ganglia  is a widespread solution for cluster
monitoring, but not the only one.

Other typical solutions with RabbitMQ configurations that are available
are Nagios 
([www.nagios.org](http://www.nagios.org/){.ulink}), Zabbix
 ([www.zabbix.com](http://www.zabbix.com/){.ulink}), and
Puppet 
([puppetlabs.com](http://puppetlabs.com/){.ulink}).



Using RabbitMQ to troubleshoot itself
-------------------------------------------------------



As mentioned in the  previous recipe, we can monitor
the RabbitMQ behavior by accessing its log files in quite a conventional
way.

It is also possible to access the same kind of information using
RabbitMQ itself, by informing a generic AMQP client of the broker
activity.





### Getting ready


To run this recipe, we need RabbitMQ up and running and the Java client
library.






### How to do it\...


To consume log messages, you can execute the Java main function in
`Consumer.java`. You can find this in the book source archive
in the directory `Lab12/Recipe02/Java/src/rmqexample`. In
the following, we highlight the main steps:


1.  Create a temporary-anonymous queue and bind it to the AMQP log
    exchange:



    ```
    String tmpQueue = channel.queueDeclare().getQueue();
    channel.queueBind(tmpQueue,"amq.rabbitmq.log","#");
    ```
    

2.  In the consumer callback (`ActualConsumer.java`), retrieve
    the message and the routing key of each message and print them:



    ```
    String routingKey = envelope.getRoutingKey();
    String message = new String(body);
    System.out.println(routingKey + ": " + message);
    ```
    

3.  At this point, you can run any RabbitMQ operation on the broker, and
    you will see it logged to the standard output.







### How it works\...


The RabbitMQ log exchange, `amq.rabbitmq.log`, is a topic
 exchange to which RabbitMQ itself publishes its log
messages.

In our example code, we consume messages from all the topics using the
`#` wildcard.

For example, by running another code that runs two connections to the
same broker and by aborting it, we get the following output:





```
info: accepting AMQP connection <0.2737.0> (127.0.0.1:54698 -> 127.0.0.1:5672)
info: accepting AMQP connection <0.2753.0> (127.0.0.1:54699 -> 127.0.0.1:5672)
warning: closing AMQP connection <0.2737.0> (127.0.0.1:54698 -> 127.0.0.1:5672):
connection_closed_abruptly
warning: closing AMQP connection <0.2753.0> (127.0.0.1:54699 -> 127.0.0.1:5672):
connection_closed_abruptly
```


It is worth to note  that the information,
`info` and `warning` reported here is not a part of
the messages themselves, but are the routing keys that we are printing
at the beginning of each message (step 2 of the preceding steps).


### Tip

In case we just want to receive warning and error messages, we can
subscribe to the corresponding topics only.







### There\'s more\...


By default, the log exchange, `amq.rabbitmq.log`, is created
in the vhost `/`. It\'s possible to customize its location by
defining `default_vhost` in the RabbitMQ configuration file.



Tracing RabbitMQ\'s ongoing activity
------------------------------------------------------



Sometimes,  we need to trace all the messages that
are being received and delivered by RabbitMQ, to analyze and debug
unexpected application behavior.

RabbitMQ provides the so-called [**firehose**] tracing
tool  to have such information available.

The tracing activity can be  enabled and disabled at
runtime, and it  should be used just for debugging
since it imposes an overhead on the broker activity.





### Getting ready


To run this recipe, we need RabbitMQ up and running and the Java client
library.






### How to do it\...


RabbitMQ sends trace messages using the same mechanism used for log
messages; so, the example code is very similar to the one of the
previous recipe.

To consume  trace messages, you can execute the Java
main function  in `Consumer.java` that you
can find in the book source archive in the directory
`Lab12/Recipe02/Java/src/rmqexample`. Here, we highlight
the main steps:


1.  Create a temporary queue and bind it to the AMQP log exchange:



    ```
    String tmpQueue = channel.queueDeclare().getQueue();
    channel.queueBind(tmpQueue,"amq.rabbitmq.trace","#");
    ```
    

2.  In the consumer callback (`ActualConsumer.java`), retrieve
    the messages with some more information for each one and print them
    using the following code:



    ```
    String routingKey = envelope.getRoutingKey();
    String message = new String(body);
    Map<String,Object> headers = properties.getHeaders();
    LongStringexchange_name = (LongString) 
    headers.get("exchange_name");
    LongString node = (LongString) headers.get("node");
    ...
    ```
    

3.  Activate  firehose by invoking it from the root
    user (Linux) or on the RabbitMQ command console (Windows). Use the
    following command to activate firehose:



    ```
    rabbimqctl trace_on
    ```
    

4.  At this point, you can start producing and sending messages to the
    broker, and you will see them traced to the standard output.

5.  Deactivate firehose by invoking the command:



    ```
    rabbimqctl trace_off
    ```
    







### How it works\...


The `amq.rabbit.trace` topic exchange, by default, does not
receive any messages, but once firehose is activated (step 3 of the
previous steps), all the messages traveling through the broker will be
copied to it by following specific rules:


-   Messages entering the broker are published with the routing key
    `publish.exchange-name`, where `exchange-name`
    is the name of the exchange where the message was originally
    published to.

-   Messages leaving the broker are published with the routing key
    `deliver.queue-name`, where `queue-name` is the
    name of the queue where the message has been originally consumed
    from.

-   The body of the message is copied from the original one.

-   The metadata of the original messages are inserted in the header
    properties of the copied message. In step 2 of the previous numbered
    bullets, we have seen how to retrieve the 
    exchange name to which the message was originally delivered, but
    it\'s possible to get all the original information, that is, find
    all the available  fields inserted into the
    message properties at the firehose official 
    documentation link at
    <http://www.rabbitmq.com/firehose.html>.



Debugging RabbitMQ\'s messages
------------------------------------------------



Sometimes, it is useful  to just have an idea of the
messages that are really  traveling through a broker
by just logging all of them to the standard output.

It is possible to trace those messages using a simple application
provided with the RabbitMQ Java client.





### Getting ready


To run this recipe, you need to have RabbitMQ up and running on the
standard port `5672` and the RabbitMQ Java client library.






### How to do it\...


RabbitMQ includes a tracing utility in the Java client library that you
can put in action by following these steps.


1.  Download the latest version of the RabbitMQ Java client library from
    <http://www.rabbitmq.com/java-client.html>.

2.  Unpack it and enter its directory.

3.  Run the Java tracer by running:



    ```
    ./runjava.sh com.rabbitmq.tools.Tracer
    ```
    

4.  Run the Java client that is to be debugged and that connects to port
    `5673`. For this recipe, we will use another Java tool
    included in the Java client library, by invoking:



    ```
    ./runjava.sh com.rabbitmq.examples.PerfTest -h amqp://localhost:5673 -C 1 -D 1
    ```
    







### How it works\...


The Java tracing tool is a simple AMQP proxy; by default, it listens on
port `5673` and forwards all the requests to the RabbitMQ
broker listening by default on `localhost` at port
`5672`.

All the messages produced or consumed, as well as the AMQP operations,
are all logged to a standard output.

To run the recipe at step 4 of the previous steps we have used another
tool that is included in the RabbitMQ Java  client
library that can be used to perform a stress test with RabbitMQ.

In our case, we have  just limited it to produce one
message (`-C 1`) and consume it (`-D 1`).


###Note

The tracing tool is available in the Java client API only.







### There\'s more\...


It\'s possible to pass some more parameters to the Java tracing program
using the following code:





```
./runjava.sh com.rabbitmq.tools.Tracer listenPort connectHost connectPort 
```


In the preceding code `listenPort` refers to the port where
the tracer is listening (default: `5673`),
`connectHost`/`connectPort` (default:
`localhost/5672`) are the host and the port where it connects
to and forwards the requests it receives.

You can find all `PerfTest` available options using:





```
./runjava.sh com.rabbitmq.examples.PerfTest --help
```







### See also


You can find the documentation of the Java tracing tool and PerfTest at
<http://www.rabbitmq.com/java-tools.html>.



What to do when RabbitMQ fails to restart
-----------------------------------------------------------



Occasionally, RabbitMQ fails to restart. This can be an important issue
in case the broker contains persistent data; otherwise, it\'s enough to
reset the broker persistent state.





### Getting ready


To run this  recipe, you just need a test RabbitMQ
broker.


###Note

We are going to destroy all the previously defined data---avoid using a
production instance.







### How to do it\...


To clean-up  RabbitMQ, it\'s enough to follow these
simple steps:


1.  Stop RabbitMQ if it is running.

2.  Locate the   [**Mnesia**] database
    directory. By default, it\'s `/var/lib/rabbitmq/mnesia`
    (Linux) or `%APPDATA%\RabbitMQ\db` (Windows).

3.  Delete it recursively.

4.  Restart RabbitMQ.







### How it works\...


The Mnesia database contains all the runtime definitions of RabbitMQ:
queues, exchanges, users, and so on.

By deleting it, (or renaming it in case we want to try to recover some
data, or to eventually fall back in case it\'s possible) RabbitMQ is
reset to the factory defaults; once started, it will create a new Mnesia
database and initialize it with default values.






### There\'s more\...


In case the broker fails to start the first time, it is probable that
there is a file permission problem in one of the system directories:
either the Mnesia database directory or the log directory or some
temporary, or custom directories that are specified in the configuration
file.

You can find quite an exhaustive list of cases in the RabbitMQ
troubleshooting page (<http://www.rabbitmq.com/troubleshooting.html>).






### See also


You can find more information on how to hack Mnesia databases at the
Mnesia API documentation pages
(<http://www.erlang.org/doc/man/mnesia.html>).



Debugging using Wireshark
-------------------------------------------



In the [*Debugging RabbitMQ\'s messages*] recipe, we have
seen how to trace messages going to/from RabbitMQ.

However, it is not always possible, or desirable, to stop a running
client (or a RabbitMQ server), modify its connection port, and point it
to a different one; we just want to monitor the messages that are
passing in real-time, impacting the system activity as little as
possible.


### Tip

However, it\'s possible to activate the firehose tracer as seen in the
recipe, [*Tracing RabbitMQ\'s ongoing activity*].


Wireshark is a free network analysis tool that has the capability to
decode AMQP messages. This tool can be used either on the client side or
on the server side to monitor the AMQP traffic flow seamlessly.





### Getting ready


To exercise this recipe, you need RabbitMQ up and running and the
RabbitMQ Java client library.






### How to do it\...


In the following steps, we are  going to see how to
use Wireshark to  trace the AMQP messages:


1.  If not already available on your system, download and install
    Wireshark from <http://www.wireshark.org/>. You can also install it
    for your distribution if it is available, for example, with:



    ```
    yum install wireshark-gnome
    ```
    

2.  Start Wireshark on Linux from the `root` user.

3.  Start to capture from the loopback interface.

4.  From a terminal from the Java client library path, run the command:



    ```
    ./runjava.sh com.rabbitmq.examples.PerfTest -C 1 -D 1
    ```
    

5.  Stop the acquisition from the Wireshark GUI and analyze the captured
    AMQP traffic.







### How it works\...


Using Wireshark, it  is possible to inspect the AMQP
 traffic exiting or entering a server that is
hosting a RabbitMQ server or client.

In our example, we have captured the network traffic running both the
client and the server on the same machine, thus connecting in
`localhost`. That\'s why we were capturing the traffic from
the loopback interface (step 3 of the previous steps).

Otherwise, we should capture the traffic from the network interface,
usually eth0 or something similar.


### Tip

While on Linux, it\'s possible to capture traffic directed to
`localhost`; the same does not apply to Windows. In this case,
the client and the server must be on two different machines, and the
capture must be activated on the network interface (either physical or
virtual), thus connecting them.


So, in order to run the Wireshark graphical user interface, in case the
RabbitMQ client and the server run on the same node, you need to select
the loopback interface, as shown in the following screenshot:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_12_02.jpg)



### Tip

On Linux, when you install the Wireshark package, you usually will have
the command line  interface only,
[**tshark**]. To have Wireshark with the GUI installed, you
have to install the appropriate package. For example, on Fedora, you
have to install the `wireshark-gnome` package.


Once the AMQP traffic  has travelled through the
loopback interface, it has been captured by Wireshark.

The experiment run in  step 4 of the previous steps
actually starts both a producer and a consumer with two separated
connections.

In order to highlight it, find a packet described as
`Basic.PublishContent-Header`, right-click on it, and select
`Follow TCP stream`. You can then close the window showing the
payload dialogue between the client and the server. In the main window,
you can now see the network packets that are exchanged between the
client and the server, as shown in the following screenshot:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_12_03.jpg)


In the same way,  you can select the traffic exiting
the RabbitMQ  server, as shown in the following
screenshot:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_cookbook/6501OS_12_04.jpg)


In the previous two screenshots, we have highlighted the AMQP payload of
both messages, but you will find plenty of details in the AMQP traffic,
thanks to the fact that Wireshark includes a very complete AMQP
dissector.


### There\'s more...

In case RabbitMQ is configured to use SSL and you want to analyze the
encrypted traffic, this is possible under some given conditions by
properly configuring the SSL public/private keys in the Wireshark
configuration.

Find more information at <http://wiki.wireshark.org/SSL>.


### See also

You can find some references to the Wireshark AMQP dissector at
<http://wiki.wireshark.org/AMQP>.
