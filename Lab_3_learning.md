

Lab 3. Administration, Configuration, and Management
-----------------------------------------------------------------


In order to get the most out of a system, you need to know how to
configure and control it. Depending on the type of system, these tasks
could turn out to be quite daunting and onerous (consider a relational
database, for example). However, the RabbitMQ team has provided very
convenient facilities for administering and managing the message broker.

Topics covered in the lab:


-   Administering RabbitMQ instances

-   Administering the RabbitMQ database

-   Installing RabbitMQ plugins

-   Configuring RabbitMQ instances

-   Managing RabbitMQ instances

-   Upgrading RabbitMQ



Administering RabbitMQ instances
--------------------------------------------------



Administration  of RabbitMQ server instances can be
considered in several directions:


-   Starting/stopping/restarting instances

-   Adding/removing/modifying/inspecting users, virtual hosts,
    exchanges, queues, and bindings

-   Backup and recovery of the RabbitMQ database

-   Setting up a different database for message persistence

-   Taking care of broker security

-   Inspecting RabbitMQ logs for errors

-   Optimizing resource utilization, tuning performance and monitoring
    the broker

-   Configuring the broker using environment variables, configuration
    parameters, and policies

-   Managing the broker by writing custom applications that make use of
    the REST API exposed by the RabbitMQ management plugin


Some of  the preceding concepts are covered in
subsequent labs. We already saw how easy it is to start/stop/restart
instances using the `rabbitmqctl` and
`rabbitmq-server` utilities that are part of the standard
RabbitMQ installation. Before diving into the nuts and bolts of RabbitMQ
administration, let\'s review the standard directory structure of a
typical RabbitMQ server installation. In Windows, run the following
command from the installation folder of RabbitMQ:





``` {.programlisting .language-markup}
    tree /A
```


The following screenshot displays the output from the preceding command:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_01.jpg)


Mnesia is a distributed NoSQL database used by RabbitMQ to store
information about users, vhosts, exchanges, queues, bindings, index
files (the position of messages in queues), and cluster information. It
can store data either on RAM or on disk. Although persistent messages
are stored along with the Mnesia files (in the Mnesia folder), they are
not managed by Mnesia. RabbitMQ provides its own persistent storage for
messages. On the one hand, persistent messages are stored in the
`msg_store_persistent` directory (both when they are persisted
when received on a queue or when memory consumption grows beyond a
specific threshold); on the other hand, non-persistent (transient)
message are persisted in the `msg_store_transient` directory
(when memory consumption on a queue grows beyond a specific threshold).

The `ebin` directory contains the Erlang compiled sources.
They are cross-platform and are interpreted by the Erlang virtual
machine installed on the machine on which the RabbitMQ server is
installed.

The `include` directory includes the Erlang header files
(similar in notion to C++ header files but for Erlang).

The `log` directory  contains the RabbitMQ
log files and the Erlang [**SASL**] ([**System Application
Support Libraries**]) log files, not to be confused with
[**SASL**] ([**Simple Authentication and Security
Layer**]), for which RabbitMQ also provides support, covered in
[Lab
9](https://subscription.packtpub.com/book/application_development/9781783984565/9){.link},
[*Security*]{.emphasis}. Erlang SASL provides support for topics such as
error logging, alarm handling, and overload regulation.

The `plugins` directory provides packages for the RabbitMQ
binaries.

The `sbin` directory contains the RabbitMQ scripts used for
server administration, such as rabbitmq-server.bat and rabbitmqctl.bat
under Windows.

The following screenshot illustrates the RabbitMQ folder structure for
Ubuntu/Debian:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_02.jpg)


And the following is for a generic Unix installation:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_03.jpg)


Note that  database and log files are not created
until the RabbitMQ broker is started for the first time. If you delete
the RabbitMQ database and/or log files, they will be recreated when the
broker is started again.

The locations of some parts of the RabbitMQ installation files can be
configured using environment variables, such as:


-   `RABBITMQ_BASE` sets  the location of
    the RabbitMQ database and log files. Note that, if it is not set
    under Windows, then the default location for the variable will be
    `%APPDATA%\RabbitMQ` (meaning that your database, log, and
    configuration files will be stored under that directory unless other
    configuration parameters are used to change their location). You can
    set this directory to be the installation folder of your RabbitMQ
    server if you want to store the database, log, and configuration in
    the same location as the other RabbitMQ server components.

