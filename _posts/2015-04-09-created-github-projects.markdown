---
layout: post
title:  "Created Github projects"
date:   2015-04-09 21:00:00
categories: github projects
tags: jekyll
---

I have finally decided to put some of my personnal projects ideas as open-source projects.

<ul>
<li>jdbc-logger  (Jdbc performance J2EE profiler)</li>
<li>sef4j : Simple Event Facade For Java  (java applicative instrumentation for profiler)</li>
<li>jxmvc : eXtensive MVC with field dependency propagation (Excel like dependency function, in a Transactional model code)</li>
<li>mda4j : Model Driven Architecture for Java (reflection framework for client-server DTO transfer with rich behavior)</li>
<li>cmdb4j : Configuration Management Database for Java (mainly for applicative parameters per environement, then deployment and Monitoring)</li>
<li>java code analysis, eclipse plugins tools</li>
<li>image and sound analysis</li>
<li>...other</li>
</ul>


Here is a short description of the aims and motivation behinds.

 
Basically, I found problems to solve at office. These are day to day developper problems and tools or framework I whish I could use if they existed.
Within the office context, these problems are sometimes too specific, less interresting that at home, because of proprietary and legacy code around, and lack of isolation. So I code them at home, with less constraints.

2 possibilities:
<ul>
<li> I start coding a specific program solution at work, and then think about re-implementing it at home, differently, trying to do it more elegantly and more generically. </li>
<li> Some other times, it helps me coding restfully at home first, and then reimporting a simpler more pragmatic solution at work, when all technical points are solved in my mind.</li>
</ul>

Unfortunatly, coding Open Source directly from my office is not legally possible... Things changes but I have to wait more.


Of course, problems at work always finishes as solution, or are replaced by other new problems, and the motivation disapeared in a solved-problem or forgotten-one!

Even after the motivation has disapeared in one of the projects, there remains something on my hard drive.
Putting these projects on github is first of all a way to give them to whoever wants to see them, use them, or improve them.

Here is a short description of these projects...


<h1>Jdbc Logger</h1>

Jdbc-Logger is one of my first tool. I have used it successfully over 10 years, on many projects... and solve hundred of sql performance problems.
Basically, it is a simple J2EE profiller, specialized on the jdbc API instrumetnation only.

It did not exists in the last century (~1999), or I didn't know a freely available tool. Now, there are thousands: sp6, hibernate "show-sql=true", or non free versions like JProfiller, Dynatrace, Appdynamics, Performazure, Willy, JxInsight, etc. etc.
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

What was captured was:
<ul>
<li>PreparedStatement sql statement/query  (containing bind-variable). example: "select * from employee e where id = ?" </li>
<li>number of times the query was executed. example: 123 times</li>
<li>cumulated time of query execution. example: 2460 ms...  meaning 123 * 20 millis in average</li>
<li>sample value for bound-variables for some of the executed queries: for the first time it was called, or last time, or better, for longest execution time</li>
<li>corresponding java StackTrace when the sql was executed, generally as the first call only, because is involve performance overhead</li> 
<li>number of time the query is currently executing, and not finished yet ... shown in logs or shown as metric gauges</li>
<li>first execution timestamp, time execution, and bound variable</li>
<il>longuest execution timsetamp, time execution and bound variable</li>
</ul>

As a very important refinement, what was really captured was a cumulated stats entries per "similar queries". By similar queries, I mean many developpers tend to forget bind-variable are so important to performances, and they write "select * from employee e where id = 12345".
So I first re-templatized the sql, to guess the "id=?" instead of hard-coded "id=?" (I use cursor-sharing anyway on database...). I also simplify selected columns, and similar parts:  "select name, firstname, telephone, whatever_field ..." is equivalent to templatized "select * ". Finally, I tend to have only few hundreds sql templatized queries, even for very complex systems, when millions of queries are really executed. In memory, the instrumentation tool is insignifiant : 1% overhead in cpu, few Kb in memory.

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
</ul>
 
 
To use the jdbc-logger tool, I usually have to change the jdbc url and driver class in the configuration placeholder file.
{% highlight properties %}
# change:
# myapp.datasource.url=jdbc:oracle:blabla...
# myapp.datasource.driverClass=org.oracle.Driver
# by:
myapp.datasource.url=jdbc:log:oracle:blabla...
myapp.datasource.driverClass=fr.an.jdbclogger.DriverLog
{% endhighlight %}

Another possibility is to simply include a spring AOP aspedct bean to wrap my datasource by an instrumented datasource

Recently I have re-implemented the jdbc-logger tool to make it more generic on metric counter, not as purely a sql jdbc metric counter.
This is described in the following second project called "sef4j"



<h1>Sef4j - Performance monitoring counters - metrics - events</h1>

Sef4j, is also a performance monitoring framework, which has now many free/commercial equivalent.
Open source "equivalent framework" would be Dropwizard Metrics (formely Codahale Metrics), JavaMelody, or many other...
Commercial equivalent are ultra impressives (and ultra expensive...).

