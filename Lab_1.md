<img align="right" src="./logo-small.png">


Introduction
------------

### Prerequisites

This tutorial assumes RabbitMQ is
[installed](https://www.rabbitmq.com/download.html) and running on
localhost on standard port (5672). In case you use a different host,
port or credentials, connections settings would require adjusting.


RabbitMQ is a message broker: it accepts and forwards messages. You can
think about it as a post office: when you put the mail that you want
posting in a post box, you can be sure that Mr. or Ms. Mailperson will
eventually deliver the mail to your recipient. In this analogy, RabbitMQ
is a post box, a post office and a postman.

The major difference between RabbitMQ and the post office is that it
doesn't deal with paper, instead it accepts, stores and forwards binary
blobs of data ‒ *messages*.

RabbitMQ, and messaging in general, uses some jargon.

-   *Producing* means nothing more than sending. A program that sends
    messages is a *producer* :

    ![](./1_files/producer.webp)

    digraph { bgcolor=transparent; truecolor=true; rankdir=LR; node
    [style="filled"]; // P1 [label="P", fillcolor="\#00ffff"]; }

-   *A queue* is the name for a post box which lives inside RabbitMQ.
    Although messages flow through RabbitMQ and your applications, they
    can only be stored inside a *queue*. A *queue* is only bound by the
    host's memory & disk limits, it's essentially a large message
    buffer. Many *producers* can send messages that go to one queue, and
    many *consumers* can try to receive data from one *queue*. This is
    how we represent a queue:

    ![](./1_files/queue.webp)

    digraph { bgcolor=transparent; truecolor=true; rankdir=LR; node
    [style="filled"]; // subgraph cluster\_Q1 { label="queue\_name";
    color=transparent; Q1 [label="{||||}", fillcolor="red",
    shape="record"]; }; }

-   *Consuming* has a similar meaning to receiving. A *consumer* is a
    program that mostly waits to receive messages:

    ![](./1_files/consumer.webp)

    digraph { bgcolor=transparent; truecolor=true; rankdir=LR; node
    [style="filled"]; // C1 [label="C", fillcolor="\#33ccff"]; }

Note that the producer, consumer, and broker do not have to reside on
the same host; indeed in most applications they don't. An application
can be both a producer and consumer, too.

Hello World!
------------

### (using the Pika Python client)

In this part of the tutorial we'll write two small programs in Python; a
producer (sender) that sends a single message, and a consumer (receiver)
that receives messages and prints them out. It's a "Hello World" of
messaging.

In the diagram below, "P" is our producer and "C" is our consumer. The
box in the middle is a queue - a message buffer that RabbitMQ keeps on
behalf of the consumer.

Our overall design will look like:

![](./1_files/python-one-overall.png)

digraph G { bgcolor=transparent; truecolor=true; rankdir=LR; node
[style="filled"]; // P1 [label="P", fillcolor="\#00ffff"]; subgraph
cluster\_Q1 { label="hello"; color=transparent; Q1 [label="{||||}",
fillcolor="red", shape="record"]; }; C1 [label="C",
fillcolor="\#33ccff"]; // P1 -\> Q1 -\> C1; }

Producer sends messages to the "hello" queue. The consumer receives
messages from that queue.

> #### RabbitMQ libraries
>
> RabbitMQ speaks multiple protocols. This tutorial uses AMQP 0-9-1,
> which is an open, general-purpose protocol for messaging. There are a
> number of clients for RabbitMQ in [many different
> languages](https://www.rabbitmq.com/devtools.html). In this tutorial
> series we're going to use [Pika
> 1.0.0](https://pika.readthedocs.org/en/stable/), which is the Python
> client recommended by the RabbitMQ team. To install it you can use the
> [pip](https://pip.pypa.io/en/stable/quickstart/) package management
> tool:
>
> ``` {.lang-bash .hljs}
> python -m pip install pika --upgrade
> ```
>
Now we have Pika installed, we can write some code.

### Sending

![](./1_files/sending.webp)

digraph { bgcolor=transparent; truecolor=true; rankdir=LR; node
[style="filled"]; // P1 [label="P", fillcolor="\#00ffff"]; subgraph
cluster\_Q1 { label="hello"; color=transparent; Q1 [label="{||||}",
fillcolor="red", shape="record"]; }; // P1 -\> Q1; }

Our first program send.py will send a single message to the queue. The
first thing we need to do is to establish a connection with RabbitMQ
server.

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
```

We're connected now, to a broker on the local machine - hence the
*localhost*. If we wanted to connect to a broker on a different machine
we'd simply specify its name or IP address here.

Next, before sending we need to make sure the recipient queue exists. If
we send a message to non-existing location, RabbitMQ will just drop the
message. Let's create a *hello* queue to which the message will be
delivered:

``` {.lang-python .hljs}
channel.queue_declare(queue='hello')
```

At this point we're ready to send a message. Our first message will just
contain a string *Hello World!* and we want to send it to our *hello*
queue.

In RabbitMQ a message can never be sent directly to the queue, it always
needs to go through an *exchange*. But let's not get dragged down by the
details ‒ you can read more about *exchanges* in [the third part of this
tutorial](https://www.rabbitmq.com/tutorials/tutorial-three-python.html).
All we need to know now is how to use a default exchange identified by
an empty string. This exchange is special ‒ it allows us to specify
exactly to which queue the message should go. The queue name needs to be
specified in the routing\_key parameter:

``` {.lang-python .hljs}
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello World!')
print(" [x] Sent 'Hello World!'")
```

Before exiting the program we need to make sure the network buffers were
flushed and our message was actually delivered to RabbitMQ. We can do it
by gently closing the connection.

``` {.lang-python .hljs}
connection.close()
```

> #### Sending doesn't work!
>
> If this is your first time using RabbitMQ and you don't see the "Sent"
> message then you may be left scratching your head wondering what could
> be wrong. Maybe the broker was started without enough free disk space
> (by default it needs at least 200 MB free) and is therefore refusing
> to accept messages. Check the broker logfile to confirm and reduce the
> limit if necessary. The [configuration file
> documentation](https://www.rabbitmq.com/configure.html#config-items)
> will show you how to set disk\_free\_limit.

### Receiving

![](./1_files/receiving.webp)

digraph { bgcolor=transparent; truecolor=true; rankdir=LR; node
[style="filled"]; // subgraph cluster\_Q1 { label="hello";
color=transparent; Q1 [label="{||||}", fillcolor="red", shape="record"];
}; C1 [label="C", fillcolor="\#33ccff"]; // Q1 -\> C1; }

Our second program receive.py will receive messages from the queue and
print them on the screen.

Again, first we need to connect to RabbitMQ server. The code responsible
for connecting to Rabbit is the same as previously.

The next step, just like before, is to make sure that the queue exists.
Creating a queue using queue\_declare is idempotent ‒ we can run the
command as many times as we like, and only one will be created.

``` {.lang-python .hljs}
channel.queue_declare(queue='hello')
```

You may ask why we declare the queue again ‒ we have already declared it
in our previous code. We could avoid that if we were sure that the queue
already exists. For example if send.py program was run before. But we're
not yet sure which program to run first. In such cases it's a good
practice to repeat declaring the queue in both programs.

> #### Listing queues
>
> You may wish to see what queues RabbitMQ has and how many messages are
> in them. You can do it (as a privileged user) using the rabbitmqctl
> tool:
>
> ``` {.lang-bash .hljs}
> sudo rabbitmqctl list_queues
> ```
>
> On Windows, omit the sudo:
>
> ``` {.lang-powershell .hljs}
> rabbitmqctl.bat list_queues
> ```
>
Receiving messages from the queue is more complex. It works by
subscribing a callback function to a queue. Whenever we receive a
message, this callback function is called by the Pika library. In our
case this function will print on the screen the contents of the message.

``` {.lang-python .hljs}
def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
```

Next, we need to tell RabbitMQ that this particular callback function
should receive messages from our *hello* queue:

``` {.lang-python .hljs}
channel.basic_consume(queue='hello',
                      auto_ack=True,
                      on_message_callback=callback)
```

For that command to succeed we must be sure that a queue which we want
to subscribe to exists. Fortunately we're confident about that ‒ we've
created a queue above ‒ using queue\_declare.

The auto\_ack parameter will be described [later
on](https://www.rabbitmq.com/tutorials/tutorial-two-python.html).

And finally, we enter a never-ending loop that waits for data and runs
callbacks whenever necessary.

``` {.lang-python .hljs}
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

### Putting it all together

send.py
([source](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/python/send.py))

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='hello')

channel.basic_publish(exchange='', routing_key='hello', body='Hello World!')
print(" [x] Sent 'Hello World!'")
connection.close()
```

receive.py
([source](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/python/receive.py))

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='hello')


def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)


channel.basic_consume(
    queue='hello', on_message_callback=callback, auto_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

Now we can try out our programs in a terminal. First, let's start a
consumer, which will run continuously waiting for deliveries:

``` {.lang-bash .hljs}
python receive.py
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received 'Hello World!'
```

Now start the producer. The producer program will stop after every run:

``` {.lang-bash .hljs}
python send.py
# => [x] Sent 'Hello World!'
```

Hurray! We were able to send our first message through RabbitMQ. As you
might have noticed, the receive.py program doesn't exit. It will stay
ready to receive further messages, and may be interrupted with Ctrl-C.

Try to run send.py again in a new terminal.

We've learned how to send and receive a message from a named queue. It's
time to move on to part 2 and build a simple *work queue*.