-   `RABBITMQ_CONFIG_FILE` sets  the
    location of the RabbitMQ configuration file (without the
    `.config` extension of the file).

-   `RABBITMQ_LOG_BASE` specifies  the base
    directory for storing RabbitMQ log files.


For more information on the various environment variables related to the
directory structure of the RabbitMQ server, you can refer to the
RabbitMQ server documentation.





### Administering RabbitMQ components


The  various RabbitMQ components can be modified in
any of the following ways:


-   From the web interface of the RabbitMQ management plugin

-   From the `rabbitmqctl` script (in the `sbin`
    directory)

-   From the REST API of the RabbitMQ management plugin


So far, we  have seen how to programmatically create
queues, exchanges, and bindings. However, they can be pre-created in the
broker so that the overhead of managing them from source code on the
producer/consumer side is minimized. Moreover, we can also create users,
vhosts, and policies using the management plugin or the rabbitmqctl
utility. For some administrative tasks, you can use a command line
utility (rabbitmqadmin) that comes with the RabbitMQ management plugin.
In order to download it, navigate to `http://localhost:1`
`5672/cli/` and save it to a proper location (for example, the
`sbin` directory of the RabbitMQ installation; make sure you
save it with a `.py` extension since it is a Python script,
and ensure you have Python 3 installed before using the script). To view
all available commands for the rabbitmqadmin.py script, you can issue
the following from the command line:





``` {.programlisting .language-markup}
rabbitmqadmin.py help
rabbitmqadmin.py help subcommands
```







### Administering users


You  can easily create new users from the command
line. For example, if you want to create a user with the name
`sam` and the password `d1v`, and a user
`jim` with the password `tester`, you can issue the
following commands:





``` {.programlisting .language-markup}
rabbitmqctl add_user sam d1v
rabbitmqctl add_user jim tester
```


The preceding users are regular (non-administrative) users and not
assigned to any vhost. At that point, if you try to access the web
management console you will receive a login failure. In order to make
`sam` an admin user you can issue the following command:





``` {.programlisting .language-markup}
rabbitmqctl.bat set_user_tags jim administrator
```


Now `jim` is able to administer the broker and login to the
management console. The users still don\'t have access to any vhost
(even the default one). If you navigate to the `Admin` tab in
the management console, you will see something like this:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_04.jpg)


The following  command can be used to list all users
in the broker instance:





``` {.programlisting .language-markup}
rabbitmqctl.bat list_users
```


If you want to change the password for `sam` to
`t1ster`, you can issue the following command:





``` {.programlisting .language-markup}
rabbitmqctl.bat change_password jim t1ster
```


If you want to delete the user `sam`, you can issue the
following command:





``` {.programlisting .language-markup}
rabbitmqctl.bat delete_user jim
```


You can also manage users from the RabbitMQ web management interface or
the `rabbitmqadmin.py` script. Let\'s make `sam` an
administrator:





``` {.programlisting .language-markup}
rabbitmqctl.bat set_user_tags sam administrator
```







### Administering vhosts


We have  already mentioned that vhosts are used to
logically separate a broker instance into multiple domains, each one
with its own set of exchanges, queues, and bindings. The following
example creates the `chat` and `events` vhost:





``` {.programlisting .language-markup}
rabbitmqctl.bat add_vhost chat
rabbitmqctl.bat add_vhost events
```


Note that it might be a better idea to name your vhosts hierarchically
(meaning that chat becomes /chat and vhost becomes /vhost; any child
vhosts can be added following the same pattern---for example,
`/chat/administrators` and `/events/follow`).

If you  navigate to the `Admin` tab in the
management console and click on Virtual Hosts, you will see something
like this:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_05.jpg)


The following command can be used to list all virtual hosts in the
broker instance:





``` {.programlisting .language-markup}
rabbitmqctl.bat list_vhosts
```


You can use the following command to the delete the `events`
vhost:





``` {.programlisting .language-markup}
rabbitmqctl.bat delete_vhost events
```


