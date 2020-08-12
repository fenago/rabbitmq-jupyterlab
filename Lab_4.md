<img align="right" src="./logo-small.png">


Routing
-------

Using the Pika Python client

#### Pre-reqs:
- Google Chrome (Recommended)

#### Lab Environment
All packages have been installed. There is no requirement for any setup.

All Notebooks are present in `work/rabbitmq-jupyterlab` folder.

You can access jupyter lab at `<host-ip>:<port>/lab/workspaces/lab4_python`

To copy and paste: use **Control-C** and to paste inside of a terminal, use **Control-V**

Run following command to move into python dssource files directory:

`cd /home/jovyan/work/rabbitmq-jupyterlab/python`

### What This Tutorial Focuses On

In the [previous
tutorial](https://www.rabbitmq.com/tutorials/tutorial-three-python.html)
we built a simple logging system. We were able to broadcast log messages
to many receivers.

In this tutorial we're going to add a feature to it - we're going to
make it possible to subscribe only to a subset of the messages. For
example, we will be able to direct only critical error messages to the
log file (to save disk space), while still being able to print all of
the log messages on the console.

Bindings
--------

In previous examples we were already creating bindings. You may recall
code like:

``` {.lang-python .hljs}
channel.queue_bind(exchange=exchange_name,
                   queue=queue_name)
```

A binding is a relationship between an exchange and a queue. This can be
simply read as: the queue is interested in messages from this exchange.

Bindings can take an extra routing\_key parameter. To avoid the
confusion with a basic\_publish parameter we're going to call it a
binding key. This is how we could create a binding with a key:

``` {.lang-python .hljs}
channel.queue_bind(exchange=exchange_name,
                   queue=queue_name,
                   routing_key='black')
```

The meaning of a binding key depends on the exchange type. The fanout
exchanges, which we used previously, simply ignored its value.

Direct exchange
---------------

Our logging system from the previous tutorial broadcasts all messages to
all consumers. We want to extend that to allow filtering messages based
on their severity. For example we may want the script which is writing
log messages to the disk to only receive critical errors, and not waste
disk space on warning or info log messages.

We were using a fanout exchange, which doesn't give us too much
flexibility - it's only capable of mindless broadcasting.

We will use a direct exchange instead. The routing algorithm behind a
direct exchange is simple - a message goes to the queues whose binding
key exactly matches the routing key of the message.

To illustrate that, consider the following setup:

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images//direct-exchange.webp)

In this setup, we can see the direct exchange X with two queues bound to
it. The first queue is bound with binding key orange, and the second has
two bindings, one with binding key black and the other one with green.

In such a setup a message published to the exchange with a routing key
orange will be routed to queue Q1. Messages with a routing key of black
or green will go to Q2. All other messages will be discarded.

Multiple bindings
-----------------

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images//direct-exchange-multiple.webp)

It is perfectly legal to bind multiple queues with the same binding key.
In our example we could add a binding between X and Q1 with binding key
black. In that case, the direct exchange will behave like fanout and
will broadcast the message to all the matching queues. A message with
routing key black will be delivered to both Q1 and Q2.

Emitting logs
-------------

We'll use this model for our logging system. Instead of fanout we'll
send messages to a direct exchange. We will supply the log severity as a
routing key. That way the receiving script will be able to select the
severity it wants to receive. Let's focus on emitting logs first.

Like always we need to create an exchange first:

``` {.lang-python .hljs}
channel.exchange_declare(exchange='direct_logs',
                         exchange_type='direct')
```

And we're ready to send a message:

``` {.lang-python .hljs}
channel.basic_publish(exchange='direct_logs',
                      routing_key=severity,
                      body=message)
```

To simplify things we will assume that 'severity' can be one of 'info',
'warning', 'error'.

Subscribing
-----------

Receiving messages will work just like in the previous tutorial, with
one exception - we're going to create a new binding for each severity
we're interested in.

``` {.lang-python .hljs}
result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

for severity in severities:
    channel.queue_bind(exchange='direct_logs',
                       queue=queue_name,
                       routing_key=severity)
```

Putting it all together
-----------------------

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images//python-four.png)

emit\_log\_direct.py
([source](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/python/emit_log_direct.py))

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='direct_logs', exchange_type='direct')

severity = sys.argv[1] if len(sys.argv) > 1 else 'info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'
channel.basic_publish(
    exchange='direct_logs', routing_key=severity, body=message)
print(" [x] Sent %r:%r" % (severity, message))
connection.close()
```

receive\_logs\_direct.py
([source](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/python/receive_logs_direct.py))

``` {.lang-python .hljs}
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='direct_logs', exchange_type='direct')

result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

severities = sys.argv[1:]
if not severities:
    sys.stderr.write("Usage: %s [info] [warning] [error]\n" % sys.argv[0])
    sys.exit(1)

for severity in severities:
    channel.queue_bind(
        exchange='direct_logs', queue=queue_name, routing_key=severity)

print(' [*] Waiting for logs. To exit press CTRL+C')


def callback(ch, method, properties, body):
    print(" [x] %r:%r" % (method.routing_key, body))


channel.basic_consume(
    queue=queue_name, on_message_callback=callback, auto_ack=True)

channel.start_consuming()
```

If you want to save only 'warning' and 'error' (and not 'info') log
messages to a file, just open a console and type:

``` {.lang-bash .hljs}
python receive_logs_direct.py warning error > logs_from_rabbit.log
```

If you'd like to see all the log messages on your screen, open a new
terminal and do:

``` {.lang-bash .hljs}
python receive_logs_direct.py info warning error
# => [*] Waiting for logs. To exit press CTRL+C
```

And, for example, to emit an error log message just type:

``` {.lang-bash .hljs}
python emit_log_direct.py error "Run. Run. Or it will explode."
# => [x] Sent 'error':'Run. Run. Or it will explode.'
```

Move on to tutorial 5 to find out how to listen for messages based on a pattern.