Sef4j stands as initials for "Simple Event Facade for Java"
It tries to offer a simple API for Events, as simple as slf4j (previously Log4J) is to Log.

Another project name would "Performance Monitoring Facade 4 Java", because these low level events are mainly for performances counters, metrics, monitoring, logging and alerting.




The big differences with Sef4j compared to Dropwizard Metrics is that Sef4j captures applicative stack traces, not flat traces counters like Metrics.
But it also allows you to manually instrument applicative parameters that make sense in your business code, and use them to categorize differents types of counters for the same stack trace. For example in a financial application, you may want to have several counters for the same pricing method depending of the portfolio used, instruments, thirdparties... because different rules are used to dispatch to different algorithms and data repositories. 

Counters may be in-memory tree, that aggregate events with predefined or user-defined listener code. 
The framework for instrumenting follows open-close principles! ... If offers you events for push() and pop() for every instrumented methods, you are free to treat these events for aggregating what make sense, and producing counter, histograms, alerts, events, logs... 



A drawing might help understand these simple things:
Say you have 2 entry points Web services, internally calling several services, and then DAOs jdbc queries.

In Dropwizard Metrics, you would instrument it like that:
{% highlight java %}
public class MyWebService {
   @Timed
   public MyResult myService1(param1, param2...) {
      dao.myQuery1(..)
   }
   @Timed
   public MyResult myService2(param1, param2...) {
      dao.myQuery1(..)
      dao.myQuery2(..)      
   }
}

public class MyDAO {
   @Timed
   public MyResult myQuery1(param1, param2...) {
      jdbc / jpa query...
   }
   @Timed
   public MyResult myQuery2(param1, param2...) {
      jdbc / jpa query...
   }
}
{% endhighlight %}

In DropWizard Metrics, You will get 4 flat metric counters:
{% highlight text %}
MyWebService.myServices1  --> counter stats, ..
MyWebService.myServices2  --> counter stats, ..
MyDAO.myQuery1  --> counter stats, ..
MyDAO.myQuery2  --> counter stats, ..
{% endhighlight %}

How can you known how are linked performance problems in DAO.myQuery1 and MyWebService entrypoint 1/2 ?.. You can't.
In Sef4j as in many J2EE profiler tools, you will get a tree of performance counters

{% highlight text %}
/
+-- MyWebService.myServices1  --> counter stats, ..
|     |
|     +-- MyDAO.myQuery1  --> counter stats, ..
|     +-- MyDAO.myQuery2  --> counter stats, ..
|
+-- MyWebService.myServices2  --> counter stats, ..
      |
      +-- MyDAO.myQuery1  --> counter stats, ..
      +-- MyDAO.myQuery2  --> counter stats, ..   
{% endhighlight %}

Some tools can offer syntaxic sugar help, to show aggregation per sub-path :so inline intermediate nodes to be able to show flat results as Dropwizard Metrics 
{% highlight text %}
aggregated MyDAO.myQuery1 =   MyWebService.myServices1/MyDAO.myQuery1  +  MyWebService.myServices2/MyDAO.myQuery1
{% endhighlight %}


In badly configured tools, you unfortunatly get the full exact java stack trace, instead of the applicative stack trace with all the intermediate applicative code, and boilerplate framework code:
{% highlight text %}
at com.oracle.driver...
at org.sef4j ...
at org.hibernate...
at org.springframework...
at java.lang.reflect..
at 
at myapp.MyDAO
at 
at myapp.MyWebService
at ..
at org.springframework
at javax.servlet..
at tomcat..
{% endhighlight %}


With Sef4j, you can instrument method with relevant applicative name-values 

{% highlight java %}
public class MyWebService {
   public MyResult myService1(param1, param2...) {
		try(AppCallStack.method("myService").WithParam("param1", param1).push())   // ==> attach inherited property to current Thread
			MyDAO.myQuery1()
			..
		}// => finally... pop("myService")
   }
}
{% endhighlight %}


In Sef4j, virtual nodes can be introcuced to split counters per applicative criteria
{% highlight java %}
public class MyWebService {
   public MyResult myService1(param1, param2...) {
		try(AppCallStack.method("myService").WithParam("param1", param1).push())   // ==> attach inherited property to current Thread
			try (AppCallStack.method("perParam:" + param1)) {
			
				MyDAO.myQuery1()
				..
			} // => finally pop("perParam")
		}// => finally... pop("myService")
   }
}
{% endhighlight %}

You will get a tree of stat counter like


{% highlight text %}
/
+-- MyWebService.myServices1  --> counter stats, ..
|     |
|     +-- perParam1:value1
|     |    +-- MyDAO.myQuery1  --> counter stats, ..
|     |    +-- MyDAO.myQuery2  --> counter stats, ..
|     +-- perParam1:value2
|     |    +-- MyDAO.myQuery1  --> counter stats, ..
|     |    +-- MyDAO.myQuery2  --> counter stats, ..
|     +-- perParam1:value3
|     |    +-- MyDAO.myQuery1  --> counter stats, ..
|     |    +-- MyDAO.myQuery2  --> counter stats, ..
|     ...
{% endhighlight %}