You can also manage vhosts from the RabbitMQ web management interface.






### Administering permissions


Now that we  have seen how to create users and
vhosts, we can assign permissions to particular users so that they are
able to access particular vhosts (and all of the RabbitMQ components
associated with that vhost). The following example grants configure,
write, and read permissions to all resources in the `chat`
vhost to the user `jim`:





``` {.programlisting .language-markup}
rabbitmqctl.bat set_permissions –p chat jim ".*" ".*" ".*"
```


Note that in some cases under Windows, any of the rabbitmqctl commands
may not be properly executed due to Erlang issues with encoding under
Windows. In that case, you can also use the `rabbitmqadmin.py`
script as follows:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare permission vhost=chat user=sam configure=.* write=.* read=.*
```


As you can  see, the configure, write, and read
permissions can be regular expressions that match the names of the
vhosts components that the user has access to. You can list all
permissions in the broker with the following command:





``` {.programlisting .language-markup}
rabbitmqctl.bat list_permissions
```


Alternatively, you can use the `rabbitmqadmin.py` script for
this purpose:





``` {.programlisting .language-markup}
rabbitmqadmin.py list permissions
```


You can delete the permission given to the user `sam` for the
`chat` vhost using the following command:





``` {.programlisting .language-markup}
rabbitmqctl.bat clear_permissions -p chat sam
```


Alternatively you can use the `rabbitmqadmin.py` script for
this purpose:





``` {.programlisting .language-markup}
rabbitmqadmin.py delete permission vhost=chat user=sam
```


If you omit the vhost from the preceding commands, you will clear all
permissions assigned to the user `sam`. You can also list all
vhosts to which the user `sam` is assigned with the following
command:





``` {.programlisting .language-markup}
rabbitmqadmin -u sam -p d1v list vhosts
```







### Administering exchanges


You can  create exchanges from the RabbitMQ
management web interface or the rabbitmqadmin script. The following
example creates the `logs` fanout exchange in the default
vhost:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare exchange name=logs type=fanout
```


The following example creates another fanout exchange with the name
`logs` in the `chat` vhost (first we set permissions
for the guest user to the vhost; otherwise, we have to specify a user
that has administrator permissions for the vhost):





``` {.programlisting .language-markup}
rabbitmqadmin.py declare permission vhost=chat user=guest
configure=.* write=.* read=.*rabbitmqadmin.py declare -V chat exchange name=logs type=fanout
```


When declaring an exchange, you can specify additional properties such
as exchange durability. To delete the `logs` exchange from the
`chat` domain, you can issue:





``` {.programlisting .language-markup}
rabbitmqadmin.py -V chat delete exchange name=logs
```


To list all exchanges in the `chat` vhost, you can issue:





``` {.programlisting .language-markup}
rabbitmqadmin.py -V chat list exchanges
```


To list all  exchanges in the default vhost, you can
issue:





``` {.programlisting .language-markup}
rabbitmqadmin.py list exchanges
```







### Administering queues


You can  create queues from the RabbitMQ management
web interface or the `rabbitmqadmin` script. The following
example creates the non-durable `error_logs` queue in the
default vhost:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare queue name=error_logs durable=false
```


The following example creates a queue with the same name in the
`chat` vhost:





``` {.programlisting .language-markup}
rabbitmqadmin.py -V chat declare queue name=error_logs
```


To delete the `error_logs` queue from the `chat`
vhost, you can issue the following:





``` {.programlisting .language-markup}
rabbitmqadmin.py -V chat delete queue name=error_logs
```


To list all queues in the default domain, you can issue:





``` {.programlisting .language-markup}
rabbitmqadmin.py list queues
```







### Administering bindings


Now  that we have seen how straightforward it is to
create exchanges and queues, let\'s see how to create bindings. The
following creates a binding between the `logs` fanout exchange
we already created and the `error_logs` queue in the default
vhost:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare binding source=logs destination=error_logs
```


In order to test that the binding works, we can use the
`rabbitmqadmin` script to publish to the `logs`
exchange, then read from the `error_logs` queue (here you can
check if the message is successfully retrieved from the queue), and
finally clear the `error_logs` queue from any messages:





