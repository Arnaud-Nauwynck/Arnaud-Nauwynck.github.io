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




<H1>First steps starting a single Cassandra Server (2 minutes)</H1>

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


<H1>First steps using Cassandra CQL Shell (5 minutes)</H1>

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


Doing select with criteria
{% highlight text %}
select * from t1 where f1 = 0 and f2 = 1;

 f1 | f2 | f3          | f4
----+----+-------------+-------------
  0 |  1 | value0-1:f3 | value0-1:f4

(1 rows)
{% endhighlight %}


Trying to do a select with un-indexed criteria ... like "Full Table Scan" in sql:
{% highlight text %}
cqlsh:myks> select * from t1 where f2 = 1;
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"
{% endhighlight %}

OK, let be explicitely doing wrong things like sql:
{% highlight text %}
cqlsh:myks> select * from t1 where f2 = 1  ALLOW FILTERING;

 f1 | f2 | f3          | f4
----+----+-------------+-------------
  0 |  1 | value0-1:f3 | value0-1:f4

(1 rows)
{% endhighlight %}


Using secondary indexes:

{% highlight text %}
cqlsh:myks> CREATE INDEX ON t1 (f3);

cqlsh:myks> select * from t1 where f3 = 'value0-0:f3';

 f1 | f2 | f3          | f4
----+----+-------------+-------------
  0 |  0 | value0-0:f3 | value0-0:f4

(1 rows)
{% endhighlight %}

Update 1 row using complete pk:
{% highlight text %}
update t1 set f3 = 'f3' where f1=0 and f2=0;
{% endhighlight %}

Try update rows using partial pk:
{% highlight text %}
cqlsh:myks> update t1 set f3 = 'f3' where f1=0;
InvalidRequest: Error from server: code=2200 [Invalid query] message="Some clustering keys are missing: f2"
{% endhighlight %}

Try update rows using non pk criteria:
{% highlight text %}
cqlsh:myks> update t1 set f3 = 'f3' where f3 = 'value0-0:f3';
InvalidRequest: Error from server: code=2200 [Invalid query] message="Some partition key parts are missing: f1"
{% endhighlight %}


Preliminary conclusions on Cassandra & CQL : it looks really simple, really like SQL, but with "natural" limitations from a Key-Value database.



<H1>Using Java client (10 minutes)</H1>

I tryed to use the all-in-one springboot-data-cassandra, but as you can see later , it is in a broken state for Cassandra 3!!
<BR/>
Let's start with a simple Cassandra driver connection.

Create a maven project, and edit your pom.xml to add 
{% highlight text %}
  <dependencies>
  	<dependency>
		<groupId>com.datastax.cassandra</groupId>
		<artifactId>cassandra-driver-core</artifactId>
		<version>3.1.3</version>
	</dependency>
  </dependencies>
{% endhighlight %}

Type "mvn install", and import in eclipse.
<BR/>

First write java code  to connect to Cassandra server:
{% highlight java %}
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

	// Connect to the cluster and keyspace "myks"
	Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
	Session session = cluster.connect("myks");
	System.out.println("connected to Cassandra localhost");
{% endhighlight %}


Then execute a query
{% highlight java %}
import com.datastax.driver.core.ResultSet;
import com.datastax.driver.core.Row;

	// execute some queries...
	ResultSet results = session.execute("SELECT * FROM T1 WHERE f1=0");
	for (Row row : results) {
		int f1 = row.getInt("f1"); 
		int f2 = row.getInt("f2"); 
		String f3 = row.getString("f3"); 
		String f4 = row.getString("f4"); 
		System.out.println("row f1:" + f1 + ", f2:" + f2 + ", f3:'" + f3 + "', f4:'" + f4 + "'");
	}
{% endhighlight %}
It really looks like jdbc API, but simply replace java.sql.ResultSet by com.datastax.driver.core.ResultSet ...
<BR/>

Run it ... it just works!
{% highlight text %}
connected to Cassandra localhost
row f1:0, f2:0, f3:'f3', f4:'value0-0:f4'
row f1:0, f2:1, f3:'value0-1:f3', f4:'value0-1:f4'
{% endhighlight %}



