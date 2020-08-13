

Lab 6. Managing Your RabbitMQ Server
-------------------------------------------------


After talking about the details of the plugins and plugin development,
we are now ready to look into the management of the RabbitMQ server. To
get the best out of RabbitMQ, we need to manage it effectively. RabbitMQ
provides support for the following:


-   Adding, updating, and showing users, virtual hosts, and permissions
-   Declaring, listing, and deleting exchanges, queues, and bindings
-   Sending and receiving messages
-   Monitoring the queue length, message rates globally and per channel,
    data rates per connection, and so forth
-   Exporting/importing object definitions to JSON
-   Forcing close connections
-   Purging queues


We can manage the RabbitMQ server using command-line tool called
`rabbitmqctl` using a plugin called [*Management
Plugin*], that is provided by default from RabbitMQ and
accessing RabbitMQ using the REST APIs. Therefore, our lab is
designed with the following topics:


-   Management via a command line
-   Management via a web plugin
-   Management via a REST API

#### Start RabbitMQ Server
Run following command in the terminal to start rabbitmq server:

`service rabbitmq-server start`

`rabbitmqctl status`

#### RabbitMQ UI

Run following command in the terminal one by one to create rabbitmq user and allow UI access:

```
rabbitmqctl add_user fenago fenago

rabbitmqctl set_user_tags fenago administrator

rabbitmqctl set_permissions -p / fenago ".*" ".*" ".*"
```


`<host-ip>:<port>:15672`

**Username:** fenago

**Password:** fenago


#### Pre-reqs:
- Google Chrome (Recommended)

Management via a command line
-----------------------------------------------



The most  powerful tool for managing RabbitMQ is
[**rabbitmqctl**], which is a command-line application that
comes with the default RabbitMQ server installation bundle. Using the
RabbitMQ  control tool is really simple; you just
need to run the tool with its parameters.


### Cluster commands


In the  following table, we 
present cluster commands:

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/1.PNG)

### User commands


In the following  table, we present 
user commands:

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/2.PNG)


### Virtual host and permission commands


In the following  table, we 
present virtual host and permission commands:

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/3.PNG)

### Miscellaneous commands


In the  following table, we 
present miscellaneous commands:

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/4.PNG)

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/5.PNG)


As shown earlier, the table that lists all the parameters, we provide
the parameters to `rabbitmqctl`, and then we execute the
command as the screenshots showed.

The `rabbitmqctl` command is really enough for managing all
the parts of the RabbitMQ servers. Therefore, it is widely used in the
RabbitMQ users.


Management via a web plugin
---------------------------------------------



After talking the  details of the management of
RabbitMQ using a command-line tool, we are now ready to talk 
about the management plugin. [**Management plugin**]
is simply a web application that is written in Erlang. You can monitor
and control RabbitMQ using the management web interface. Management
plugin is provided as default by the RabbitMQ installation; however, you
need to enable the management plugin to use it by performing the
following steps:


1.  Enable the management plugin with the help of the
    `rabbitmq-plugins` command:



    ``` {.programlisting .language-markup}
    rabbitmq-plugins enable rabbitmq_management
    ```
    

2.  You should restart the RabbitMQ Server with the following command:



    ``` {.programlisting .language-markup}
    rabbitmqctl stop
    rabbitmq-server
    ```
    

3.  Now, you are ready to open the management dashboard with the
    following URL:

    `http://{your-ip-address}:15672/`

4.  The RabbitMQ server gives you a default username and password, that
    is, `guest:guest`. Note that `guest:guest`
    won\'t work for remote a RabbitMQ server later than version 3.3.


After locating the, management URL, you can see a dashboard, as shown
here:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image6_chapter6.jpg)


Dashboard of Management Web Interface



In the dashboard interface, you  can view the
RabbitMQ server statistics and information related to current 
connections, channels, exchanges, queues, and consumers.
Additionally, the dashboard can be used for monitoring.
Moreover, you have a menu on the header side
that redirects to the detailed part of each module.

After clicking on the [**Connections**] tab on the menu, you
can see the [**Connections**] module in the following image:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image7_chapter6.jpg)


Connections



You can find the related  information for each of
the connections. Moreover, you are allowed to close the connection with
the help of the [**Connections**] web page. After clicking on
the related  connection, you can find a button
titled [**Force Close**]. Whenever you click on this button,
the connection will be closed forcibly.

The following image simply describes the [**Channels**] tab and
its web page:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image8_chapter6.jpg)


Channels



In the [**Channels**] web page, you can find the related
information about each channel. Each channel is monitored with its
message rates, connections, and so on in this web page.

The following image shows the  
[**Exchanges**] module and its web page in the Management
plugin:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image9_chapter6.jpg)


