<img align="right" src="./logo-small.png">



# Python code for RabbitMQ

To successfully use the examples you will need a running RabbitMQ server.

## Requirements

To run this code you need to install the `pika` package version `1.0.0` or later. To install it, run

    python -m pip install pika


## Code

[Tutorial one: "Hello World!"]  `<host-ip>:<port>/lab/workspaces/lab1_python`

    python send.py
    python receive.py


[Tutorial two: Work Queues] `<host-ip>:<port>/lab/workspaces/lab2_python`

    python new_task.py "A very hard task which takes two seconds.."
    python worker.py


[Tutorial three: Publish/Subscribe] `<host-ip>:<port>/lab/workspaces/lab3_python`

    python receive_logs.py
    python emit_log.py "info: This is the log message"


[Tutorial four: Routing] `<host-ip>:<port>/lab/workspaces/lab4_python`

    python receive_logs_direct.py info
    python emit_log_direct.py info "The message"


[Tutorial five: Topics] `<host-ip>:<port>/lab/workspaces/lab5_python`

    python receive_logs_topic.py "*.rabbit"
    python emit_log_topic.py red.rabbit Hello


[Tutorial six: RPC] `<host-ip>:<port>/lab/workspaces/lab6_python`

    python rpc_server.py
    python rpc_client.py