``` {.programlisting .language-markup}
rabbitmqadmin.py publish exchange=logs routing_key= payload="new log"
rabbitmqadmin.py get queue=error_logs
rabbitmqadmin.py purge queue name=error_logs
```







### Administering policies


Policies  allow you to define (and change) certain
properties of exchanges and queues at runtime. Since no more than one
policy can be defined per exchange/queue, a policy can incorporate
multiple settings at once. Let\'s consider the following scenarios:


-   We decide to set a limit on the capacity of a queue; if it is
    exceeded then the messages are either dropped or dead-lettered
    (meaning they are redirected to an alternative exchange)

-   We decide to set a limit on the time that a message is allowed to
    stay in a queue; if that time is exceeded for a message then it is
    either dropped or dead-lettered

-   We want to define a dead-letter exchange that receives dead-letter
    messages from one or more queues


In order to set the capacity of the `error_logs` queue in the
default (\'/\') vhost to 200,000 bytes, you can apply the following
policy:





``` {.programlisting .language-markup}
rabbitmqctl  set_policy max-queue-len "error_logs" "{""max-length-bytes"" : 200000}" apply-to queue 
```


You can also use the rabbitmqadmin.py script for this purpose:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare policy name=max-queue-len pattern=error_logs definition="{""max-length-bytes"":200000}" apply-to=queues
```


The following policy sets the maximum queue length in terms of messages
(if you want to apply it you must first drop the previously created
policy):





``` {.programlisting .language-markup}
rabbitmqadmin.py declare policy name=max-queue-len pattern=error_logs definition="{""max-length"":200000}" apply-to=queues
```


Notice that instead of the name of the queue (`error_logs` in
that case), you can specify a pattern for the names of the queues to
which the policy applies. This means that policies apply to queues that
match the pattern and they are added after the policy is created. To
delete the policy you can issue:





``` {.programlisting .language-markup}
rabbitmqctl.bat clear_policy max-queue-len
```


Alternatively you can issue:





``` {.programlisting .language-markup}
rabbitmqadmin.py delete policy name=max-queue-len
```


Note that the queue length might also be set from the client using the
`x-max-length` arguments passed to the arguments map in the
declaration of a queue from the client.

In order to set the   [**TTL**]
([**time-to-live**]) of the messages to all queues in the
default vhost to three seconds, you can apply the following policy:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare policy name=ttl pattern=.* definition="{""message-ttl"":3000}" apply-to=queues
```


Note that the  message TTL for the queue might also
be set from the client using the `x-message-ttl` arguments
passed to the arguments map in the declaration of a queue from the
client or on a per-message basis using the `expiration` field
set properly on the `AMQP.BasicProperties` instance passed
when publishing a message. You can also set expiration for the entire
queue, which means that the queue will be automatically deleted after a
certain period of idle time; this is particularly useful when a large
number of queues is created and they need to be purged over time. The
following example sets the queue TTL for all queues starting with the
`response` prefix to 10 minutes:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare policy name=queue-ttl          pattern=response.* definition="{""expires"":600000}" apply-to=queues
```


Note that the queue TTL might also be set from the client using the
`x-queue` arguments passed to the arguments map in the
declaration of a queue from the client.

If a message TTL expires, the queue capacity is exhausted, or a message
received from a queue is explicitly rejected from a consumer, it can be
routed to an alternative dead-letter exchange. The following diagram
provides an overview of the scenario:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_06.jpg)


The following example creates the `logs_dlx` exchange and sets
it as a dead-letter exchange to the `error_logs` queue:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare exchange name=logs_dlx type=fanoutrabbitmqadmin.py declare policy name=ttl                pattern="^error_logs$" definition="{""dead-letter-exchange"": ""logs_dlx"", ""message-ttl"":3000}" apply-to=queues
```


Note that if we use only `"error_logs"` instead of
`"^error_logs$"` then `error_logs_dlx` will also be
matched and we don\'t want this to happen. Notice that in the preceding
example we combined the dead-letter-exchange policy with the message-ttl
policy. You can list all policies with the following command:





``` {.programlisting .language-markup}
rabbitmqadmin.py list policies
```


Note that you  have to make sure that only one
policy applies at a time on a queue; if two or more patterns match a
queue name then it becomes unclear which policy will be applied. If that
happens, remove policies that apply to a queue and combine them in a
single composite policy. To delete the `max-queue-len` policy
we created earlier, issue the following command:





``` {.programlisting .language-markup}
rabbitmqadmin.py delete policy name=max-queue-len
```


In order to test that the dead-letter exchange is properly configured we
can use the following scenario:


-   Create a queue named `error_logs_dlx` that binds to the
    `logs_dlx` exchange

-   Send a message to the `logs` exchange

-   Wait for more than three seconds

-   Check that the message can be consumed from
    `error_logs_dlx`

-   Clear the `error_logs_dlx` queue


The following example can be used to test the preceding scenario:





``` {.programlisting .language-markup}
rabbitmqadmin.py declare queue name=error_logs_dlx
rabbitmqadmin.py declare binding source=logs_dlx        destination=error_logs_dlx
rabbitmqadmin.py publish exchange=logs routing_key=     payload="dlx message"
```


Wait at least three seconds and execute the following in order to verify
that the message is sent to the dead-letter queue (clearing the queue at
the end):





``` {.programlisting .language-markup}
rabbitmqadmin.py get queue=error_logs_dlx
rabbitmqadmin.py purge queue name=error_logs   
```







### Administering the RabbitMQ database


The  RabbitMQ database stores both message server
metadata and messages from queues. In the next sections we will see how
can we manage this database for the purpose of disaster recovery.






### Full backup and restore


As we have  already seen, RabbitMQ uses Mnesia to
store information about the various components of the broker as well as
cluster configuration and a custom database for storing persistent
messages. In that regard it is straightforward to back up the contents
of the RabbitMQ database:


-   Stop the broker

-   Copy the Mnesia folder and archive it

-   Restart the broker


The restore  procedure, as you might have guessed,
is pretty similar. You should also consider the fact that if a message
is not persistent it may not be backed up using the preceding procedure
since it is not written to the persistent store of RabbitMQ (in the
event of a crash). In order for a message to be persistent, the exchange
and queue through which it passes must be durable (marked as such during
creation) and the message must be marked as persistent (with a delivery
mode set to 2 from the sender). A response for a successfully received
persistent message is not sent until a message is written to the
persistent log file on an exchange. You may be wondering about the case
when a live backup must be made on the RabbitMQ database with
preservation of messages at a particular point in time. In this case you
have a number of options to consider, such as:


-   Using the exchange-to-exchange bindings extension that allows you to
    pass a message through multiple exchanges. In this regard you can
    create a separate exchange for backup purposes and bind all other
    exchanges to that one; a dedicated queue bound to that exchange can
    be used to save messages to a persistent store along with a
    timestamp for a custom point-in time recovery implementation.

-   Creating a federated exchange (in the same or another broker),
    linked to all exchanges in the broker, that receives all of the
    messages published to exchanges from the broker. The federated
    exchange can then be bound to a dedicated queue that can be used to
    save messages to a persistent store along with a timestamp for a
    custom point-in time recovery implementation; the Federation plugin
    is required for that purpose.

-   Replicating messages from all queues to a destination exchange using
    shovels; the Shovel plugin is required for that purpose.


In many cases however you may need to backup/restore only the
configuration of RabbitMQ components at a particular point in time.






### Backing up and restoring the broker metadata


In order  to back up the RabbitMQ broker metadata
(the configuration of broker components) you can use the rabbitmqadmin
management plugin as follows (assuming we want to backup the broker
configuration to a file named `broker.export` in the current
directory):





``` {.programlisting .language-markup}
rabbitmqadmin.py export broker.json
```


If you open the  file you will notice that there is
a section for each type of component, along with the version of the
broker:





``` {.programlisting .language-markup}
{  
   "rabbit_version":"3.4.4",
   "users":[  
      {  
         "name":"sam",
         "password_hash":"y7CFOccmv5tReRwEskXapNOSsmM=",
         "tags":"administrator"
      },
      ….
   ],
   "vhosts":[  
      {  
         "name":"chat"
      },
      {  
         "name":"/"
      }
   ],
…
}
```


To import back the configuration, you can use the following command:





``` {.programlisting .language-markup}
rabbitmqadmin.py import broker.json
```


Note that it is a good idea to add a user-readable timestamp to the name
of the export file, based on the utilities provided by your OS for that
purpose. You can also perform the export/import of the current RabbitMQ
configuration for the management web interface from the
[**Overview**] tab.



Installing RabbitMQ plugins
---------------------------------------------



So far, we have used the rabbitmq-plugins utility in order to enable the
management plugin (already part of the RabbitMQ installation). You may
want to install additional (for example, community) plugins that allow
you to extend the features of the broker, thus giving you the
opportunity to implement a wider range of messaging scenarios.
Installing a plugin is a two-step process:


-   Download the ez archive (Erlang ZIP archive) of the plugin and copy
    it to the `plugins` folder from the RabbitMQ installation

-   Enable the plugin with the rabbitmq-plugins utility


Let\'s say we want to be able to send e-mails from our messages directly
from the RabbitMQ instance  that receives the
messages. For that reason, you can install the rabbitmq\_email plugin
that provides the AMQP-SMTP and SMTP-AMQP protocol conversion plugins.
Download the AMQP-SMTP plugin from
<https://www.rabbitmq.com/community-plugins/v3.4.x/gen_smtp-0.9.0-rmq3.4.x-61e19ec5-gita62c02e.ez>
and copy it to the `plugins` folder in the RabbitMQ
installation. You can see that the plugin can now be managed from the
broker by issuing:





``` {.programlisting .language-markup}
rabbitmq-plugins.bat list
```


You should see that the `gen_smtp` plugin is present in the
lists and points to the archive we copied to the `plugins`
folder. In order to enable it, you can issue the following:





``` {.programlisting .language-markup}
rabbitmq-plugins.bat enable gen_smtp
```


To delete a plugin you can disable it and remove it from the
`plugins` directory.



Configuring RabbitMQ instances
------------------------------------------------



RabbitMQ configuration  can be established in
several ways:


-   By setting proper environment variables

-   By modifying the RabbitMQ configuration file

-   By defining runtime parameters and policies that can be modified at
    runtime






### Setting environment variables


Environment variables  can be set using a standard
mechanism provided by your OS (for example, using the Control Panel in
Windows or setting them permanently from the shell in Linux). However
they can also be specified in the scripts used to run the RabbitMQ
broker, such as the `rabbitmq-server` utility, the
`rabbitmq-service` utility (used in Windows to start RabbitMQ
as a Windows service), or `rabbitmq-env.conf` (using in
Unix-like operating systems by RabbitMQ to configure environment
variables). At the beginning of the lab we covered several such
variables related to the location of the RabbitMQ database, logs, and
configuration file. Here are several more you can configure:


-   `RABBITMQ_NODE_IP_ADDRESS`: The  IP
    address of network interface to which you want to bind the RabbitMQ
    broker. This is useful if you have multiple such interfaces on the
    machine where the broker is installed and you want to bind it to
    only one of them (an empty value means that the broker is bound to
    all network interface addresses).

-   `RABBITMQ_NODE_PORT`: The  port on
    which the RabbitMQ broker listens.

-   `RABBITMQ_NODENAME`: The  name of the
    RabbitMQ broker instance (this is required in a clustered
    configuration---more on that in the next lab).

-   `RABBITMQ_SERVICENAME`: The  name of
    the Windows service for the RabbitMQ broker instance.







### Modifying the RabbitMQ configuration file


The  rabbitmq configuration file
(`rabbitqm.config`) can be used to provide additional
configuration of the broker, such as how much RAM the broker is allowed
to consume before messages are flushed to the hard disk
(`vm_memory_high_watermark`); what IP addresses and ports of
the network interfaces the broker listens on
(`tcp_listeners`); or what the maximum file size is of the
RabbitMQ message stores--- both transient and persistent
(`msg_store_file_size_limit`). If that limit is exceeded then
messages are garbage-collected. The default location for
`rabbitmq.config` is under the `%RABBITMQ_BASE%`
directory; if RabbitMQ is not specified under Windows the default
location of the file will be under `%APPDATA%`. There is a
sample configuration file in the etc directory for the installation of
the RabbitMQ server. If you copy it and save it under the root
installation directory of RabbitMQ with the name
`rabbitmq.config`, you can simply uncomment and change the
various configuration parameters based on your preferences. Here is a
sample configuration that sets limits on the used RAM and message store
size:





``` {.programlisting .language-markup}
[
 {rabbit,
  [
   {vm_memory_high_watermark, 0.4},
   {msg_store_file_size_limit, 16777216}
   ]
 }
]
```



Managing RabbitMQ instances
---------------------------------------------



RabbitMQ  provides a number of utilities for
managing RabbitMQ instances since the AMQP protocol provides limited
support for that purpose (and it is not a responsibility of the protocol
in general to do so). So far we have seen how we can administer RabbitMQ
from the command line using the rabbitmqctl or the rabbitmqadmin
utilities. However there are many scenarios where more sophisticated
tools for provisioning and managing the RabbitMQ broker components are
needed (for example, in the form of an alternative web interface).

In that case, the management plugin provides an interface of REST
(Representational State Transfer)-based web services. In order to see
all the available services in your current installation of the
management plugin you can navigate from the browser to
`http://localhost:15672/api/`---there is a short description
with basic examples and a reference guide for the various services. For
testing purposes, you can use any utility (such as cURL) that
 allows you to send HTTP requests to the manage REST API. As
everything in REST is a resource that is managed with `CRUD`
operations provided by the HTTP methods (such as `GET`,
`POST`, `PUT`, `DELETE`), so are RabbitMQ
resources. If you take a closer look you will notice that all of the
resources are precisely the various types of RabbitMQ components (such
as vhosts, users, permissions, queues, exchanges, and bindings); no
rocket science here. The REST interface respects the current user
permissions (configure, write, read for particular components) when
checking for permissions for performing a certain action.

Let\'s assume that we want to implement a simple utility called
ComponentFinder that allows us to list particular RabbitMQ components in
a given vhost based on a regular expression. For that purpose we will
create a new Maven project that uses the REST client from the Apache
Jersey library provided as a Maven dependency, along with the standard
JSON utility in Java:





``` {.programlisting .language-markup}
<dependency><groupId>com.sun.jersey</groupId><artifactId>jersey-client</artifactId><version>1.19</version></dependency>
<dependency>
   <groupId>org.json</groupId>
   <artifactId>json</artifactId>
   <version>20140107</version>
</dependency>
```


Here is the class for the ComponentFinder utility:





``` {.programlisting .language-markup}
import java.util.Scanner;
import java.util.regex.Pattern;

import org.json.JSONArray;
import org.json.JSONObject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.sun.jersey.api.client.Client;
import com.sun.jersey.api.client.WebResource;
import com.sun.jersey.api.client.filter.HTTPBasicAuthFilter;

public class ComponentFinder {

    private final static Logger LOGGER = LoggerFactory
        .getLogger(ComponentFinder.class);
private static final String API_ROOT =                         "http://localhost:15672/api";
```


The `main()` method  provides the logic
for the tool, reading from the standard input and processing the request
based on the input parameters. A simple HTTP client is used for the
purpose:





``` {.programlisting .language-markup}
    public static void main(String[] args) {

        Scanner scanner = null;
        try {
            scanner = new Scanner(System.in);
            System.out.println("Enter component type in                     plural form (for example, queues, exchanges) ");
            String type = scanner.nextLine();
            System.out.println("Enter vhost (leave empty                     for default vhost) ");
            String vhost = scanner.nextLine();
            System.out.println("Enter name pattern (leave                     empty for match-all pattern)");
            String pattern = scanner.nextLine();

            Client client = Client.create();
            String path;
            if (vhost.trim().isEmpty()) {
                path = API_ROOT + "/" + type +                             "?columns=name";
            } else {
                path = API_ROOT + "/" + type +                             "/" + vhost + "?columns=name";
            }

            WebResource resource = client.resource(path);
            resource.header("Content-Type",                                 "application/json;charset=UTF-8");
            resource.addFilter(new HTTPBasicAuthFilter("guest", "guest".getBytes()));
            String result = resource.get(String.class);
            JSONArray jsonResult = new JSONArray(result);
            LOGGER.debug("Result: \n" + jsonResult.toString(4));
filterResult(jsonResult, pattern);
        } finally {
            if (scanner != null) {
                scanner.close();
            }
        }
    }
```


The `filterResult()` helper  method is
used to filter the response from the management API based on a regular
expression:





``` {.programlisting .language-markup}
private static void filterResult(JSONArray jsonResult,                 String pattern) {
        // filter the result based on the pattern
        for (int index = 0; index < jsonResult.length();                 index++) {
            JSONObject componentInfo =                                 (JSONObject) jsonResult.get(index);
            String componentName =                                 (String) componentInfo.get("name");
            if (Pattern.matches(pattern, componentName)) {
                LOGGER.info("Matched component: " +                         componentName);
                // do something else with component
            }
        }
    }}
```


Upgrading RabbitMQ
------------------------------------



Upgrading  RabbitMQ can be considered in two
directions:


-   Upgrading the Erlang installation

-   Upgrading the broker installation


In both cases, it is good practice to perform a full backup of the
RabbitMQ broker before performing an upgrade. Also you should check out
the release notes for all the versions issued between the old and the
new version to see if there are any specific steps that must be
performed during the update. Typically, installation of a RabbitMQ
broker preserves data and updates only the RabbitMQ installation and the
database structures used for representing the broker metadata and
message stores. It is important to make sure that, if you have to update
nodes in a cluster, you first stop all nodes and use the same version of
RabbitMQ for the update over all nodes in the cluster.





### Case study: Administering CSN


For easier  management, we have decided to
pre-configure our CSN RabbitMQ broker (using a custom script) with two
separate vhosts:


-   `v_chat`: For handling all chat messages in CSN

-   `v_events`: For handling of all events in CSN


Moreover we have decided to separate the users that are allowed to
access each vhost. The users of the `v_events` group are
further divided into the following logical groups:


-   Administrators have the ability to create event queues, and publish
    and consume messages

-   `event_publishers` have the ability to publish messages

-   `event_subscribers` have the ability to consume messages


As you may guess, we can implement the preceding logical separation
easily for the users in the `v_events` host using policies.
The users of the `v_chat` vhosts have full configure, read,
and write access to the components of the vhost.

Another thing we want to provide is the ability to log all messages that
pass through the broker for backup and restore purposes. We also decide
to set limitations on the RAM and disk storage used by the broker using
a custom `rabbitmq.config` file.


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_learning/4565OS_03_07.jpg)