Exchanges



In the [**Exchanges**] web page, we can monitor all of the
exchanges in the current RabbitMQ server with its related 
information. Moreover, if you click on each exchange, you\'ll
get detailed information about the clicked exchange item. You can
publish and delete a message through a selected exchange. Furthermore,
you can delete the selected exchange as well. In the
[**Exchanges**] web page, you are allowed to add a new
exchange.

The following image shows the [**Queues**] web page:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image10_chapter6.jpg)


Queues



In the [**Queues**] web page, you can  find
the list of the queues and their related information. Moreover, you can
add a new queue with the [**Add a new queue**] option and fill
in the details in the  required tabs. If you click
on one of the queues in this web page, you can find the detailed
information about the selected queue. Furthermore, you can add a new
routing key for binding, publishing, and getting a message from the
queue.

Let\'s talk about the last item of our Management plugins called the
[**Users**] web page, which is as shown in the following image:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image11_chapter6.jpg)


Users



Our last web page is the admin part that is accessible for only
administrators. You can see the details of each user 
and add a new user with the help of the [**Admin**]
web page. Additionally, the [**Admin**] web 
page gives us an opportunity to control and manage virtual
hosts and policies as well.



Management via a REST API
-------------------------------------------



Our last choice of  management and controlling the
RabbitMQ is using the [**REST API**]. RabbitMQ supports REST
API  to get lots of information from the RabbitMQ
server and add, edit, and delete some parameters and properties on it.

As REST services rely on the HTTP protocol, we can easily communicate
with RabbitMQ using web pages with [**AJAX**], HTTP clients on
every language, and so on. We\'d like to show the examples that use
RabbitMQ\'s REST API, using the [**Postman**] that is a REST
client for Google Chrome. Postman is a free extension on Google Chrome,
and you can add the Postman using the extension market of Google Chrome.

Before diving into the REST APIs, we\'d like to talk about the
authentication issue and return of the REST API after solving the issue.
REST API of the RabbitMQ uses basic authentication and returns only
[**JSON**] format. Therefore, we should configure our custom
monitoring and managing tool with respect to these authentication and
resource types. Lastly, RabbitMQ uses `55672` as a default
port for the REST API port.

With the REST API, we can access the  overview
information about the RabbitMQ server. You should provide a username and
password for basic authentication and just add the related URL for an
overview of the REST API. Now, you are ready to send the request using
the [**Send**] button, ah shown in the following image:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image12_chapter6.jpg)


Overview Request



As we examined the screenshot of the overview result of REST API, we
easily found the information and its statistics for RabbitMQ. We also
found a similar view using the dashboard of the management web page.

Let\'s now move on to the queues  and their details
with the following image:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image13_chapter6.jpg)


Queues Request



The queues service simply returns the list of all the queues, their
information, and statistics. We can use these statistics to monitor our
queues.

The following screenshot describes the connections service and type is
JSON. The connections service simply returns the statistics and
information about the current connections, which are 
established on the RabbitMQ server:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image14_chapter6.jpg)


Connections Request



Similarly, we can control and monitor the channels using the REST API.
As you can see in the following screenshot, we  can
fetch the information and statistics about the channels in the RabbitMQ
server:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image15_chapter6.jpg)


Channels Request



Statistics and information about the  bindings can
be easily fetched from the REST API as well.


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image16_chapter6.jpg)


Bindings Request



We sometimes need to view the permissions of the user within the
RabbitMQ server instance. With the help of RabbitMQ\'s REST service, we
can easily fetch and show the permissions of the user as shown in
 the following screenshot:


![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/image17_chapter6.jpg)


Permissions Request



As seen in the screenshots, both the services and results are in the
JSON format. We can easily integrate them with our software system using
the service-oriented architectures.

The RabbitMQ REST API has lots of services. Therefore, we can\'t show
all of the services within screenshots of the each REST service. So, it
is good to list all the services in a table. The following table just
shows each service of the REST API of RabbitMQ with its HTTP methods and
its description that explains the functionality of these parameters:

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/6_1.PNG)

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/7.PNG)

![](https://raw.githubusercontent.com/fenago/rabbitmq-jupyterlab/master/images/images_mastering/8.PNG)

Summary
-------------------------



As we have seen, managing RabbitMQ is easy with the tools it provides.
RabbitMQ gives us three opportunities to manage it using its
command-line tool `rabbitmqctl`, Management plugin, and REST
API. Therefore, we are comfortably managing our RabbitMQ server
instances using the RabbitMQ provided structures.

In the next lab, we will introduce the monitoring of the RabbitMQ
server instances, such as monitoring the resource usage, monitoring the
internal structures of RabbitMQ, and so on.
