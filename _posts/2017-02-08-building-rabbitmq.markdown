---
layout: post
title:  "Building RabbitMQ from sources (and dependencies)"
date:   2017-02-08 19:50:00
categories: 
tags: rabbitmq messaging stomp amqp erlang clustering 
---

<H1>Building RabbitMQ </H1>

I tryed to install &amp; launch RabbitMQ as a Debian admin. No problem
{% highlight shell %}
sudo apt-get install rabbitmq-server
/etc/init.d/rabbimq-server start
{% endhighlight %}

Then I wanted to test the clusted mode (2 nodes server) running on the same host.
I have to reconfigure env variables, ... and unfortunatly after 1 hour I gave up. My node1 / node2 don't start, and it seems the configuration still try to start the default node installed by root.

I decided to enable verbose trace, which I did not find... So let's recompile it and see.


Building RabbitMQ like this ? ...

{% highlight shell %}
sudo apt-get install build-essential libxslt-dev xsltproc erlang-dev erlang-nox erlang-src xmlto
wget http://www.rabbitmq.com/releases/rabbitmq-server/v2.8.1/rabbitmq-server-2.8.1.tar.gz
tar zxvf rabbitmq-server-2.8.1.tar.gz
cd rabbitmq-server-2.8.1
make
sudo TARGET_DIR=/opt/rabbitmq-server \
SBIN_DIR=/opt/rabbitmq-server/sbin \
MAN_DIR=/opt/rabbitmq-server/man \
make install
{% endhighlight %}

Not sure it fully works... And I prefer building from github source

<H2>Step 1/ building RabbitMQ</H2>

{% highlight shell %}
$ git clone https://github.com/rabbitmq/rabbitmq-public-umbrella
$ cd rabbitmq-public-umbrella
$ make co
$ make
{% endhighlight %}

error ... "elixir" not found, and nothing in synaptic / apt-get ?
 
<H2>Step => 1/a/ need to build (or download) Elixir</H2>

See <A href="http://elixir-lang.org/">http://elixir-lang.org/</A>

{% highlight shell %}
$ git clone --depth 1 https://github.com/elixir-lang/elixir.git
$ elixir
$ make

...
At least Erlang 18.0 is required to build Elixir
Makefile:62: recipe for target 'lib/elixir/src/elixir.app.src' failed
{% endhighlight %}

<H2>Step 1/a/i need to build Erlang ...</H2>

cf doc: https://github.com/erlang/otp/blob/maint/HOWTO/INSTALL.md

{% highlight shell %}
$ git clone --depth 1 https://github.com/erlang/otp
$ cd otp
$ ./otp_build autoconf
$ ./configure --prefix=/opt/erlang/LATEST
$ make
{% endhighlight %}

Installing binaries to /opt/erlang/LATEST:
{% highlight text %}
sudo mkdir -p /opt/erlang/LATEST
sudo make install
....
make[1]: Leaving directory '.../erlang/otp/lib'
(cd "/opt/erlang/LATEST/lib/erlang" \
 && ./Install  -minimal "/opt/erlang/LATEST/lib/erlang")
..
ln -s ../lib/erlang/bin/erl erl
..
{% endhighlight %}

Testing
{% highlight shell %}
$ export PATH=/opt/erlang/LATEST/bin:$PATH
$ export LD_LIBRARY_PATH=/opt/erlang/LATEST/lib:$LD_LIBRARY_PATH

