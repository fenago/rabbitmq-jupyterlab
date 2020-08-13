
[]{#ch09}Chapter 9. Security {#chapter-9.-security .title}
----------------------------

</div>

</div>
:::

A system is as secure as its weakest component taking the message broker
into account. As RabbitMQ instances can be used to carry sensitive
application data or affect the stability of an entire system, we need to
make sure that our RabbitMQ deployments are secured properly.

The topics covered in this chapter are as follows:

::: {.itemizedlist}
-   Types of threats

-   Authentication

-   Authorization

-   Secure communication

-   Penetration testing



[]{#ch09lvl1sec57}Types of threats {#types-of-threats .title style="clear: both"}
----------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

There []{#id362 .indexterm}are several aspects in which the security of
the message broker is affected. RabbitMQ hasn\'t been planned to be
exposed on the Internet initially; however, a number []{#id363
.indexterm}of security concerns exist even with in-house deployments of
the message broker. We will stay away from this fact and not make
assumptions on whether the broker instances under consideration are
accessible via the Internet or not.

Let\'s consider again the standard three-cluster diagram (along with an
additional remote broker instance) that we have been using so that we
can see what security issues may arise in practice:

::: {.mediaobject}
![](2_files/4565OS_09_01.jpg)
:::

We can apply the following mechanisms in order to mitigate the
identified threats:

::: {.itemizedlist}
-   [**Authentication**]{.strong}: This[]{#id364 .indexterm} allows you
    to identify who connects to the message broker.

-   [**Authorization**]{.strong}: This []{#id365 .indexterm}allows you
    to determine the set of privileges and permissions for the
    authenticated user.

-   [**Secure communication between the clients and the
    broker**]{.strong}: By default, messages []{#id366 .indexterm}are
    exchanged by the senders/receivers and broker instances in an
    unsecure manner; however, RabbitMQ provides you with a mechanism to
    establish secure SSL communication.

-   [**Secure communication between cluster nodes**]{.strong}:
    Communication between the cluster nodes in[]{#id367 .indexterm} the
    form of Erlang messages is also unsecure, and SSL communication can
    be established between instance nodes in a RabbitMQ cluster.

-   [**Secure communication between remote nodes**]{.strong}:
    As[]{#id368 .indexterm} federation links and shovels provide a
    mechanism to mirror messages across instances over the WAN in a
    client-server fashion in an unsecure manner, you can establish SSL
    communication between them as well.

-   [**Message encryption**]{.strong}: If, by []{#id369 .indexterm}some
    chance, you cannot reliably secure all the message broker
    communication channels using SSL, you can encrypt the messages that
    are sent between the sender and consumer using a proper encryption
    mechanism (for example, asymmetric encryption with the RSA algorithm
    using a key of proper length, 2048, 4096, or others). Depending on
    the mechanism used and performance requirements of the application,
    there could be a trade-off between security and performance. This
    applies to the previous cases when SSL communication takes place as
    well.

-   [**Proper client settings**]{.strong}: When[]{#id370 .indexterm} we
    discussed performance tuning, we discussed a number of settings for
    resource utilization of the broker. Many of them can be applied in
    order to mitigate DoS or DDoS attacks that target resource
    exhaustion on the message broker by means of sending excessive
    number of messages, creating a huge number of connections (thus
    preventing other clients from connecting), or sending an excessive
    number of AMQP messages.

-   [**Physical security**]{.strong}: Physical access []{#id371
    .indexterm}to the workstations where the message broker is deployed
    should be properly restricted, and the disks where Mnesia tables
    reside should be properly encrypted in order to mitigate the risk of
    data leakage in case of theft (typically, in cases where the message
    broker stores sensitive data passed through messages).

-   [**Plugin security**]{.strong}: Plugins []{#id372 .indexterm}can
    also expose vulnerabilities, so it is important to use plugins from
    trusted sources that are updated on a regular basis or at least do
    proper verification that the plugin isn\'t doing something
    malicious.
:::

Vulnerability databases such as CVE (Common Vulnerabilities and
Exposures) along with other resources on the Internet could prove to be
good sources of information regarding known issues against which you can
check production deployments of the broker for possible security issues.

In the next sections, we will demonstrate other basic types of attacks
and how to get protection against them. Apart from the techniques, we
will demonstrate that you need to make sure that you have a message
broker upgrade plan set in place. The RabbitMQ team provides security
fixes with upcoming releases of the message broker.



[]{#ch09lvl1sec58}Authentication {#authentication .title style="clear: both"}
--------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Let\'s []{#id373 .indexterm}consider []{#id374 .indexterm}the default
setup of a RabbitMQ instance. It comes with a default `guest`{.literal}
user (with a `guest`{.literal} password) known by anyone with basic
knowledge about the broker. Moreover, this user has an
`administrator`{.literal} tag giving them full access to administer the
broker, and, even worse, if the RabbitMQ instance port is visible to the
outside world, remote commands can be executed using the
`rabbitmqctl`{.literal} utility on that workstation using the
`eval`{.literal} command. For this reason, it is advisable (not to say
mandatory) to remove the `guest`{.literal} user in production
deployments. Although the latest versions of RabbitMQ allow only
localhost access for the `guest`{.literal} user, this still imposes a
high risk for insider attacks. RabbitMQ stores information about users
in an internal database (in the same location where Mnesia stores
information about transient and persistent messages by default).
RabbitMQ authentication is provided by means of the []{#id375
.indexterm} [**SASL**]{.strong} ([**Simple Authentication and Security
Layer**]{.strong}) framework that allows the communicating endpoints to
negotiate authentication data before authentication actually takes
place. It is defined in the Internet standard RFC 4422. The following
diagram provides a high-level overview of how SASL works in terms of a
sender and the RabbitMQ message broker (note that the diagram is similar
for the message broker and consumer):

::: {.mediaobject}
![](3_files/4565OS_09_02.jpg)
:::

When the client[]{#id376 .indexterm} initiates a connection to a
RabbitMQ instance, the following things occur:

::: {.orderedlist}
1.  The message broker suggests one or more authentication method to the
    client. By default, the authentication method suggested by the
    server and supported by all the clients is PLAIN, which is the most
    basic type of authentication (equivalent to HTTP basic
    authentication). RabbitMQ client libraries also provide a mechanism
    to specify the SASL configuration for the client before trying to
    establish a connection with the message broker.

2.  The client selects one of the methods and sends this information
    back to the message broker.

3.  The client and server start exchanging security information by means
    of proper handlers depending on the authentication mechanism that is
    selected. As the SASL framework provides a mechanism for pluggable
    authentication, each particular authentication mechanism provides a
    set of server/client handlers to establish and exchange security
    data. The number of steps in this phase depends on the
    authentication method.

4.  After the authentication mechanism is negotiated and the client is
    authenticated successfully, the exchange of messages starts taking
    place. SASL provides a mechanism to establish the confidentiality
    and integrity of the messages exchanged between the client and
    server if this is negotiated by them in the previous steps.
:::

When a user is created from the management console, REST API, or the
`rabbitadmin`{.literal} script by default, it is stored as part of the
RabbitMQ instance and the information about the user is propagated among
the cluster nodes. In practice, however, the instance can be configured
to negotiate other types of authentication. If the instance is deployed
in an environment where many applications share the same credentials
(such as a large enterprise or even a system with multiple components),
then the instance may need to use an external service such as an
[]{#id377 .indexterm} [**LDAP**]{.strong} ([**Lightweight Directory
Access Protocol**]{.strong}) server or RDBMS for authentication. In this
case, you need to make sure that the same SASL configuration is applied
among the cluster nodes so that clients that need to reconnect to
another cluster node are able to negotiate and authenticate with the
same authentication mechanism as the one used when connecting to the
original RabbitMQ instance.

In practice, SASL can be implemented in a more general way that allows
the client to authenticate the server (in this case, RabbitMQ) but this
is not provided out of the box by RabbitMQ (although a plugin with
proper support by RabbitMQ clients can provide this support). Currently,
the following SASL methods are supported directly (more information is
present in the RabbitMQ documentation):

::: {.itemizedlist}
-   [**PLAIN**]{.strong}: This[]{#id378 .indexterm} is the default one

-   [**AMQPLAIN**]{.strong}: This[]{#id379 .indexterm} is the custom
    version of PLAIN as defined by the AMQP 0-8 standard

-   [**RABBIT-CR-DEMO**]{.strong}: This []{#id380 .indexterm}is the
    custom challenge response authentication

-   [**EXTERNAL**]{.strong}: This[]{#id381 .indexterm} is currently
    supported by means of `rabbitmq_auth_mechanism_ssl`{.literal} that
    provides the ability to authenticate a client using the client\'s
    public certificate
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch09lvl2sec72}Configuring the LDAP backend {#configuring-the-ldap-backend .title}

</div>

</div>
:::

Let\'s see, for []{#id382 .indexterm}example, how to move from the
default storage of RabbitMQ users to an LDAP server using the OpenLDAP
server distribution. First, download OpenLDAP for the operating system
of your choice (for Unix-based distribution, you either use the package
manager or go to <http://www.openldap.org/>, and[]{#id383 .indexterm}
for a Windows port, you can go to
<http://sourceforge.net/projects/openldapwindows/>). For a Ubuntu-based
installation, you need to install the `slapd`{.literal} and
`ldap-utils`{.literal} packages in order to install OpenLDAP using the
following command:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
sudo apt-get install slapd ldap-utils
```
:::

The Windows installation comes with a convenient installer. After the
LDAP server is installed in Windows, you can start the server by running
the `<OpenLDAP_install_dir>\libexec\StartLDAP.cmd`{.literal} script.
After the OpenLDAP server is started, navigate to the
`<OpenLDAP_install_dir>\sbin`{.literal} directory and run the following
utility in order to set a new root password for the LDAP server (the
same configuration applies to the other operating systems):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
slappasswd.exe
```
:::

You will be prompted to supply a proper root password. After you supply
the password twice, you will see it in an encrypted form. Assuming that
we have set an example as the root password, you can see the following:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
{SSHA}VUCblOSqFJn/L9O2bMTrP/YpGDJyAYYx
```
:::

Copy the encrypted password and apply it (the `rootpw`{.literal}
parameter) along with the name of your organization (the
`suffix`{.literal} parameter) and directory root (the `rootdn`{.literal}
parameter) to the
`<OpenLDAP_install_dir>\etc\openldap\slapd.conf`{.literal} OpenLDAP
configuration file as follows (modify the already existing parameters):

::: {.itemizedlist}
-   `suffix`{.literal}: dc=example,dc=com

-   `rootdn`{.literal}: cn=organization,dc=example,dc=com

-   `rootpw`{.literal}: {SSHA}VUCblOSqFJn/L9O2bMTrP/YpGDJyAYYx
:::

After the preceding configuration changes have been made, you need to
restart the OpenLDAP server. Note that the restart may fail in case
there is an `<OpenLDAP_install_dir>\etc\openldap\alock`{.literal} lock
file existing. If this is the case, delete the file and try to start the
OpenLDAP server again. Entries in a directory server are organized
hierarchically and the mechanisms to add / edit / delete or retrieve
them is provided by the LDAP protocol. LDAP and OpenLDAP are huge topics
that we will not cover in detail. For []{#id384 .indexterm}now, you can
assume that the preceding configuration specifies the root directory for
your entries along with the root password used to access this directory.
Definitions of the LDAP entries are stored in an `ldiff`{.literal} file.
Using the configured root and password, we will create the following
directory structure that has the organization at the root along with a
subentry that represents the group of users in the organization and a
single user (`Martin`{.literal}):

::: {.mediaobject}
![](3_files/4565OS_09_03.jpg)
:::

The structure is represented by the following `ldiff`{.literal} file
(`sample.ldiff`{.literal}):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
# This distinguished name (DN) determines the organization
dn: dc=example,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
dc: example            
o: example
description: Sample description

## Example.com users
dn: ou=users,dc=example,dc=com
ou: users
description: Users in the organization
objectClass: organizationalUnit

## Sample user
dn: cn=Martin,ou=users,dc=example,dc=com
objectclass: inetOrgPerson
cn: Martin
sn: Toshev
uid: mtoshev
mail: martin@example.com
```
:::

Now, you need to import the element definitions from the
`ldiff`{.literal} file using the
`<OpenLDAP_install_dir>\bin\ldapadd`{.literal} utility as follows:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
ldapadd -x -D "cn=organization,dc=example,dc=com" -W –f sample.ldiff
```
:::

Note that to delete entries, you can use the `ldapdelete`{.literal}
utility as follows (for example, if you want to remove the last entry
that we added):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
ldapdelete  -D "cn=organization,dc=example,dc=com" "cn= Martin,ou=users,dc=example,dc=com" -W
```
:::

RabbitMQ provides []{#id385 .indexterm}you with an LDAP backend by means
of the `rabbitmq-auth-backend-ldap`{.literal} plugin. As the LDAP plugin
is already included in the RabbitMQ distribution, you can simply enable
it on a node:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
rabbitmq-plugins enable rabbitmq_auth_backend_ldap
```
:::

After the plugin is enabled, you need to provide proper configuration
for the LDAP backend as part of the RabbitMQ configuration. Uncomment
the following line in order to enable the backend for a node (remember
to apply the same configuration over all the nodes in a cluster):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
{auth_backends, [rabbit_auth_backend_ldap]}
```
:::

In case you want to fall back to using the standard authentication
backend provided by RabbitMQ, you can also add the
`rabbit_auth_backend_internal`{.literal} entry to the list. Add the
following under the `rabbitmq_auth_backend_ldap`{.literal} section to
the configuration file:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
{servers, ["localhost"]},{user_dn_pattern, "cn=${username},ou=users,dc=example,dc=com"}{tag_queries, [{administrator, {constant, false}},{management,    {constant, true}}]}
```
:::

This specifies the hostname (`localhost`{.literal}) and user []{#id386
.indexterm} [**DN**]{.strong} ([**Distinguished Name**]{.strong})
pattern; in this case, this is the path of the LDAP entry containing
`${username}`{.literal}, which is replaced by RabbitMQ with the supplied
username. Note that there is an alternative mechanism that can bind the
username to an arbitrary attribute of the user. Refer to the RabbitMQ
LDAP plugin for more details on the alternative configuration. The last
section specifies that our users are able to access the management
console but they don\'t have administrative privileges. By default, all
LDAP users are non-administrative and are allowed access to the entire
broker (all the objects in all `vhosts`{.literal}). In the next section,
we will see how to configure additional permissions for LDAP users when
using the plugin (the authorization part of the plugin). Before being
able to log in using the preceding user DN pattern, we must set a
password for our users. To set a password for the user with the name
`Martin`{.literal} that we created earlier, you can use the
`ldappasswd`{.literal} utility as follows (specify the encrypted form of
the example that we used earlier to configure our root LDAP password):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
ldappasswd -D "cn=organization,dc=example,dc=com" "cn=Martin,ou=users,dc=example,dc=com" –W -S
```
:::

Now, in order to check whether a user can successfully authenticate, you
can take the DN from the RabbitMQ configuration, replace
`${username}`{.literal} with the name of the user (in this case, Martin)
that you want to check, and use the `ldapwhoampi`{.literal} utility as
follows:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
ldapwhoami -vvv -D "cn=Martin,ou=users,dc=example,dc=com" -x -w example
```
:::

You should see the following if the test succeeds:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
dn:cn=Martin,ou=users,dc=example,dc=com
Result: Success (0)
```
:::

You should []{#id387 .indexterm}now be able to log in from the
management console with the `Martin`{.literal} user and
`example`{.literal} password. If you omit the `tag_queries`{.literal}
entry from the preceding configuration, you will see a warning similar
to the following in the log file when you attempt a login (indicating
that the LDAP user is not allowed to access the management console):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
HTTP access denied: user 'Martin' - Not management user
```
:::

The LDAP plugin provides additional configurations such as SSL support
for the LDAP communication; you can refer to the plugin documentation.
The authentication backend can be used with other types of SASL
authentication such as EXTERNAL, as we will see later in this chapter.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch09lvl2sec73}Security considerations {#security-considerations .title}

</div>

</div>
:::

Having []{#id388 .indexterm}configured the proper authentication
mechanisms and removing the default user is merely not enough. Simple
passwords are easily guessable and a very basic tool can be created
based on a RabbitMQ client library that tries to connect to the broker
using a list of pregenerated passwords from a proper source that can be
used to execute a brute force attack on a RabbitMQ message broker. For
this reason, you need to consider the following:

::: {.itemizedlist}
-   Setting strong passwords for RabbitMQ users whether they are stored
    internally in the broker or in an LDAP server.

-   Setting a broker threshold on the number of failed login attempts
    for a RabbitMQ user, which, at the time of writing this, is not
    supported directly by the message broker unfortunately. However,
    with some more effort, a good plugin can be contributed that
    implements a way to configure and enforce password policies.

-   Setting SSL communication in order to prevent password sniffing, as
    we will cover later in this chapter.

-   Deploying a proper monitoring solution on the broker workstation
    that takes into consideration the resource utilization factors that
    can indicate a security breach, such as increased memory or CPU time
    consumption.

-   Configuring a log auditing tool and storing audit logs for the
    auditing access. You can further combine the audit logs with a log
    analyzer that can scan them for []{#id389 .indexterm}possible
    security breaches. Unfortunately, RabbitMQ does not have such
    built-in capabilities or plugins; you can either decide to implement
    a plugin for the purpose or use the utilities provided by the OS
    (such as [**tcpdump**]{.strong} or [**iptables**]{.strong} logging
    rules for Unix-based operating systems) with proper log auditing
    tools in order to be able to analyze incoming traffic for the
    message broker.



[]{#ch09lvl1sec59}Authorization {#authorization .title style="clear: both"}
-------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

After a []{#id390 .indexterm}client is successfully authenticated by the
message broker, it needs to perform some activities in some virtual
hosts. In the earlier chapters, we saw that []{#id391
.indexterm}permissions are defined per vhost and live either internally
in the message broker or externally. The RabbitMQ LDAP backend plugin
that we saw earlier provides you with an ability to store permissions in
an LDAP server. The following types of permissions are configured in the
message broker:

::: {.itemizedlist}
-   [**configure**]{.strong}: This allows a resource to be created,
    modified, or deleted

-   [**write**]{.strong}: This allows a resource to be written to

-   [**read**]{.strong}: This allows a resource to be read from
:::

We already discussed how to manage permissions using the
`rabbitmqctl`{.literal} utility and the HTTP API. The following commands
can be used from the utility to manage permissions:

::: {.itemizedlist}
-   `set_permissions`{.literal}: This sets permissions per user per
    vhost

-   `clear_permissions`{.literal}: This clears permissions per user per
    vhost

-   `list_permissions`{.literal}: This lists the users that are granted
    access to a particular vhost along with their permissions

-   `list_user_permissions`{.literal}: This lists the permissions of a
    particular user
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch09lvl2sec74}LDAP authentication {#ldap-authentication .title}

</div>

</div>
:::

The LDAP []{#id392 .indexterm}user that we created earlier by default
has all the permissions to the broker (except for being an
administrator). Let\'s suppose that we want to disable the
`configure`{.literal} permissions, allow more fine-grained
`write`{.literal} permissions only to certain queues (in certain
vhosts), or make it an administrator. The RabbitMQ LDAP provides a query
mechanism to check permissions as configured in the LDAP server. There
are three types of queries that can be specified in the RabbitMQ
configuration and further contain different types of subqueries that are
executed against the LDAP server:

::: {.itemizedlist}
-   `vhost_access_query`{.literal}: As[]{#id393 .indexterm} users and
    permissions must be checked against vhosts that must be created in
    RabbitMQ, we can define vhost entries in the LDAP server against
    which to check for available permissions and tags. In fact, these
    entries represent a subset of the existing vhosts in the RabbitMQ
    server against which we check whether users have further access
    permissions or not. The default query is
    `{constant, true}`{.literal}, which specifies that access to all
    vhosts is given to all the users (the `constant`{.literal} queries
    are aliases for all, which return true or false for any value
    checked by `vhost_access_query`{.literal}).

-   `resource_access_query`{.literal}: These[]{#id394 .indexterm} are
    the types of queries that allow you to check whether a user has
    specific permissions (read, write, or configure) for a particular
    vhost to which the user has access (as checked by
    `vhost_access_query`{.literal}). The default is
    `{constant, true}`{.literal}.

-   `tag_queries`{.literal}: These []{#id395 .indexterm}are the types of
    queries that allow you to specify the tags that are given to
    particular users (such as [**management**]{.strong} or
    [**administrator**]{.strong}). The default is
    `{administrator, {constant, false}}`{.literal}.
:::

The types []{#id396 .indexterm}of subqueries that can be specified for
each type of these queries use a simple DSL; you can review the LDAP
RabbitMQ plugin documentation for an extensive list of all types of
subqueries. We will specify the following access domains for our message
broker:

::: {.itemizedlist}
-   The `test`{.literal} user is the vhost

-   The `guest`{.literal} user is an administrator and has access to the
    management console

-   The `Martin`{.literal} user has access only to the `test`{.literal}
    vhost and can publish to exchanges starting with the
    `test_`{.literal} prefix

-   The `Subscriber`{.literal} user has access to the `test`{.literal}
    vhost only and can read messages from queues starting with the
    `test_`{.literal} prefix
:::

The following diagram specifies the LDAP structure of the organization:

::: {.mediaobject}
![](4_files/4565OS_09_04.jpg)
:::

Before we can implement this setup, we need to create the test vhost in
RabbitMQ. The following example creates the test vhost using the
`rabbitmqctl`{.literal} utility:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
rabbitmqctl add_vhost test
```
:::

You also need to []{#id397 .indexterm}create LDAP entries for the
`guest`{.literal} and `Subsciber`{.literal} users in the same manner
that we created the entry for the user with the name Martin earlier.
Here is a sample `ldiff`{.literal} file (`users.ldiff`{.literal}) for
the two users:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
## guest user
dn: cn=guest,ou=users,dc=example,dc=com
objectclass: inetOrgPerson
cn: guest
sn: guest
uid: guest
mail: guest@example.com

## Subscriber user
dn: cn=Subscriber,ou=users,dc=example,dc=com
objectclass: inetOrgPerson
cn: Subscriber
sn: Subscriber
uid: Subscriber
mail: subscriber@example.com
```
:::

To import the preceding ldiff file and set a password for the users, you
can execute the following set of commands:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
ldapadd -x -D "cn=organization,dc=example,dc=com" -W –f users.ldiff
ldappasswd -D "cn=organization,dc=example,dc=com" "cn=guest,ou=users,dc=example,dc=com" –W –S
ldappasswd -D "cn=organization,dc=example,dc=com" "cn=Subscriber,ou=users,dc=example,dc=com" –W –S
```
:::

Finally, we need to create the `vhosts`{.literal} group along with an
entry for the `test`{.literal} vhost (`vhosts.ldiff`{.literal}):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
## Example.com vhosts
dn: ou=vhosts,dc=example,dc=com
ou: vhosts
description: Vhosts in the organization
objectClass: organizationalUnit

## test vhost
dn: cn=test,ou=vhosts,dc=example,dc=com
objectclass: organizationalRole
description: test vhost
```
:::

Execute the following in order to import the preceding entries:

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
ldapadd -x -D "cn=organization,dc=example,dc=com" -W –f vhosts.ldiff
```
:::

Note that we are []{#id398 .indexterm}using a predefined object class
(`organizationalRole`{.literal}) for the vhost entry in LDAP. You can
prefer to create your own object class for the purpose of describing a
vhost along with its attributes in your organization. Finally, we need
to specify the proper queries for permission checking in the LDAP
configuration (as part of the `rabbitmq_auth_backend_ldap`{.literal}
section in your RabbitMQ configuration file):

::: {.informalexample}
::: {.toolbar .clearfix}
Copy
:::

``` {.programlisting .language-markup}
{vhost_access_query,    {exists, "cn=${vhost},ou=vhosts,dc=example,dc=com"}},
{resource_access_query,
   {for, [ 
    {permission, configure, {match, 
{string, "${username}"},{string, "guest"}}},
    {permission, write, {'and', [
    {match, {string, "${username}"},{string, "(Martin|guest"}},
    {match, {string, "${name}"},{string, "test_*"}}]} },
    {permission, read, {'and', [
    {match, {string, "${username}"},{string, "(Subscriber|guest)"}},
    {match, {string, "${name}"},{string, "test_*"}}]} }
    ] }},
{tag_queries, [{administrator, {match, {string, "${username}"},{string, "guest"}}},
               {management,    {constant, true}}]}
```
:::

The preceding configuration is easy to understand, but it might turn out
to be clumsy to write and test. After it is added to the configuration
file, you can try to log in with the guest/guest user and check whether
it has administrative access. You can try to create an object using the
`Subscriber`{.literal} user or send/receive messages using the
`Martin`{.literal}/`Subscriber`{.literal} user. In practice, the
preceding configuration should be designed carefully based on the
organizational LDAP schema in order to prevent security holes.



### []{#ch09lvl2sec76}Secure cluster communication {#secure-cluster-communication .title}

</div>

</div>
:::

Although []{#id410 .indexterm}an Erlang cookie is used to allow
communication between nodes in a cluster, it still doesn\'t enable
secure communication between these nodes. To do so, you need to enable
SSL in the Erlang application that runs the RabbitMQ instance. For
further details on how to enable SSL communication between nodes, you
can check the clustering SSL guide from the RabbitMQ documentation:

<http://www.erlang.org/doc/apps/ssl/ssl_distribution.html>

Apart from setting secure clustering, you must also ensure that
filesystem access to the Erlang cookie is given only to users who are
allowed to run and manage the RabbitMQ instance on that system.
:::

::: {.section lang="en" lang="en"}
::: {.titlepage}
<div>

<div>

### []{#ch09lvl2sec77}EXTERNAL SSL authentication {#external-ssl-authentication .title}

</div>

</div>
:::

The `rabbitmq-auth-mechanism-ssl`{.literal} context provides an
`SASL EXTERNAL`{.literal} type of authentication []{#id411
.indexterm}that uses a client certificate to authenticate a user. The
plugin requires SSL communication to be enabled between the client and
RabbitMQ server. For more details about the configuration of the plugin,
you can check the plugin repository at
<https://github.com/rabbitmq/rabbitmq-auth-mechanism-ssl>.



[]{#ch09lvl1sec61}Penetration testing {#penetration-testing .title style="clear: both"}
-------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Now that []{#id412 .indexterm}we have seen how to secure our message
broker, we also need to test that our setup is indeed in place and
really prevents attackers from bringing []{#id413 .indexterm}down the
message broker or stealing messages. For this reason, you can build your
own custom tool for penetration testing of the message broker, which
performs the following functions:

::: {.itemizedlist}
-   It checks whether the guest/guest user is present and it can perform
    administrative activities.

-   It tries to brute-force passwords for an existing set of users,
    either based on a password generation policy or using a predefined
    password database.

-   It tries to access prohibited vhosts from a particular set of users.

-   It uses nmap to check whether the management console and RabbitMQ
    communication ports are visible; this step may include checks on
    ports that are exposed by RabbitMQ plugins.

-   It checks[]{#id414 .indexterm} the RabbitMQ configuration settings,
    authentication []{#id415 .indexterm}mechanism, and currently-set
    limits such as minimum free disk space, memory limits, or maximum
    number of channels. (Most of these options were covered when we
    discussed the performance tuning of the message broker.)

-   It checks the maximum limit per user as specified by the operating
    system, for example, this could be the maximum number of processes
    or file descriptors that can be used for the user that runs the
    RabbitMQ instance. In Linux, this could be checked against the
    `etc/security/limits.conf`{.literal} file.
:::

More features can be derived from the following article that covers
several security considerations and resource utilization settings for
production deployments of RabbitMQ:
<https://www.rabbitmq.com/production-checklist.html>



[]{#ch09lvl1sec62}Case study -- securing CSN {#case-study-securing-csn .title style="clear: both"}
--------------------------------------------

</div>

</div>

------------------------------------------------------------------------
:::

Once the []{#id416 .indexterm}CSN was in alpha testing and the good
performance of the system was reached, the CSN team was required to take
two important steps in order to meet the company\'s security policy:

::: {.itemizedlist}
-   Enable SSL over all the communication links between all the
    components in the system (including the RabbitMQ cluster links,
    federation link with the remote RabbitMQ instance, and communication
    links between the broker and its clients)

-   Enable the central management of CSN users and their associated
    permissions by means of the corporate LDAP server
:::

Further security testing was made by the team in order to ensure that no
major vulnerabilities were found in the setup of the system:

::: {.mediaobject}
![](7_files/4565OS_09_10.jpg)



[]{#ch09lvl1sec63}Summary {#summary .title style="clear: both"}
-------------------------

</div>

</div>

------------------------------------------------------------------------
:::

In this chapter, we discussed the various aspects of security related to
RabbitMQ and the types of vulnerabilities that can come up in practice
and how to mitigate them. We covered the SASL mechanism provided by
RabbitMQ for the purpose of authentication and extended further on this
concept by providing an integration of the authentication backend with
the OpenLDAP server. Additionally, we discussed how to store and manage
permissions in LDAP and provide secure communication with the message
broker, management console, and cluster nodes. In the end, we covered
several guidelines in establishing a successful penetration testing
strategy to verify that the message broker meets the minimum level of
security as required by the policy of your organization.



[]{#ch09lvl1sec64}Exercises {#exercises .title style="clear: both"}
---------------------------

</div>

</div>

------------------------------------------------------------------------
:::

::: {.orderedlist}
1.  What types of security threats are imposed on the message broker?

2.  What is SASL, and what types of SASL authentication are supported in
    RabbitMQ?

3.  How does RabbitMQ enable authentication and authorization against an
    LDAP server?

4.  How does RabbitMQ provide SSL support?

5.  How can you test whether your RabbitMQ setup provides a good degree
    of security?
