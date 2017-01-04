---
layout: post
title:  "Introduction to Apache Cassandra"
date:   2017-01-04 20:14:00
categories: 
tags: cassandra nosql bigdata
---

<H1>What is Apache Cassandra ?</H1>

Cassandra is a distributed NoSql database, similar to Google BigTable or HBase.
<br/>
It is hybrid between Key-Value and Column oriented database.
<br/>
There are client drivers for all langages (internal protocol is binary "Thrift").
<br/>
There is also a langage CQL very similar to SQL, with an interactive shell to execute commands: CQLSH
<br/>

Some links:

<ul>
<li>Google: <A href="https://www.google.fr/search?q=cassandra">Google Cassandra</A> ... over 105 Millions results
</li>
<li>Wikipedia: 
<A href="https://en.wikipedia.org/wiki/Apache_Cassandra">https://en.wikipedia.org/wiki/Apache_Cassandra</A>
</li>
<li>Home page: <A href="http://cassandra.apache.org/">http://cassandra.apache.org/</A>
</li>
<li>Doc: <A href="http://cassandra.apache.org/doc/latest">Cassandra Doc</A>
</li>
</ul>




<H1>First steps using Apache Cassandra (3 minutes)</H1>


Download link:
<A href="http://www.apache.org/dyn/closer.lua/cassandra/3.9/apache-cassandra-3.9-bin.tar.gz">apache-cassandra-3.9-bin.tar.gz</A>
<br/>

Unzip, then start it:
{% highlight text %}
apache-cassandra-3.9$ ./bin/cassandra -f
...
...
INFO  19:07:18 Starting listening for CQL clients on localhost/127.0.0.1:9042 (unencrypted)...
INFO  19:07:18 Not starting RPC server as requested. Use JMX (StorageService->startRPCServer()) or nodetool (enablethrift) to start it
INFO  19:07:27 Scheduling approximate time-check task with a precision of 10 milliseconds
INFO  19:07:28 Created default superuser role 'cassandra'
{% endhighlight %}


Starting CQLSH shell command client:
{% highlight text %}
$ bin/cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.8 | CQL spec 3.4.2 | Native protocol v4]
Use HELP for help.
{% endhighlight %}

First command to check server topology (=single node)
{% highlight text %}
cqlsh> SELECT cluster_name, listen_address FROM system.local;

 cluster_name | listen_address
--------------+----------------
 Test Cluster |      127.0.0.1

(1 rows)
cqlsh>
{% endhighlight %}


First commands to create Keyspace, then Table, then Data..

<A href="http://cassandra.apache.org/doc/latest/cql/ddl.html">Cql DDL Doc</A>

{% highlight text %}
cqlsh> CREATE KEYSPACE myks WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 1};
cqlsh> 
cqlsh> USE myks;
cqlsh:myks> 
{% endhighlight %}


Create table:
{% highlight text %}
CREATE TABLE t1 (
    f1 int,
    f2 int,
    f3 text,
    f4 text,
    PRIMARY KEY (f1, f2)
);
{% endhighlight %}

CRUD DML commands:
{% highlight text %}
INSERT INTO t1 (f1,f2,f3,f4) VALUES (0, 0, 'value0-0:f3', 'value0-0:f4');
INSERT INTO t1 (f1,f2,f3,f4) VALUES (0, 1, 'value0-1:f3', 'value0-1:f4');

SELECT * FROM t1;
{% endhighlight %}

It just works !

{% highlight text %}
 f1 | f2 | f3          | f4
----+----+-------------+-------------
  0 |  0 | value0-0:f3 | value0-0:f4
  0 |  1 | value0-1:f3 | value0-1:f4

(2 rows)
{% endhighlight %}