$ erl
Erlang/OTP 19 [erts-8.2.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V8.2.2  (abort with ^G)
1> 
{% endhighlight %}



<H2>return to Step 1/a/ need to build Elixir</H2>

{% highlight shell %}
cd elixir
$ make
{% endhighlight %}

{% highlight text %}
==> elixir (compile)
Compiled src/elixir_parser.yrl
...
==> iex (compile)
Generated iex app
{% endhighlight %}


Installing binaries
{% highlight shell %}
$ sudo mkdir -p /opt/erlang/elixir
$ export DESTDIR=/opt/erlang/elixir
$ export DESTDIR=
$ sudo make install
{% endhighlight %}

Testing elixir:
{% highlight shell %}
$ ls -l /opt/erlang/elixir/bin
total 0
 elixir -> ../lib/elixir/bin/elixir
 elixirc -> ../lib/elixir/bin/elixirc
 iex -> ../lib/elixir/bin/iex
 mix -> ../lib/elixir/bin/mix

$ export PATH=/opt/erlang/elixir/bin:$PATH
$ export LD_LIBRARY_PATH=/opt/erlang/elixir/lib:$LD_LIBRARY_PATH

$ which iex
/opt/erlang/elixir/bin/iex

$ iex
Erlang/OTP 19 [erts-8.2.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Interactive Elixir (1.5.0-dev) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> 
{% endhighlight %}



<H2>=> return to Step 1/ building RabbitMQ</H2>

{% highlight shell %}
$ cd rabbitmq-public-umbrella
$ make co
$ make 
{% endhighlight %}

Testing RabbitMQ

{% highlight shell %}
$ cd deps/rabbit
$ make run-broker 
...
ERROR: node with name "rabbit" already running on "localhost"

$ sudo /etc/init.d/rabbitmq-server stop
$ make run-broker
...
  rabbitmq-public-umbrella/deps/rabbit/scripts/rabbitmq-server
Erlang/OTP 19 [erts-8.2.2] [source] [64-bit] [smp:4:4] [async-threads:64] [hipe] [kernel-poll:true]

Eshell V8.2.2  (abort with ^G)
(rabbit@arn)1> 
  ##  ##
  ##  ##      RabbitMQ 3.7.0.milestone11+3.g60fe60d. Copyright (C) 2007-2016 Pivotal Software, Inc.
  ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
  ######  ##
  ##########  Logs: /tmp/rabbitmq-test-instances/rabbit/log/rabbit.log
                    /tmp/rabbitmq-test-instances/rabbit/log/rabbit_upgrade.log

              Starting broker...
 completed with 0 plugins.
{% endhighlight %}

Looks cool!


<H2>Clustering RabbitMQ ...</H2>

<A href="https://www.rabbitmq.com/clustering.html">https://www.rabbitmq.com/clustering.html</A>
<BR/>


I would like to have 1 (development) shared installation dir, and 2 nodes dir
<BR/>

Changing only tcp port and trying to start another node => error eaddr already in use
<BR/>
 
Here is the list of opened ports for a single node:

{% highlight shell %}
$ netstat -nlapute | grep 30091/
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:25672           0.0.0.0:*               LISTEN      1000       137903      30091/beam.smp  
tcp        0      0 127.0.0.1:46703         127.0.0.1:4369          ESTABLISHED 1000       137905      30091/beam.smp  
tcp6       0      0 :::35672                :::*                    LISTEN      1000       139288      30091/beam.smp  
{% endhighlight %}


Here are the 2 rabbitmq.config files I have edited  (there was a conflict on ports..)  
 
{% highlight text %}
%% node1.config
[
	{kernel, [
	    {inet_dist_listen_min, 15682},  %% default: RABBITMQ_NODE_PORT (AMQP port) + 20000
	    {inet_dist_listen_max, 15682}
	  ]},
    {rabbit, [
		{tcp_listeners, [15672]}
		]
    }
].
{% endhighlight %}


{% highlight text %}
%% node2.config
[
	{kernel, [
	    {inet_dist_listen_min, 15782},  %% default: RABBITMQ_NODE_PORT (AMQP port) + 20000
	    {inet_dist_listen_max, 15782}
	  ]},
    {rabbit, [
		{tcp_listeners, [15772]}
		]
    }
].
{% endhighlight %}
 

I edited a file node1-ctl.sh and node2-ctl.sh for each node

{% highlight shell %}
# node1-ctl.sh
. ./setenv-node1.sh
$HOME/rabbitmq-public-umbrella/deps/rabbit/scripts/rabbitmqctl "$@"
{% endhighlight  %}
 
{% highlight shell %}
$ ./node1-ctl.sh status
...
Status of node node1@arn ...
[
{pid,4300}
,{running_applications,[{rabbit,"RabbitMQ","3.7.0.milestone11+3.g60fe60d"},
...
,{kernel,{net_ticktime,60}}
]

$ ./node1-ctl.sh cluster_status
Cluster status of node node1@arn ...
[
{nodes,[{disc,[node1@arn]}]}
,{running_nodes,[node1@arn]}
,{cluster_name,<<"node1@arn.home">>}
,{partitions,[]}
,{alarms,[{node1@arn,[]}]}
]

$ ./node1-ctl.sh stop_app
Stopping rabbit application on node node1@arn ...

$ ./node1-ctl.sh join_cluster node2@arn
Clustering node node1@arn with node2@arn

$  ./node1-ctl.sh cluster_status
Cluster status of node node1@arn ...
[
{nodes,[{disc,[node1@arn,node2@arn]}]}
,{alarms,[{node2@arn,[]}]}
]

$ ./node1-ctl.sh start_app
Starting node node1@arn ...
completed with 0 plugins.

$ ./node1-ctl.sh cluster_status
Cluster status of node node1@arn ...
[
{nodes,[{disc,[node1@arn,node2@arn]}]}
,{running_nodes,[node2@arn,node1@arn]}
,{cluster_name,<<"node1@arn.home">>}
,{partitions,[]}
,{alarms,[{node2@arn,[]},{node1@arn,[]}]}
]

$ ./node1-ctl.sh set_policy ha-all "^ha\." '{"ha-mode":"all"}' 
Setting policy "ha-all" for pattern "^ha\." to "{"ha-mode":"all"}" with priority "0" for vhost "/" ...


# Checking clust_status on other node: node2
$ ./node2-ctl.sh cluster_status
Cluster status of node node2@arn ...
[
{nodes,[{disc,[node1@arn,node2@arn]}]}
,{running_nodes,[node1@arn,node2@arn]}
,{cluster_name,<<"node1@arn.home">>}
,{partitions,[]}
,{alarms,[{node1@arn,[]},{node2@arn,[]}]}
]

{% endhighlight %}


Testing with a (java springboot) client to send messages and listen messages... then kill node1, restart node1, kill node2, restart node2 ....

I get error in sendMessage() : "Auto recovery connection is not currently open"

{% highlight text %}
2017-02-09 23:26:57.746 ERROR 6438 --- [127.0.0.1:15672] c.r.c.impl.ForgivingExceptionHandler     : Caught an exception when recovering topology Caught an exception while recovering binding between spring-boot-exchange and spring-boot: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset

com.rabbitmq.client.TopologyRecoveryException: Caught an exception while recovering binding between spring-boot-exchange and spring-boot: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverBindings(AutorecoveringConnection.java:645) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverEntities(AutorecoveringConnection.java:584) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.beginAutomaticRecovery(AutorecoveringConnection.java:506) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.access$000(AutorecoveringConnection.java:53) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection$1.recoveryCanBegin(AutorecoveringConnection.java:435) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.notifyRecoveryCanBeginListeners(AMQConnection.java:693) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.doFinalShutdown(AMQConnection.java:687) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:577) [amqp-client-4.0.1.jar:4.0.1]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]
{% endhighlight %}
	

To make the sending more Robust (springboot does not auto-retry on sending??), I added explicitely a retry loop
	
{% highlight java %}
    	for (int i = 0; i < 100; i++) {
	        System.out.println("Sending message[" + i + "]...");
	        
	        for(int retryCount = 0; ; retryCount++) {
		        try {
		        	rabbitTemplate.convertAndSend(Application.queueName, "Hello from RabbitMQ[" + i + "]");
		        	if (retryCount > 0) {
		        		System.out.println("#### OK Sent message[" + i + "]   (after " + retryCount + " retries)");
		        	}
		        	break;
		        } catch(Exception ex) {
		        	System.out.println("Failed to send message[" + i + "]  .. retry!! (ex was " + ex.getMessage() + ")");
		        	Thread.sleep(1000);
		        	continue;
		        }
	        }
	        
	        receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    	
	        Thread.sleep(3000);
    	}
{% endhighlight %}


I expect all messages that are sent succesfully to be consumed later ...
Unfortunatly, it is NOT the case. There are missing messages, never received? !! 

{% highlight text %}

Sending message[20]...
Received <Hello from RabbitMQ[20]>
Sending message[21]...
Received <Hello from RabbitMQ[21]>
Sending message[22]...
Sending message[23]...
Sending message[24]...
2017-02-10 00:23:01.726  WARN 20642 --- [127.0.0.1:15772] c.r.c.impl.ForgivingExceptionHandler     : An unexpected connection driver error occured (Exception message: Connection reset)
2017-02-10 00:23:01.731 ERROR 20642 --- [127.0.0.1:15772] o.s.a.r.c.CachingConnectionFactory       : Channel shutdown: connection error
2017-02-10 00:23:01.731 ERROR 20642 --- [127.0.0.1:15772] o.s.a.r.c.CachingConnectionFactory       : Channel shutdown: connection error
Sending message[25]...
Failed to send message[25]  .. retry!! (ex was Auto recovery connection is not currently open)
2017-02-10 00:23:02.182  WARN 20642 --- [    container-1] o.s.a.r.l.SimpleMessageListenerContainer : Consumer raised exception, processing can restart if the connection factory supports it

com.rabbitmq.client.ShutdownSignalException: connection error
	at com.rabbitmq.client.impl.AMQConnection.startShutdown(AMQConnection.java:861) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.shutdown(AMQConnection.java:851) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.handleFailure(AMQConnection.java:674) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.access$400(AMQConnection.java:47) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:575) ~[amqp-client-4.0.1.jar:4.0.1]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]
