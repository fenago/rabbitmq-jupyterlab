<img align="right" src="./logo-small.png">


Publish/Subscribe
-----------------

### (using the Pika Python client)

### Prerequisites

This tutorial assumes RabbitMQ is
[installed](https://www.rabbitmq.com/download.html) and running on
localhost on standard port (5672). In case you use a different host,
port or credentials, connections settings would require adjusting.

#### Lab Environment
All packages have been installed. There is no requirement for any setup.

All Notebooks are present in `work/rabbitmq-jupyterlab` folder.

You can access jupyter lab at `<host-ip>:<port>/lab/workspaces/lab3_python`

**Note:** 
- Terminal is already running. You can also open new terminal by clicking:
`File` > `New` > `Terminal`.
- To copy and paste: use **Control-C** and to paste inside of a terminal, use **Control-V**

Run following command in the terminal and move into python files directory:

`cd /home/jovyan/work/rabbitmq-jupyterlab/python`

### What This Tutorial Focuses On

In the previous tutorial we created a work queue. The assumption behind a work queue is that each
task is delivered to exactly one worker. In this part we'll do something
completely different -- we'll deliver a message to multiple consumers.
This pattern is known as "publish/subscribe".

To illustrate the pattern, we're going to build a simple logging system.
It will consist of two programs -- the first will emit log messages and
the second will receive and print them.

In our logging system every running copy of the receiver program will
get the messages. That way we'll be able to run one receiver and direct
the logs to disk; and at the same time we'll be able to run another
receiver and see the logs on the screen.

Essentially, published log messages are going to be broadcast to all the
receivers.

Exchanges
---------

In previous parts of the tutorial we sent and received messages to and
from a queue. Now it's time to introduce the full messaging model in
Rabbit.

Let's quickly go over what we covered in the previous tutorials:

-   A *producer* is a user application that sends messages.
-   A *queue* is a buffer that stores messages.
-   A *consumer* is a user application that receives messages.

The core idea in the messaging model in RabbitMQ is that the producer
never sends any messages directly to a queue. Actually, quite often the
producer doesn't even know if a message will be delivered to any queue
at all.

Instead, the producer can only send messages to an *exchange*. An
exchange is a very simple thing. On one side it receives messages from
producers and the other side it pushes them to queues. The exchange must
know exactly what to do with a message it receives. Should it be
appended to a particular queue? Should it be appended to many queues? Or
should it get discarded. The rules for that are defined by the *exchange
type*.

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images//exchanges.webp)

There are a few exchange types available: direct, topic, headers and
fanout. We'll focus on the last one -- the fanout. Let's create an
exchange of that type, and call it logs:

``` {.lang-python .hljs}
channel.exchange_declare(exchange='logs',
                         exchange_type='fanout')
```

The fanout exchange is very simple. As you can probably guess from the
name, it just broadcasts all the messages it receives to all the queues
it knows. And that's exactly what we need for our logger.

> #### Listing exchanges
>
> To list the exchanges on the server you can run the ever useful
> rabbitmqctl:
>
> ``` {.lang-bash .hljs}
> sudo rabbitmqctl list_exchanges
> ```
>
> In this list there will be some amq.\* exchanges and the default
> (unnamed) exchange. These are created by default, but it is unlikely
> you'll need to use them at the moment.
>
> #### The default exchange
>
> In previous parts of the tutorial we knew nothing about exchanges, but
> still were able to send messages to queues. That was possible because
> we were using a default exchange, which we identify by the empty
> string ("").
>
> Recall how we published a message before:
>
> ``` {.lang-python .hljs}
> channel.basic_publish(exchange='',
>                       routing_key='hello',
>                       body=message)
> ```
>
> The exchange parameter is the name of the exchange. The empty string
> denotes the default or *nameless* exchange: messages are routed to the
> queue with the name specified by routing\_key, if it exists.

Now, we can publish to our named exchange instead:

``` {.lang-python .hljs}
channel.basic_publish(exchange='logs',
                      routing_key='',
                      body=message)
```

Temporary queues
----------------

As you may remember previously we were using queues that had specific
names (remember hello and task\_queue?). Being able to name a queue was
crucial for us -- we needed to point the workers to the same queue.
Giving a queue a name is important when you want to share the queue
between producers and consumers.

But that's not the case for our logger. We want to hear about all log
messages, not just a subset of them. We're also interested only in
currently flowing messages not in the old ones. To solve that we need
two things.

Firstly, whenever we connect to Rabbit we need a fresh, empty queue. To
do it we could create a queue with a random name, or, even better - let
the server choose a random queue name for us. We can do this by
supplying empty queue parameter to queue\_declare:

``` {.lang-python .hljs}
result = channel.queue_declare(queue='')
```

At this point result.method.queue contains a random queue name. For
example it may look like amq.gen-JzTY20BRgKO-HjmUJj0wLg.

Secondly, once the consumer connection is closed, the queue should be
deleted. There's an exclusive flag for that:

``` {.lang-python .hljs}
result = channel.queue_declare(queue='', exclusive=True)
```

You can learn more about the exclusive flag and other queue properties
in the [guide on queues](https://www.rabbitmq.com/queues.html).

Bindings
--------

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images//bindings.webp)


We've already created a fanout exchange and a queue. Now we need to tell
the exchange to send messages to our queue. That relationship between
exchange and a queue is called a *binding*.

``` {.lang-python .hljs}
channel.queue_bind(exchange='logs',
                   queue=result.method.queue)
```

From now on the logs exchange will append messages to our queue.

> #### Listing bindings
>
> You can list existing bindings using, you guessed it,
>
> ``` {.lang-bash .hljs}
> rabbitmqctl list_bindings
> ```
>
Putting it all together
-----------------------

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images//python-three-overall.webp)

The producer program, which emits log messages, doesn't look much
different from the previous tutorial. The most important change is that
we now want to publish messages to our logs exchange instead of the
nameless one. We need to supply a routing\_key when sending, but its
value is ignored for fanout exchanges.

emit\_log.py
([source](https://github.com/fenago/rabbitmq-jupyterlab/blob/master/python/emit_log.py))

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='logs', exchange_type='fanout')

message = ' '.join(sys.argv[1:]) or "info: Hello World!"
channel.basic_publish(exchange='logs', routing_key='', body=message)
print(" [x] Sent %r" % message)
connection.close()
```

As you see, after establishing the connection we declared the exchange.
This step is necessary as publishing to a non-existing exchange is
forbidden.

The messages will be lost if no queue is bound to the exchange yet, but
that's okay for us; if no consumer is listening yet we can safely
discard the message.

receive\_logs.py
([source](https://github.com/fenago/rabbitmq-jupyterlab/blob/master/python/receive_logs.py))

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='logs', exchange_type='fanout')

result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

channel.queue_bind(exchange='logs', queue=queue_name)

print(' [*] Waiting for logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(" [x] %r" % body)

channel.basic_consume(
    queue=queue_name, on_message_callback=callback, auto_ack=True)

channel.start_consuming()
```

We're done. If you want to save logs to a file, just open a console and
type:

``` {.lang-bash .hljs}
python receive_logs.py > logs_from_rabbit.log
```

If you wish to see the logs on your screen, spawn a new terminal and
run:

``` {.lang-bash .hljs}
python receive_logs.py
```

And of course, to emit logs type:

``` {.lang-bash .hljs}
python emit_log.py
```

Using rabbitmqctl list\_bindings you can verify that the code actually
creates bindings and queues as we want. With two receive\_logs.py
programs running you should see something like:

``` {.lang-bash .hljs}
sudo rabbitmqctl list_bindings
# => Listing bindings ...
# => logs    exchange        amq.gen-JzTY20BRgKO-HjmUJj0wLg  queue           []
# => logs    exchange        amq.gen-vso0PVvyiRIL2WoV3i48Yg  queue           []
# => ...done.
```

The interpretation of the result is straightforward: data from exchange
logs goes to two queues with server-assigned names. And that's exactly
what we intended.

To find out how to listen for a subset of messages, let's move on to tutorial 4.