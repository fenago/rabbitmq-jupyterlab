

[]{#ch03}Chapter 3. Managing RabbitMQ {#chapter-3.-managing-rabbitmq .title}
-------------------------------------

</div>

</div>
:::

In this chapter we will cover:

::: {.itemizedlist}
-   Using vhosts

-   Configuring users

-   Using SSL

-   Implementing client-side certificates

-   Managing RabbitMQ from a browser

-   Configuring RabbitMQ parameters

-   Developing Python applications to monitor RabbitMQ

-   Developing your own web applications to monitor RabbitMQ



[]{#ch03lvl1sec32}Introduction {#introduction .title style="clear: both"}
------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Once installed, RabbitMQ[]{#id136 .indexterm} just works; it\'s really a
zero configuration service. However, RabbitMQ has a lot of configuration
options, which make it very flexible and able to work in different
environments. In this chapter we will see how to change the
configuration to meet the requirements of your application.

We will also begin to use the tools that we will treat in detail in the
following chapters in order to show you how to monitor RabbitMQ with an
application.



[]{#ch03lvl1sec33}Using vhosts {#using-vhosts .title style="clear: both"}
------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

With [**virtual hosts**]{.strong} ([**vhosts**]{.strong}), it[]{#id137
.indexterm} is possible to have many different, independent virtual
brokers within one single RabbitMQ instance. In this way, it is possible
to use the same broker on the parts of many different applications
without worrying about name clashes. This is the same approach used by
web servers with virtual hosts.

You can find the simple Java example in the
`Chapter03/Recipe01 `{.literal}directory, which is identical to the
example in the first recipe of the book, except for the usage of the
vhost.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec100}Getting ready {#getting-ready .title}

</div>

</div>
:::

To exercise this recipe, you just need to issue some commands at the
Linux command prompt, that is, [**RabbitMQ Command Prompt
(sbindir)**]{.strong} in the Windows Start menu.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec101}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

To create a new []{#id138 .indexterm}vhost, perform the following steps:

::: {.orderedlist}
1.  List available virtual hosts with the command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl list_vhosts
    ```
    :::

2.  Create a new virtual host `book_orders`{.literal} by issuing the
    command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl add_vhost book_orders
    ```
    :::

3.  List its exchanges:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl list_exchanges -p book_orders
    ```
    :::

4.  List user permissions:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl list_permissions
    rabbitmqctl list_permissions -p book_orders
    ```
    :::

5.  Allow the user, `guest`{.literal}, to access the
    `book_orders`{.literal} vhost:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl set_permissions guest .* .* .* -p book_orders
    ```
    :::

6.  Use it with the Java client library:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    factory.setVirtualHost("book_orders");
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec102}How it works... {#how-it-works .title}

</div>

</div>
:::

Once installed, RabbitMQ has []{#id139 .indexterm}just the default vhost
`/`{.literal} defined, as we can easily check with the command in step
1.

We then create a new vhost by issuing the command
`rabbitmqctl add_vhost`{.literal} (step 2). After that, we must issue
all the commands related to this new vhost by specifying it with the
`–p`{.literal} option. If this is omitted, the commands are applied to
the default vhost.

The new vhost you have added cannot be used yet. A quick check via
listing permissions (step 4) will show that the new vhost has no
authorization to perform any action. Then we give the predefined user,
`guest`{.literal}, all the permissions in the context of the
`book_orders`{.literal} vhost, as we will see in the next recipe (step
5).

At this point, it is enough to specify the vhost to the connection
factory (step 6) in order to let a RabbitMQ client connect to it instead
of the default vhost.



[]{#ch03lvl1sec34}Configuring users {#configuring-users .title style="clear: both"}
-----------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

RabbitMQ[]{#id140 .indexterm} users are []{#id141 .indexterm}global to
the broker instance, but each user can have his/her own set of
permissions for each individual vhost.

Different applications can be totally decoupled using independent users
and vhosts.

However, the same application can benefit from the usage of user
permissions within the same vhost.

We are going to see how to manage users and their permissions and how to
use them in the Java example in `Chapter03/Recipe02`{.literal}.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec103}Getting ready {#getting-ready .title}

</div>

</div>
:::

In order to run this recipe, we need to issue some
`rabbitmqctl`{.literal} configuration commands and exercise the
configurations using the usual Java setup.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec104}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

Perform the following steps to see how to manage users and their
permissions as well as how to use them:

::: {.orderedlist}
1.  Create some users with their passwords:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl add_user stat_sender password1
    rabbitmqctl add_user stat_receiver password2
    ```
    :::

2.  Grant some permissions to them:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl set_permissions stat_sender "stat_exchange.*""stat_.*" "^$"
    rabbitmqctl set_permissions stat_receiver "stat_.*""stat_queue_.*" "(stat_exchange_.*)|(stat_queue_.*)"
    ```
    :::

3.  Let `Producer.java`{.literal} connect to RabbitMQ using the
    `stat_sender`{.literal} user:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    factory.setUsername("stat_sender");
    factory.setPassword("password1");
    ```
    :::

4.  And then perform the following operations:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    channel.exchangeDeclare(...);
    channel.basicPublish(...);
    ```
    :::

5.  Let `Consumer.java`{.literal} connect using the user,
    `stat_receiver`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    factory.setUsername("stat_receiver");
    factory.setPassword("password2");
    ```
    :::

6.  Then perform the following operations:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    channel.exchangeDeclare(...);
    channel.queueDeclare(...);
    channel.queueBind(...);
    channel.basicConsume(...);
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec105}How it works... {#how-it-works .title}

</div>

</div>
:::

In this[]{#id142 .indexterm} exercise []{#id143 .indexterm}we have
created a couple of users (step 1). In order to use them, it is needed
to assign permission to them with the command:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
rabbitmqctl set_permissions <username> <conf> <write> <read>
```
:::

Here, `<conf>`{.literal}, `<write>`{.literal}, and `<read>`{.literal}
are three regular expressions. The specified `<username>`{.literal} will
have the permission to configure, write, and read queues and the
exchanges matching them.

In our example, `Producer.java`{.literal} accesses RabbitMQ with the
`stat_sender`{.literal} user. As shown in step 4, it has called
`queueDeclare()`{.literal}; so, it needs to have configuration
permission for the exchange named `stat_exchange_03/02`{.literal}.

It then publishes messages to the same exchange, so the user needs to
have write permissions to it. But then messages will be routed to the
`stat_queue_03/02`{.literal} queue; the user also needs write
permissions to it or the messages won\'t be routed to this queue. By
setting the write permission to the regular expression
`"stat_.*"`{.literal}, the user is authorized to publish messages to
both the exchange and the queue.

`Producer.java`{.literal} does not need any read permission. It is
possible to deny any read permission by specifying the empty regular
expression `"^$"`{.literal}, as shown in the example, or just an empty
string.

On the other side, the user of `Consumer.java`{.literal} needs the
configure permission on both the exchange and the queue as well as the
read permission in our recipe: `"stat_.*"`{.literal} is synonymous to
`"(stat_exchange_.*)|(stat_queue_.*)"`{.literal} in the given example.

`Consumer.java`{.literal} also needs write permissions on the
`stat_queue_03/02`{.literal} queue to be able to bind the given queue to
the exchange.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec106}There\'s more... {#theres-more .title}

</div>

</div>
:::

With users, we can restrict the access and capabilities of different
RabbitMQ clients, tuning the set of permissions available to different
components of a distributed application and conforming to their roles.

We can set a different set of permissions to each user, which applies to
queues and exchanges specifying a pattern following regular expression
matching rules.

Permission roles[]{#id144 .indexterm} refer to the authorization to
access specific AMQP resources:

::: {.itemizedlist}
-   `configure`{.literal}: The authorized user can declare and delete
    queues and exchanges that match the given pattern

-   `write`{.literal}: The authorized user can bind to queues and
    exchanges and publish messages to them if they match the given
    pattern

-   `read`{.literal}: The authorized user can bind to matching queues
    and exchanges and consume, get, and purge messages out of them
:::

For the full table of the authorizations needed by the various AMQP
operations, you can use the table at
<http://www.rabbitmq.com/access-control.html>.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

#### []{#ch03lvl3sec07}User tags for the management plugin {#user-tags-for-the-management-plugin .title}

</div>

</div>
:::

User permissions[]{#id145 .indexterm} refer just to AMQP operations. The
RabbitMQ management plugin (more details later in this chapter) extends
this permission model using user tags; it\'s possible to associate one
or more arbitrary tag strings to any user.

To let a user access the management plugin, it must have one of the
following tags associated to it:

::: {.itemizedlist}
-   `management`{.literal}: The user has access to his/her vhosts only
    and can view all the queues and the exchanges within them

-   `monitoring`{.literal}: The user has access to all the vhosts and
    can view all the queues and exchanges

-   `administration`{.literal}: The user can do any administrative task
:::

For example, to assign the administrator tag to a user, use the command:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
rabbitmqctl set_user_tags username administrator
```
:::

The default user, `guest`{.literal}, has the `administrator`{.literal}
tag set by default.



[]{#ch03lvl1sec35}Using SSL {#using-ssl .title style="clear: both"}
---------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Whenever the []{#id146 .indexterm}RabbitMQ broker is exposed to the
Internet, it is highly advisable to protect the connections by using the
SSL library.

RabbitMQ does not implement SSL by itself, but it uses the certificate
mechanism of the given language, Erlang for the server and Java, .NET,
or whatever for the clients.

Here, we will see how to have a basic protection with SSL, that is, how
to encrypt the connections from the RabbitMQ clients to the broker.

This is enough to avoid simple security attacks. With no SSL, usernames
and passwords are sent just in clear through the network.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec107}Getting ready {#getting-ready .title}

</div>

</div>
:::

The current example has the following prerequisites:

::: {.itemizedlist}
-   A Linux OS hosting the RabbitMQ broker

-   openssl Linux package

-   The latest Erlang distribution---at least R14B

-   Java JDK on the client, either on Linux or Windows
:::

We have chosen to limit this recipe to just Linux on the server side
because on Windows, there are too many version combinations---some with
limited or no functionality at all. It is a good idea to run your
secured Internet facing RabbitMQ servers on Linux or a Unix OS.

For more information on Windows support, you can go to
<http://www.rabbitmq.com/ssl.html>.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec108}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

Perform the following steps to set up the Certificate Authority and
configure the server:

::: {.orderedlist}
1.  On the server, create the[]{#id147 .indexterm} [**Certificate
    Authority**]{.strong} ([**CA**]{.strong}) directory stub as per the
    script in the example path
    `Chapter03/Recipe03/certificates/01_setup_CA.sh`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    mkdir testca
    cd testca
    mkdir certs private
    chmod 700 private
    echo 01 > serial
    touch index.txt
    ```
    :::

2.  Customize the `openssl`{.literal} configuration file, which you can
    already find in
    `Chapter03/Recipe03/certificates/testca/openssl.cnf`{.literal}.

3.  Create the self-signed CA certificates as done in
    `Chapter03/Recipe03/certificates/02_create_CA_certificates.sh`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl req -x509 -config openssl.cnf -newkeyrsa:2048 -days 365 -out cacert.pem -outform PEM -subj /CN=MyTestCA/ -nodes
    openssl x509 -in cacert.pem -out cacert.cer -outform DER
    ```
    :::

4.  Create the[]{#id148 .indexterm} RabbitMQ server private key as in
    `Chapter03/Recipe03/certificates/03_create_server_certificates.sh`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl genrsa -out key.pem 2048
    ```
    :::

5.  Create the server certificate request:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl req -new -key key.pem -out req.pem -outform PEM -subj /CN=$(hostname)/O=server/ -nodes
    ```
    :::

6.  In the CA directory, sign the request to obtain the signed
    certificate:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl ca -config openssl.cnf -in ../server/req.pem –out ../server/cert.pem -notext -batch –extensions server_ca_extensions
    ```
    :::

7.  Copy[]{#id149 .indexterm} from
    `Chapter03/Recipe03/certificates`{.literal}, the CA certificate, the
    server certificate, and the server private key, which we have just
    created, to the absolute paths:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    /usr/local/certificates/testca/cacert.pem
    /usr/local/certificates/server/cert.pem
    /usr/local/certificates/server/key.pem
    ```
    :::

8.  Create the RabbitMQ configuration file, `rabbitmq.config`{.literal},
    in the appropriate directory (`/etc/rabbitmq`{.literal}) by copying
    it from `Chapter03/Recipe03/rabbitmq.config`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    [
    {rabbit, [
    {ssl_listeners, [5671]},
    {ssl_options, [
    {cacertfile,"/usr/local/certificates/testca/cacert.pem"},
    {certfile,"/usr/local/certificates/server/cert.pem"},
    {keyfile,"/usr/local/certificates/server/key.pem"},
    {verify,verify_peer},
    {fail_if_no_peer_cert,false}]}
    ]}
    ].
    ```
    :::

9.  Restart the RabbitMQ server:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl stop
    rabbitmq-server–detached.
    ```
    :::

    When you restart a RabbitMQ node, the Erlang Node will also restart.

10. In the []{#id150 .indexterm}Java client, the connection to the
    server is now made, as shown in
    `Chapter03/Recipe03/src/rmqexample/Publish.java`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost(hostname);
    factory.setPort(5671);
    factory.useSslProtocol();
    Connection connection = factory.newConnection();
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec109}How it works... {#how-it-works .title}

</div>

</div>
:::

We started this []{#id151 .indexterm}recipe by creating a CA with which
we will sign the certificates for the server.

We have performed this step on the server, but in the real world, the CA
and, in particular, its private key (created in step 3) should be kept
separate.

After the CA is ready, we have to prepare the server certificate as
shown in steps 4-6.

We are almost done. We just need to copy the CA certificate, the server
certificate, and the server public key to the final path (step 7). We
have chosen to store them in `/usr/local/certificates`{.literal}, but it
is totally arbitrary since they are referenced in the RabbitMQ
configuration file, `rabbitmq.config`{.literal}.

This file does not exist by default. It must be placed in the standard
configuration directory, usually in `/etc/rabbitmq`{.literal}.

Apart from the security files, we have configured the RabbitMQ SSL port
(5671), and a couple of options in `rabbitmq.config`{.literal}:

::: {.itemizedlist}
-   `Verify`{.literal}: When this is set to `verify_peer`{.literal}, it
    tells RabbitMQ that if the client presents a certificate, it will be
    checked and rejected if not valid (because the CA is not the same or
    because it is expired)

-   `fail_if_no_peer_cert`{.literal}: When this is set to
    `false`{.literal}, it tells RabbitMQ to accept clients that do not
    present any certificate at all
:::

After we have restarted RabbitMQ (you must use `rabbitmqctl`{.literal}
stop and restart the service), you can verify whether it has got the
options by examining the logfile in `/var/log/rabbitmq`{.literal}. You
should be able to find a line as follows:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
started SSL Listener on [::]:5671
```
:::

Furthermore, by opening the management plugin from a browser, you will
be able to get similar information (see the [*Managing RabbitMQ from a
browser*]{.emphasis} recipe), as shown in the following screenshot:

::: {.mediaobject}
![](5_files/6501OS_03_09.jpg)
:::

At this point it is []{#id152 .indexterm}possible to connect via SSL
from a client by just adding these two options to the connection
factory:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
factory.setPort(5671);
factory.useSslProtocol();
```
:::

Now the connection is encrypted using the keys configured in the server.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec110}There\'s more... {#theres-more .title}

</div>

</div>
:::

Since we are using server certificates only, the communication between
the server and the client is encrypted, but we are not protected
against[]{#id153 .indexterm} [**MITM**]{.strong}
([**man-in-the-middle**]{.strong}) attacks.

If we want to let any client connect to the server and avoid MITM
attacks, we can use the same strategy as that used by HTTPS and the
browsers, which is signing the certificates with trusted third-party CAs
and verifying the domain signed in the certificates. Otherwise, we can
just go on and read the next recipe.



[]{#ch03lvl1sec36}Implementing client-side certificates {#implementing-client-side-certificates .title style="clear: both"}
-------------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In case the RabbitMQ broker and client communicate through the Internet,
it sounds reasonable that only authorized clients can connect to the
broker.

This is the scope of []{#id154 .indexterm}typical user password
authentication, but by using, in addition, client-side certificates, the
security of the distributed application is highly improved. It also
avoids the possibility of MITM attack.

This recipe is the extension/prosecution of the previous one. So, we
assume that we already have the CA set up and the server configured, as
shown previously.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec111}Getting ready {#getting-ready .title}

</div>

</div>
:::

This recipe is just an extension of the previous one---the same
recommendations apply.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec112}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

Perform the following steps for the client to be able to connect to the
RabbitMQ server:

::: {.orderedlist}
1.  Copy the certificates and the keys, created in the previous recipe,
    from the `Chapter03/Recipe04/certificates`{.literal} directory:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    cp –rp ../Recipe03/certificates/testca .
    cp –rp ../Recipe03/certificates/server .
    ```
    :::

2.  In the client certificate directory, create the client private key,
    as shown in
    `Chapter03/Recipe04/certificates/04_create_client_certificates.sh`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl genrsa -out key.pem 2048
    ```
    :::

3.  Create a certificate signing request:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl req -new -key key.pem -out req.pem -outform PEM 
    -subj /CN=$(hostname)/O=client/ -nodes
    ```
    :::

4.  In the CA directory, sign the client certificate:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl ca -config openssl.cnf -in ../client/req.pem 
    -out ../client/cert.pem -notext -batch -extensions client_ca_extensions
    ```
    :::

5.  Back in the client directory, create a `PKCS#12`{.literal} store
    containing the client certificate and the key protected by a
    password:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    openssl pkcs12 -export -out keycert.p12 -in cert.pem 
    -in keykey.pem -passout pass:client1234passwd
    ```
    :::

6.  Create a Java key store containing the server certificate protected
    with a password, as shown in
    `Chapter03/Recipe04/certificates/05_create_keystore.sh`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    keytool -importcert -alias server001 -file server/cert.pem -keystore keystore/rabbit.jks -keypass passwd1234
    ```
    :::

7.  Change the `rabbitmq.config`{.literal} option
    `fail_if_no_peer_cert`{.literal} to `true`{.literal}:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    {fail_if_no_peer_cert,true}
    ```
    :::

8.  Restart RabbitMQ:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmqctl stop
    rabbitmq-server –detached
    ```
    :::

9.  On the client []{#id155 .indexterm}side, set up a secure connection
    by setting up the SSL context:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    char[] keyPassphrase = "client1234passwd".toCharArray();
    KeyStoreks = KeyStore.getInstance("PKCS12");
    ks.load(newFileInputStream("certificates/client/keycert.p12"), keyPassphrase);

    KeyManagerFactorykmf = KeyManagerFactory.getInstance("SunX509");
    kmf.init(ks, keyPassphrase);

    char[] trustPassphrase = "passwd1234".toCharArray();  
    KeyStoretks = KeyStore.getInstance("JKS");
    tks.load(newFileInputStream("certificates/keystore/rabbit.jks"), trustPassphrase);

    TrustManagerFactorytmf = TrustManagerFactory.getInstance("SunX509");
    tmf.init(tks);

    SSLContext c = SSLContext.getInstance("SSLv3");
    c.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);

    ConnectionFactory factory = newConnectionFactory();
    factory.setHost(hostname);
    factory.setPort(5671);
    factory.useSslProtocol(c);
    Connection connection = factory.newConnection();
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec113}How it works... {#how-it-works .title}

</div>

</div>
:::

For the client certificate to work, it must be signed with the same CA
that has been used to sign the server. Once the certificate is prepared,
it is very useful to save it in a keystore, a `PKCS#12`{.literal} store
as shown in step 5.

The client needs the server certificate too---it contains the server
public key---and so we have prepared a keystore for this one too using a
Java keystore with the Java keytool command this time.

Then we reconfigured RabbitMQ (steps 6-7). In this way, the server will
deny access if the client has not presented any certificate. You can
easily check this by running the previous example now.

Once the client certificates are ready, the RabbitMQ client (step 8)
must use them to set up SSLContext. Unlike the previous example, we pass
the following call:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
factory.useSslProtocol(c);
```
:::

Only a client[]{#id156 .indexterm} following this setup can connect to
the RabbitMQ server. The client presenting these certificates cannot
connect to a server that has a different server private key since the
traffic is being encrypted using the server public key stored in the
server certificate.



[]{#ch03lvl1sec37}Managing RabbitMQ from a browser {#managing-rabbitmq-from-a-browser .title style="clear: both"}
--------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In this recipe we\'re showing you how to admin RabbitMQ from an HTTP API
using a Management Plugin.

The plugin provides real-time charts to monitor the flow of your
messages. Furthermore, it provides HTTP APIs to analyze RabbitMQ. This
is required by external monitoring systems such as []{#id157
.indexterm}Ganglia (<http://ganglia.sourceforge.net/>), []{#id158
.indexterm}Puppet
([http://puppetlabs.com](http://puppetlabs.com/){.ulink}), and others in
order to perform their activities.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec114}Getting ready {#getting-ready .title}

</div>

</div>
:::

You just need a web browser.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec115}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

In order to[]{#id159 .indexterm} enable the plugin you have to perform
the[]{#id160 .indexterm} following steps:

::: {.orderedlist}
1.  Issue the following command:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    rabbitmq-plugins enable rabbitmq_management
    ```
    :::

2.  Restart RabbitMQ.

3.  The plugin enables a web server that is accessible via the URL
    `http://localhost:15672/`{.literal}. Replace `localhost`{.literal}
    with your RabbitMQ hostname/IP to access from another machine.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec116}How it works... {#how-it-works .title}

</div>

</div>
:::

By default, the web application uses `guest`{.literal}/`guest`{.literal}
as the RabbitMQ users\' username/password. Web management is very
intuitive, and you can manage queues, exchanges, users, []{#id161
.indexterm}connections, and virtual hosts and also send []{#id162
.indexterm}and receive messages.

::: {.mediaobject}
![](7_files/6501OS_03_01.jpg)
:::

On the first tab, you can find the system overview with the queued
messages, message rates, and global count.

Then, you can find node description. Click on the node to get more
details:

::: {.mediaobject}
![](7_files/6501OS_03_10.jpg)
:::

In the previous []{#id163 .indexterm}screenshot you can see some metrics
that can help[]{#id164 .indexterm} you to diagnose eventual problems.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip17}Tip {#tip .title}

If you are running RabbitMQ on Windows, you need to install the
`Hanlde.exe`{.literal} SysInternals tool
(<http://technet.microsoft.com/en-us/sysinternals/bb896655>) in an
executable path such as `C:/Windows/system32`{.literal}. Otherwise, the
console won\'t be able to show the count of the file descriptors.
:::

On the same page, there is other information about the active Erlang
modules with their version numbers.

In the lower part of the overview, you can check the listening ports of
the AMQP broker and the various installed plugins (web contexts):

::: {.mediaobject}
![](7_files/6501OS_03_02.jpg)
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec117}There\'s more... {#theres-more .title}

</div>

</div>
:::

At `http://localhost:15672/api/`{.literal}, you can get access to the
HTTP API. By using the HTTP APIs, the users can create a custom console
as we are going to see in the recipe [*Developing Python applications to
monitor RabbitMQ*]{.emphasis}.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec118}See also {#see-also .title}

</div>

</div>
:::

At <http://www.rabbitmq.com/management.html>, you can find the full
documentation about the console.



[]{#ch03lvl1sec38}Configuring RabbitMQ parameters {#configuring-rabbitmq-parameters .title style="clear: both"}
-------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In this recipe, we are []{#id165 .indexterm}going to introduce the
RabbitMQ parameters. By default, the[]{#id166 .indexterm} broker
doesn\'t create the configuration files []{#id167 .indexterm}because in
most of the cases you don\'t need to change them. However, it\'s
important to know how to configure the environment variables and the
broker parameters.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec119}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

In RabbitMQ, you can configure the [**environment
variables**]{.strong}[]{#id168 .indexterm} and the server file
configuration. With the environment variables, you can change parameters
such as the server port or the node name. There are two ways to change
these variables:

::: {.orderedlist}
1.  Define the variables in your shell environment.

2.  Create a file `rabbitmq-env.conf`{.literal} located in
    `/etc/rabbitmq`{.literal}.
:::

If, for example, you want to change the RabbitMQ node name, you have to
do the following:

::: {.orderedlist}
1.  Stop the server.

2.  Either issue `exportRABBITMQ_NODENAME=mylittlerabbit`{.literal} on
    the shell or insert the string `NODENAME=mylittlerabbit`{.literal}
    in the file, `/etc/rabbitmq/rabbitmq-env.conf`{.literal}.

3.  Restart the broker.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec120}How it works... {#how-it-works .title}

</div>

</div>
:::

Originally, the[]{#id169 .indexterm} web []{#id170 .indexterm}management
shows the default node []{#id171 .indexterm}name,
`rabbit@hostname`{.literal}, as follows:

::: {.mediaobject}
![](8_files/6501OS_03_07.jpg)
:::

After our configuration, it shows `mylittlerabbit@hostname`{.literal} as
shown in the following screenshot:

::: {.mediaobject}
![](8_files/6501OS_03_08.jpg)
:::

In this way, you can change all the parameters that you can find in the
URL
<https://www.rabbitmq.com/configure.html#define-environment-variables>.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip18}Tip {#tip .title}

The environment variable name is prefixed with `RABBITMQ_`{.literal}.
You do not have to use it if you write the variable in
`rabbitmq-env.conf`{.literal}.
:::

With the []{#id172 .indexterm}server file configuration, you can change
the[]{#id173 .indexterm} internal broker configuration. The
configuration []{#id174 .indexterm}file is `rabbitmq.config`{.literal}
and the location is the same as that of `rabbitmq-env.conf`{.literal}.
Use the `RABBITMQ_CONFIG_FILE`{.literal} environment variable to change
the location. You can find the complete list of parameters at
<https://www.rabbitmq.com/configure.html#configuration-file>.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec121}There\'s more... {#theres-more .title}

</div>

</div>
:::

In this recipe we have just introduced the RabbitMQ configuration. In
the next chapters, we will change some of the default parameters to tune
the performance or configure the cluster.



[]{#ch03lvl1sec39}Developing Python applications to monitor RabbitMQ {#developing-python-applications-to-monitor-rabbitmq .title style="clear: both"}
--------------------------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In this example we\'re going to create a Python script to monitor
RabbitMQ using the JSON API that you can access from the URL
`http://localhost:15672/api/`{.literal}.

The scope of this example is to create a custom Python script, which
performs some checks and sends an e-mail if it detects some errors. You
can find the source code at `Chapter03/Recipe07`{.literal}.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec122}Getting ready {#getting-ready .title}

</div>

</div>
:::

You need Python 2.7+ and the management plugin enabled (as we have seen
in the [*Managing RabbitMQ from a browser*]{.emphasis} recipe).
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec123}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

Perform the following []{#id175 .indexterm}steps to create a Python
script to monitor RabbitMQ using the JSON API:

::: {.orderedlist}
1.  Import the following libraries:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    import sys
    import urllib2,base64
    import json
    import logging
    ```
    :::

2.  Get the required RabbitMQ information from the JSON API:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    urllib2.Request("http://"+rabbitmqhost+":15672/api/nodes")
    urllib2.Request("http://"+rabbitmqhost+":15672/api/connections")
    urllib2.Request("http://"+rabbitmqhost+":15672/api/aliveness-test/%2f")
    ```
    :::

3.  Load and evaluate the JSON result:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    json.load(urllib2.urlopen(request));
    ```
    :::

4.  Send an e-mail if there is some error (or a condition for which you
    want to be alerted):

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    if error_message != "":
      sendAlarmStatus(error_message);
    ```
    :::
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec124}How it works... {#how-it-works .title}

</div>

</div>
:::

With the `/api/nodes`{.literal} JSON call, we check the
`running state`{.literal} and the `memory alarm`{.literal} for each
RabbitMQ node. With `/api/connections`{.literal},` `{.literal}we are
monitoring the connected clients. The `/api/aliveness-test/`{.literal}
JSON call creates a test queue and tries to send and receive a message
on the default virtual host `%2f`{.literal} (this URL encoding is needed
when referring to the character \"/\" in an URL, with \"/\" being the
default AMQP virtual host). The result is `{"status": "ok"}`{.literal}
if there are no errors.

The script needs to authenticate the API using the code:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
base64.encodestring('%s:%s' % ('guest', 'guest'))request.add_header("Authorization", "Basic %s" % base64string);
```
:::

::: {.note style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#note07}Note {#note .title}

You must use a RabbitMQ user with monitor rights.
:::

If some checks fail, the script sends an e-mail with the identified
problems. To try this example, you need to modify the function
`sendAlarmStatus(message)`{.literal} with your e-mail account
credentials.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip19}Tip {#tip .title}

The script can be scheduled with the Linux crontab or using Scheduler
Task in Windows.
:::

Our example[]{#id176 .indexterm} writes the logs in the
`/var/tmp/myMonitorRMQ.log`{.literal} file.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec125}There\'s more... {#theres-more .title}

</div>

</div>
:::

In this example we have checked just a few APIs. Obviously there are
more, which let you create a monitoring tool to check the RabbitMQ
health customized depending on your requirements.

You can also use `rabbitmqadmin`{.literal} to monitor the broker: the
file is downloadable from the broker itself at
`http://localhost:15672/cli/`{.literal}. This is a Python script that
can be invoked from a shell script, for example:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
./rabbitmqadmin –f raw_json list nodes 
```
:::

This command will return the node information in the JSON format to the
stdout, as we have already seen in step 2 of this recipe.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec126}See also {#see-also .title}

</div>

</div>
:::

You can find the whole API documentation at
<http://hg.rabbitmq.com/rabbitmq-management/raw-file/rabbitmq_v3_0_4/priv/www/api/index.html>.



[]{#ch03lvl1sec40}Developing your own web applications to monitor RabbitMQ {#developing-your-own-web-applications-to-monitor-rabbitmq .title style="clear: both"}
--------------------------------------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In this recipe we\'ll show you []{#id177 .indexterm}how to create a
custom web application to monitor the RabbitMQ logs. In order to check
the logs, you can bind a queue to the `amq.rabbitmq.log`{.literal}
exchange, as we will see in [Chapter
12](https://subscription.packtpub.com/book/application_development/9781849516501/12){.link},
[*Managing RabbitMQ Error Conditions*]{.emphasis}, with much more
details.

You can find the source code for this recipe in
`Chapter03/Recipe08`{.literal}.

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec127}Getting ready {#getting-ready .title}

</div>

</div>
:::

In order to understand this example, we will make use of the following:

::: {.itemizedlist}
-   Spring

-   Apache Tomcat

-   Apache Maven

-   WebSocket

-   Query/HTML5/Twitter Bootstrap
:::

We recommend the Spring tool suite. You will need Internet access for
the Maven, bootstrap, and jQuery includes.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec128}How to do it... {#how-to-do-it .title}

</div>

</div>
:::

Perform the following []{#id178 .indexterm}steps to create a custom web
application to monitor the RabbitMQ logs:

::: {.orderedlist}
1.  Create a basic [**MVC**]{.strong} ([**Model View
    Controller**]{.strong}) Spring project using the available template
    from the tool as shown in the following screenshot:

    ::: {.mediaobject}
    ![](10_files/6501OS_03_03.jpg)
    :::

2.  Modify the Maven file (`POM.xml`{.literal}) by adding the following
    dependencies:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    tomcat-coyote   // needed for websockets
    tomcat-servlet-api// needed for websockets
    tomcat-catalina// needed for websockets
    spring-rabbit// needed for rabbitmq
    ```
    :::

3.  Create a new bean `RabbitMQInstance`{.literal} to create an
    interface with RabbitMQ using the
    `org.springframework.amqp`{.literal} package.

    You can find the life cycle of bean in `root-context.xml`{.literal}.
    In this example RabbitMQ must be run on the same host as Tomcat, but
    you can change the connection parameters.

4.  Create a new class called `RmqWebSocket`{.literal} that extends
    `WebSocketServlet`{.literal} (Tomcat WebSocket class) and add the
    `@WebServlet ("/websocket")`{.literal} annotation.

5.  Create a private `amqp`{.literal} consumer inside the
    `RmqWebSocket`{.literal} class:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    LogListenerimplements MessageListener. 
    ```
    :::

6.  The consumer[]{#id179 .indexterm} starts using
    `SimpleMessageListenerContainer`{.literal} as follows:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    container.setConnectionFactory(rmq.getConnectionFactory());
    ...
    container.start();
    ```
    :::

7.  Bind the browser WebSocket to the server using the code:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    ws = new WebSocket('ws://' host':8080/rmq/websocket');
    ```
    :::

8.  Subscribe the browser using the code:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    ws.onmessage = function(message){..} 
    ```
    :::

9.  Each time the browser gets a message, it updates the HTML table
    using jQuery:

    ::: {.informalexample}
    ::: {.toolbar .clearfix}
    Copy
    :::

    ``` {.programlisting .language-markup}
    $("#tablebody").prepend("<tr><td>"+message.data+ "</td></tr>");
    ```
    :::

10. Now, we are ready. You can build Maven and deploy to the Tomcat
    container.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec129}How it works... {#how-it-works .title}

</div>

</div>
:::

In the bean `RabbitMQInstance`{.literal}, we create and bind a queue to
the topic exchange `amq.rabbitmq.log`{.literal} with the routing key
`#`{.literal}. In the `root-context.xml`{.literal} file, we define the
bean ID and the init/deinit methods in order to connect AMQP clients
during the web application startup and disconnect during the shutdown.

The RabbitMQ connection and queue are ready.

The `RmqWebSocket`{.literal} is a `Tomcat websocket`{.literal}, and here
we start a consumer using the `SimpleMessageListenerContainer`{.literal}
implementation with the `rabbitMQInstance`{.literal} parameters
(connection and queue).

The consumer is ready.

With the method `StreamInbound`{.literal} in `RmqWebSocket`{.literal},
we register the `websocket`{.literal} connection by the browser
instantiating a new `ClientMessage`{.literal} class for each connection
and the `Collection<ClientMessage> clients`{.literal} code line contains
the active clients.

The `websocket`{.literal} backend is ready.

In step 7 the browser is []{#id180 .indexterm}connected to the server
and now it\'s ready, as shown in the following figure:

::: {.mediaobject}
![](10_files/6501OS_03_06.jpg)
:::

So when a log message is published to the topic exchange
`amq.rabbitmq.log`{.literal}, the class `LogListener`{.literal} raises
an event:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
public void onMessage(Message message) {..}
```
:::

The message is broadcasted to the connected browser with a
`for`{.literal} loop to the `clients`{.literal} collection:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
for(ClientMessage item: clients){
  CharBuffer buffer = CharBuffer.wrap(new String(message.getBody()) );
item.myoutbound.writeTextMessage(buffer);
```
:::

Now the message is received by the browser, which updates the local HTML
table with `jquery`{.literal}.

Tomcat, by []{#id181 .indexterm}default, uses the `8080`{.literal} port
and the URL application is `/rmq`{.literal}. The complete URL is
`http://yourtomcatmachine:8080/rmq/`{.literal} and you should have a web
browser pointing to this address as shown:

::: {.mediaobject}
![](10_files/6501OS_03_04.jpg)
:::

We have used HTML5 and Bootstrap, and you can also use a mobile device.

::: {.note style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#note08}Note {#note .title}

The connecting or disconnecting of a client is enough to raise a
standard log event.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec130}There\'s more... {#theres-more .title}

</div>

</div>
:::

In this example, we have introduced many heterogeneous technologies in
order to present a realistic example on how to merge RabbitMQ and
current web technologies.

Here we have used RabbitMQ log messages, but you can easily apply the
same architecture to any messaging need of your distributed/web
application.

::: {.tip style="margin-left: 0.5in; margin-right: 0.5in;"}
### []{#tip20}Tip {#tip .title}

In order to try to run the example directly, you can go to the source
code path in the book archive
(`Chapter03`{.literal}/`Recipe08`{.literal}) and prepare a package using
Maven with [**mvn package**]{.strong}. Then deploy the package to
Tomcat.
:::
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch03lvl2sec131}See also {#see-also .title}

</div>

</div>
:::

::: {.itemizedlist}
-   Spring: <http://spring.io/>

-   Apache Tomcat: <http://tomcat.apache.org/>

-   Apache Maven: <http://maven.apache.org/>

-   WebSocket: <http://www.websocket.org/>

-   <http://tomcat.apache.org/tomcat-7.0-doc/web-socket-howto.html>

-   jQuery: <http://jquery.com/>

-   HTML5: <http://en.wikipedia.org/wiki/HTML5>

-   Twitter Bootstrap: <http://getbootstrap.com/>