<H1>Using springboot-data-cassandra with cassandra (~ 2 hours trial solving errors)</H1>

Springboot is an amazing project, and springboot-data is also an amazing sub-project for simplifying java code for acessing repositories of data (not only jpa hibernate, but also key-values db, neo4j db, ...).
<br/>

To test a Java client using springboot-data + Cassandra, go to
<A href="http://start.spring.io/">http://start.spring.io/</A>
<br/>
check option "Cassandra", then click on "download" button
<br/>
Unzip, and type "mvn install", then in Eclipse "import as maven project".
<BR/>
the project is only a maven skeleton, it does not contains code.
If you ar einterrested, you can search for "google springdata cassandra sample", or also read the doc
<A href="http://docs.spring.io/autorepo/docs/spring-data-cassandra/current/reference/html/">http://docs.spring.io/autorepo/docs/spring-data-cassandra/current/reference/html/</A>

I have then coded my class with @Table, @PrimaryKey, my primary key class with @PrimaryKeyClass and @PrimaryKeyColumn(..) columns and my interface Repository ...

{% highlight java %}
@PrimaryKeyClass
public class T1Key implements Serializable {

	/** */
	private static final long serialVersionUID = 1L;
	
	@PrimaryKeyColumn(ordinal = 0, type = PrimaryKeyType.PARTITIONED)
	private final int f1;

	@PrimaryKeyColumn(ordinal = 1, type = PrimaryKeyType.PARTITIONED)
	private final int f2;

	... 
	code ommitted : ctor + getters for f1, f2 + equals/hashCode
}


@Table
public class T1 {

	@PrimaryKey
	private T1Key pk;
	
	private String f3;
	private String f4;
	
	// ------------------------------------------------------------------------
	
	// silly constructor required by spring-data-cassandra WHY WHY?!! 
	// ... otherwise you have error "Property [pk] has no single column mapping"
	public T1() {
	}
	
	public T1(T1Key pk) {
		this.pk = pk;
	}

	// ------------------------------------------------------------------------

	// setter not needed on pk from spring-data-cassandra!
	public T1Key getPK() {
		return pk;
	}
	
	// dummy destructured getters getF1() and getF2() for composite key
	public int getF1() {
		return pk.getF1();
	}
	public int getF2() {
		return pk.getF2();
	}

	...
	code ommitted: regular getter/setters for field f3, f4
}


public interface T1Repository extends TypedIdCassandraRepository<T1,T1Key> {

	@Query("select * from T1 where f1=?0")
	public List<T1> findByF1(int f1);

}


@SpringBootApplication
public class TestCassandraApplication implements CommandLineRunner {

	@Autowired
	private T1Repository repository;

	@Value("${spring.data.cassandra.keyspace-name}")
	private String cassandraKeyspace;
	
	public static void main(String[] args) {
		SpringApplication.run(TestCassandraApplication.class, args);
	}

	@Override
	public void run(String... args) throws Exception {
		System.out.println("using Cassandra keyspace:" + cassandraKeyspace);
		
		long countAll = repository.count();
		System.out.println("count -> " + countAll + " elt(s)");
		Iterable<T1> all= repository.findAll();
		System.out.println("findAll -> [0]:" + all.iterator().next() + " ...");
		
		T1 res00 = repository.findOne(new T1Key(0, 0));
		System.out.println("find pk:(0,0) -> " + res00);
		
		List<T1> resLs = repository.findByF1(0);
		System.out.println("find by f1=0 -> " + resLs.size() + " elt(s)");
		
		int maxId = resLs.stream().mapToInt(x -> x.getPK().getF1()).max().getAsInt();
		T1Key newPK = new T1Key(maxId+1, 0);
		T1 newT1 = new T1(newPK);
		newT1.setF3("test from springboot-data");
		repository.save(newT1);
	}
}
{% endhighlight %}

The code with spring-data is very elegant. (Could be even better with lombok for getter/setter removal).