Caused by: java.net.SocketException: Connection reset
	at java.net.SocketInputStream.read(SocketInputStream.java:189) ~[na:1.8.0]
	at java.net.SocketInputStream.read(SocketInputStream.java:121) ~[na:1.8.0]
	at java.io.BufferedInputStream.fill(BufferedInputStream.java:246) ~[na:1.8.0]
	at java.io.BufferedInputStream.read(BufferedInputStream.java:265) ~[na:1.8.0]
	at java.io.DataInputStream.readUnsignedByte(DataInputStream.java:288) ~[na:1.8.0]
	at com.rabbitmq.client.impl.Frame.readFrom(Frame.java:91) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.SocketFrameHandler.readFrame(SocketFrameHandler.java:164) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:571) ~[amqp-client-4.0.1.jar:4.0.1]
	... 1 common frames omitted

2017-02-10 00:23:02.184  INFO 20642 --- [    container-1] o.s.a.r.l.SimpleMessageListenerContainer : Restarting Consumer@21719a0: tags=[{amq.ctag-HKMJICvgBpjioX2BmcDytA=ha.spring-boot}], channel=Cached Rabbit Channel: AMQChannel(amqp://guest@127.0.0.1:15772/,1), conn: Proxy@55ad6766 Shared Rabbit Connection: SimpleConnection@7bfe7c5d [delegate=amqp://guest@127.0.0.1:15772/, localPort= 50290], acknowledgeMode=AUTO local queue size=0
2017-02-10 00:23:02.188 ERROR 20642 --- [    container-2] o.s.a.r.l.SimpleMessageListenerContainer : Failed to check/redeclare auto-delete queue(s).

org.springframework.amqp.rabbit.connection.AutoRecoverConnectionNotCurrentlyOpenException: Auto recovery connection is not currently open
	at org.springframework.amqp.rabbit.connection.SimpleConnection.isOpen(SimpleConnection.java:95) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory$ChannelCachingConnectionProxy.isOpen(CachingConnectionFactory.java:1151) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory.getChannel(CachingConnectionFactory.java:420) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory.access$1500(CachingConnectionFactory.java:97) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory$ChannelCachingConnectionProxy.createChannel(CachingConnectionFactory.java:1084) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.core.RabbitTemplate.doExecute(RabbitTemplate.java:1435) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.core.RabbitTemplate.execute(RabbitTemplate.java:1411) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.core.RabbitTemplate.execute(RabbitTemplate.java:1387) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.core.RabbitAdmin.getQueueProperties(RabbitAdmin.java:336) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.redeclareElementsIfNecessary(SimpleMessageListenerContainer.java:1114) [spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.access$1100(SimpleMessageListenerContainer.java:95) [spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:1333) [spring-rabbit-1.7.0.RELEASE.jar:na]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]

Failed to send message[25]  .. retry!! (ex was Auto recovery connection is not currently open)
Failed to send message[25]  .. retry!! (ex was Auto recovery connection is not currently open)
Failed to send message[25]  .. retry!! (ex was Auto recovery connection is not currently open)
Failed to send message[25]  .. retry!! (ex was Auto recovery connection is not currently open)
2017-02-10 00:23:06.751 ERROR 20642 --- [127.0.0.1:15772] c.r.c.impl.ForgivingExceptionHandler     : Caught an exception when recovering topology Caught an exception while recovering exchange ha.spring-boot-exchange: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset

com.rabbitmq.client.TopologyRecoveryException: Caught an exception while recovering exchange ha.spring-boot-exchange: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverExchanges(AutorecoveringConnection.java:597) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverEntities(AutorecoveringConnection.java:582) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.beginAutomaticRecovery(AutorecoveringConnection.java:506) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.access$000(AutorecoveringConnection.java:53) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection$1.recoveryCanBegin(AutorecoveringConnection.java:435) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.notifyRecoveryCanBeginListeners(AMQConnection.java:693) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.doFinalShutdown(AMQConnection.java:687) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:577) [amqp-client-4.0.1.jar:4.0.1]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]
Caused by: com.rabbitmq.client.AlreadyClosedException: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.AMQChannel.ensureIsOpen(AMQChannel.java:198) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.rpc(AMQChannel.java:244) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.privateRpc(AMQChannel.java:222) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:117) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.exchangeDeclare(ChannelN.java:763) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.exchangeDeclare(ChannelN.java:705) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.exchangeDeclare(ChannelN.java:50) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.RecordedExchange.recover(RecordedExchange.java:35) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverExchanges(AutorecoveringConnection.java:593) [amqp-client-4.0.1.jar:4.0.1]
	... 8 common frames omitted

