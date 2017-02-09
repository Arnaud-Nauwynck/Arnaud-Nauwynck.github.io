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


<H2>Conclusion</H2>

OK to compile Elixir+Erlang+RabbitMQ all from github
<BR/>
Next step ... connecting a AMQP client, and checking 