Unfortunatly, the first time I launched the client, I got this error ! ... I forgot to change the schema keyspace ?
{% highlight text %}
Caused by: com.datastax.driver.core.exceptions.NoHostAvailableException: All host(s) tried for query failed (tried: localhost/127.0.0.1:9042 (com.datastax.driver.core.exceptions.InvalidQueryException: unconfigured table schema_keyspaces))
	at com.datastax.driver.core.ControlConnection.reconnectInternal(ControlConnection.java:240) ~[cassandra-driver-core-2.1.9.jar:na]
{% endhighlight %}

The "magic" initialisation comes from class org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration.
<BR/>
It uses properties from org.springframework.boot.autoconfigure.cassandra.CassandraProperties
<BR/>

So I have added this line in file application.properties
<BR/>
{% highlight text %}
spring.data.cassandra.keyspace-name: myks
{% endhighlight %}
 ... and re-executed "mvn install" !! (because eclipse does NOT update resources files from src/main/resources to target/classes)
<br/>

But apparently, another problem comes from a mismatch between client (v2) and server (v3):
<A href="http://stackoverflow.com/questions/35532569/exception-while-running-spring-boot-cassandra-project">Stackoverflow cassandra unconfigured table schema_keyspaces</A>

From maven pom.xml dependency (mvn dependency:list, then mvn help:effective-pom), I discovered that the version used was cassandra-driver-core:2.1.9

I looked for an upgraded version of cassandra-driver-core.. using <A href="
http://search.maven.org/#search|ga|1|a%3Acassandra-driver-core">http://search.maven.org  a:cassandra-driver-core</A>
I peeked up latest version 3.1.3
{% highlight text %}
<properties>
	<cassandra-driver.version>3.1.3</cassandra-driver.version>
</properties>
{% endhighlight %}

Unfortunatly, this version does not compile, because cassadra-driver-dse is not released yet on same version 3.1.3, and version 3.0.0-rc1 is not compatible...

I got errors like, for an incompatibility between spring-data-cassandra and cassandra-driver ...
{% highlight text %}
Caused by: java.lang.NoSuchMethodError: com.datastax.driver.core.DataType.asJavaClass()Ljava/lang/Class;
	at org.springframework.data.cassandra.mapping.CassandraSimpleTypeHolder.<clinit>(CassandraSimpleTypeHolder.java:62) ~[spring-data-cassandra-1.4.6.RELEASE.jar:na]
{% endhighlight %}
.. as described here :<A href="http://stackoverflow.com/questions/36558099/spring-data-cassandra-1-3-4-not-compatible-with-cassandra-3-x">StackOverflow spring-data-cassandra-1-3-4-not-compatible-with-cassandra-3</A>

I tryed using the latest spring-data-cassandra, directly from github source code
{% highlight text %}
git clone https://github.com/spring-projects/spring-data-cassandra.git
cd spring-data-cassandra
mvn clean install -DskipTests
{% endhighlight %}

even test do not compile! (so... rm -rf spring-data-cassandra/src/test/java/org)

I have also cloned the springboot-starter github project, and use the SNAPSHOT build..
{% highlight text %}
git clone https://github.com/spring-projects/spring-boot.git
cd spring-boot
mvn clean install -DskipTests
{% endhighlight %}

{% highlight text %}
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.0.BUILD-SNAPSHOT</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.data</groupId>
				<artifactId>spring-data-cassandra</artifactId>
				<version>1.5.0.BUILD-SNAPSHOT</version>
			</dependency>
		</dependencies>
	</dependencyManagement>
{% endhighlight %}


Notice also that when not putting a default empty constructor on my @Table class, I had a silly confusing error message
{% highlight text %}
Property [pk] has no single column mapping
{% endhighlight %}


Finally, I got some successful results!!

{% highlight text %}
using Cassandra keyspace:myks
count -> 2 elt(s)
findAll -> [0]:fr.an.tests.T1@65d8dff8 ...
find pk:(0,0) -> fr.an.tests.T1@4730e0f0
find by f1=0 -> 2 elt(s)
{% endhighlight %}

Conclusion: waiting springboot-data to publish an official release that work successfully with Cassandra version 3 !!
<BR/>
Cassandra, springboot & springboot-data are amazing projects.