2017-02-10 00:23:06.752 ERROR 20642 --- [127.0.0.1:15772] c.r.c.impl.ForgivingExceptionHandler     : Caught an exception when recovering topology Caught an exception while recovering queue ha.spring-boot: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset

com.rabbitmq.client.TopologyRecoveryException: Caught an exception while recovering queue ha.spring-boot: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverQueues(AutorecoveringConnection.java:632) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverEntities(AutorecoveringConnection.java:583) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.beginAutomaticRecovery(AutorecoveringConnection.java:506) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.access$000(AutorecoveringConnection.java:53) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection$1.recoveryCanBegin(AutorecoveringConnection.java:435) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.notifyRecoveryCanBeginListeners(AMQConnection.java:693) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.doFinalShutdown(AMQConnection.java:687) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:577) [amqp-client-4.0.1.jar:4.0.1]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]
Caused by: com.rabbitmq.client.AlreadyClosedException: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.AMQChannel.ensureIsOpen(AMQChannel.java:198) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.rpc(AMQChannel.java:244) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.privateRpc(AMQChannel.java:222) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:117) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.queueDeclare(ChannelN.java:948) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.queueDeclare(ChannelN.java:50) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.RecordedQueue.recover(RecordedQueue.java:53) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverQueues(AutorecoveringConnection.java:608) [amqp-client-4.0.1.jar:4.0.1]
	... 8 common frames omitted