You can provision the additional components as part of the setup process
easily by using custom code and the REST API, which allows to create the
vhosts, users for them (with the appropriate policies to act as access
control based on the logical separation of the users), and a backup
exchange that receives a copy of all messages passed to all other
exchanges in the broker. A custom utility (that could be part of the
backup databases as well) subscribes to that exchange and stores the
messages in the database.



Summary
-------------------------



In this lab,  we saw how to administer a
standalone RabbitMQ broker along with its components, users, vhosts,
permissions, queues, exchanges, bindings, and policies. We discussed the
structure of a typical RabbitMQ installation (along with the parameters
that allow us to configure different locations for various parts of the
broker) and how to provide further configuration in terms of environment
variables and the `rabbitmq.config` file. We discussed
administrative tasks such as backing up and restoring the RabbitMQ
database, updating a RabbitMQ broker, and plugin installation and
management of the broker using the management REST API. In the next
lab we will explore what clustering support the message broker
provides for the purpose of scalability.



Exercises
---------------------------




1.  What utilities can you use to create users, vhosts, and policies in
    a RabbitMQ broker?

2.  What utilities can you use to create exchanges, queues, and
    bindings?

3.  How can you back up and restore RabbitMQ broker metadata?

4.  How can you set limits on the maximum RAM and disk space for storing
    messages in RabbitMQ?

5.  What happens when the various resource limits set on the broker are
    exceeded?

6.  Assuming you need to migrate the RabbitMQ message stores to a larger
    disk mounted on the current machine, how can you do it?

7.  A new version of RabbitMQ comes out that provides a major security
    fix. How can you upgrade your installation of RabbitMQ?
