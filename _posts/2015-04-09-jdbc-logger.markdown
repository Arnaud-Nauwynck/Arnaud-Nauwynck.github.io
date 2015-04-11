---
layout: post
title:  "Jdbc-Logger"
date:   2015-04-09 22:00:00
categories: github projects performance monitoring
tags: 
---


<h1>Jdbc Logger</h1>

Jdbc-Logger is one of my first tool. I have used it successfully over 10 years, on many projects... and solved hundred of sql performance problems.
Basically, it is a simple J2EE profiller, specialized on the jdbc API instrumentation only.

It did not exist in the last century (~1999), or I didn't know a freely available tool. Now, there are thousands: sp6, hibernate "show-sql=true", or non free versions like JProfiller, Dynatrace, Appdynamics, Performazure, Willy, JxInsight, etc. etc.
Basically what I needed was a simple java class acting as a proxy for "java.sql.PreparedStatement", to instrument and capture metrics counters, then display them. It is of course fully integrated with the whole API of Driver, DataSource, XADataSource, XAResource, Statement/CallableStatement/PreparedStatement, ResultSet...


{% highlight java %}
class LoggerPreparedStatement implements PreparedStatement {

	PreparedStatement delegateTo;
	
	..
	
	public void execute() {
		long startTime = System.currentTimeMillis();
		pre("execute");
		
		// do delegate to underlying "delegateTo"
		delegateTo.execute();
		
		long timeMillis = System.currentTimeMillis() - startTime;
		post("execute", timeMillis);
	}
	
}
{% endhighlight %}

internally, the "pre()" / "post()" methods acts as an instumetnation AOP aspect, where the post() method specifically log slow SQL queries, and increment an in-memory counter entry, for the sql key

What is captured:
<ul>
<li>PreparedStatement sql statement/query  (containing bind-variable placeholders). example: "select * from Employee e where e.id = ?" </li>
<li>number of times the query was executed. example: 123 times</li>
<li>cumulated time of query execution. example: 2460 ms...  meaning 123 * 20 millis in average</li>
<li>sample value for bound-variables for some of the executed queries: for the first time it was called, or last time, or better, for longest execution time. Example: variables=[12345]</li>
<li>more easy to use, I show directly the sql with inlined values: "select * from Employee e where e.id = 12345"</li> 
<li>corresponding java StackTrace when the sql was executed, generally as the first call only, because is involve performance overhead</li> 
<li>number of time the query is currently executing, and not finished yet ... shown in logs or shown as metric gauges</li>
<li>first execution timestamp, time execution, and bound variable</li>
<li>longuest execution timsetamp, time execution and bound variable</li>
</ul>

As a very important refinement, what was really captured was a cumulated stats entries per "similar queries". By similar queries, I mean many developpers tend to forget bind-variable are so important to performances, and they write "select * from employee e where id = 12345".
So I first re-templatized the sql, to guess the "id=?" instead of hard-coded "id=12345".  Finally, I tend to have only few hundreds sql templatized queries, even for very complex systems, when millions of queries are really executed. 

The instrumentation tool is insignifiant : less than 1% overhead in cpu (few nanos for an IO-bound program), few Kb in memory.

To analyse a sql capture, What I do is: 
<ul>
<li>export to a CSV file  (from JMX MBean management action)</li>
<li>import CSV as Excel file</li>
<li>compute cumulated importance of each sql templatized query (by decreasing order of importance)</li>
<li>hide to ignore irrelevant 10%, focusing on 90% of the cumulated time only</li>
<li>among the top 10 queries, determine why they are in the top 10 ... </li>
</ul>

As strange as it might be, the CSV extraction containing stack-trace plus few indicators is in my experience 99% of the time enough to determine the performance problems!
Basically, it is oftern very fast and easy to classify these performances points into one of these categories (... but then the correction is not so fast/easy in legacy code): 
 <ul>
   <li>slow queries because of bad execution plan (lack of indexes, or bad indexes in schema database / bad sql design)</li>
   <li>queries called to many times, in a N+1 fetching problem (usually a bad Object Hierarchical mapping of List relationship, too fine granularity in code, no level 2 caching, ...)</li>
   <li>queries fetching too many data (so needing a better fetch size, a better granularity, or better doing database aggregation instead server-side post-treatments)</li>
   <li>...</li>
</ul>
 
 
To use the jdbc-logger tool, I add the jar in the CLASSPATH and have to change the jdbc url and driver class in the configuration placeholder file.
{% highlight properties %}
# change:
# myapp.datasource.url=jdbc:oracle:blabla...
# myapp.datasource.driverClass=org.oracle.Driver
# by:
myapp.datasource.url=jdbc:log:oracle:blabla...
myapp.datasource.driverClass=fr.an.jdbclogger.DriverLog
{% endhighlight %}

Another possibility is to simply include a spring AOP aspect bean to wrap my datasource by an instrumented datasource

Recently I have re-implemented the jdbc-logger tool to make it more generic on metric counter, not as purely a sql jdbc metric counter.
This is described in the following second project called "sef4j"