2017-02-10 00:23:06.754 ERROR 20642 --- [127.0.0.1:15772] c.r.c.impl.ForgivingExceptionHandler     : Caught an exception when recovering topology Caught an exception while recovering binding between ha.spring-boot-exchange and ha.spring-boot: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset

com.rabbitmq.client.TopologyRecoveryException: Caught an exception while recovering binding between ha.spring-boot-exchange and ha.spring-boot: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverBindings(AutorecoveringConnection.java:645) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverEntities(AutorecoveringConnection.java:584) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.beginAutomaticRecovery(AutorecoveringConnection.java:506) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.access$000(AutorecoveringConnection.java:53) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection$1.recoveryCanBegin(AutorecoveringConnection.java:435) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.notifyRecoveryCanBeginListeners(AMQConnection.java:693) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection.doFinalShutdown(AMQConnection.java:687) [amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:577) [amqp-client-4.0.1.jar:4.0.1]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]
Caused by: com.rabbitmq.client.AlreadyClosedException: connection is already closed due to connection error; cause: java.net.SocketException: Connection reset
	at com.rabbitmq.client.impl.AMQChannel.ensureIsOpen(AMQChannel.java:198) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.rpc(AMQChannel.java:244) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.privateRpc(AMQChannel.java:222) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:117) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.queueBind(ChannelN.java:1057) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.ChannelN.queueBind(ChannelN.java:50) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.RecordedQueueBinding.recover(RecordedQueueBinding.java:30) ~[amqp-client-4.0.1.jar:4.0.1]
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.recoverBindings(AutorecoveringConnection.java:641) [amqp-client-4.0.1.jar:4.0.1]
	... 8 common frames omitted

#### OK Sent message[25]   (after 5 retries)
2017-02-10 00:23:07.191  WARN 20642 --- [    container-2] o.s.a.r.l.SimpleMessageListenerContainer : Consumer raised exception, processing can restart if the connection factory supports it

org.springframework.amqp.rabbit.connection.AutoRecoverConnectionNotCurrentlyOpenException: Auto recovery connection is not currently open
	at org.springframework.amqp.rabbit.connection.SimpleConnection.isOpen(SimpleConnection.java:95) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory$ChannelCachingConnectionProxy.isOpen(CachingConnectionFactory.java:1151) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory.getChannel(CachingConnectionFactory.java:420) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory.access$1500(CachingConnectionFactory.java:97) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory$ChannelCachingConnectionProxy.createChannel(CachingConnectionFactory.java:1084) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.ConnectionFactoryUtils$1.createChannel(ConnectionFactoryUtils.java:95) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.ConnectionFactoryUtils.doGetTransactionalResourceHolder(ConnectionFactoryUtils.java:144) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.connection.ConnectionFactoryUtils.getTransactionalResourceHolder(ConnectionFactoryUtils.java:76) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.start(BlockingQueueConsumer.java:505) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:1335) ~[spring-rabbit-1.7.0.RELEASE.jar:na]
	at java.lang.Thread.run(Thread.java:744) [na:1.8.0]

2017-02-10 00:23:07.192  INFO 20642 --- [    container-2] o.s.a.r.l.SimpleMessageListenerContainer : Restarting Consumer@4c66db5: tags=[{}], channel=null, acknowledgeMode=AUTO local queue size=0
Received <Hello from RabbitMQ[25]>
Sending message[26]...
Received <Hello from RabbitMQ[26]>
Sending message[27]...
Received <Hello from RabbitMQ[27]>
Sending message[28]...
Received <Hello from RabbitMQ[28]>
{% endhighlight %}



Where is the Problem ?
<BR/>
Maybe the queue in mode "auto-delete" is not the correct settings: the queue mmay be deleted with pending messages while lossing connection, then recreated just after with message lost.
<BR/>
My test-case is special because my consumer and publisher are shring the same channel/connection (in same java process)!
<BR/> 
Maybe the consumer with "acknowledgeMode=AUTO" is not the good consumer settings, because the connection may be lost during the consume ??

NO Idea yet ... investigating ...


 


<H2>Conclusion</H2>

OK to compile Elixir+Erlang+RabbitMQ all from github
<BR/>
Connecting a AMQP client, is easy.
<BR/>
Ensuring Clustering + High Availability really works is not trivial !!! By default, it does not work !!!
<BR/> 



