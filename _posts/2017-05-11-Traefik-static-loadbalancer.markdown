---
layout: post
title:  "Using Traefik as a simple Http Reverse Proxy / LoadBalancer"
date:   2017-05-11 22:30:00
categories: 
tags: web http reverse-proxy traefik
---

<H1>Which Lightweigth Http Reverse Proxy ?</H1>

I was looking for a "simple" http reverse proxy.<BR/>
Apache HttpD, NGinx, or any other reverse proxy would do the job.

Traefik has many advantages:
<ul>
<li>very easy to install: simply copy a single executable file (statically linked executable, NO installation required)</li>
<li>ligthweigth proxy</li>
<li>run on windows/linux (2 different go executables)</li>
<li>relatively easy to configure</li>
<li>has rich flexible concepts : entrypoints(:8080) - frontends(logical domain/routes) - backend(logical servers group or api) - servers(physical server)</li>
<li>auto reload configuration file if changing</li>
<li>evolutive to use dynamic discovery routing, for clustering with Docker,Mesos,Kubernetes,Consul,Zookeeper,...</li>
</ul>

<H1>Performing a local test: port 8080 - proxy to ports 8081 or 8082</H1>

One of the goals is to obtain "High Availability", by load-balancing to at least 2 server instances for the same logical service.<BR/>

<ul>
<li> node1 : running on port 8081  (port may be visible to localhost only, using bind ::1) </li>
<li> node2 : running on port 8082</li> 
<li> reverse-proxy: running on port 8080 ... the public port url "http(s)://host:8080"  for client users</li> 
</ul>

The load-balancer will re-write every HTTP request to one of the node, and re-write the HTTP response back from the node to the client.

Benefits:
To redeploy my server without interruption of service, 
I want to kill node1, redeploy it, restart node1, then kill node2, redeploy it, start node2.
<BR/>


<H2>Preparing spring-boot web project to 2 instance nodes on port 8081 and 8082</H2>
I have created a hello-world web server, using springboot project <A href="https://start.spring.io/">https://start.spring.io/</A><BR/>
(choose component "web" + click "download", then "mvn package", import in your IDE, add @RestController, start main!) 

My Rest application contains 2 resource:
a static html page for testing, and a Rest-Json endpoint.

Using springboot jar application, the static file is src/main/resources/static/app/index.html
{% highlight html %}
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>test-spring-web</title>
    <meta name="description" content="">
</head>
<body>
<H1>Test Springboot Web</H1>
</body>
</html>
{% endhighlight  %}

I have also added a redirect html page from url "/index.html" to "/app/index.html"</BR>
The ultimate goal is to route requests using url PrefixPath

{% highlight html %}
<!doctype html>
<html>
<head>
    <meta http-equiv="refresh" content="0; url=app/index.html" />
</head>
<body>
Redirect... <a href="app/index.html">app/</a>
</body>
</html>
{% endhighlight  %}




The Rest-Json endpoint is
{% highlight java %}
@RestController
@RequestMapping("/app")
public class MyRestController {

	@GetMapping("/helloParams")
	public Map<String,String> helloParams() {
		Map<String,String> res = new HashMap<>();
		res.put("hello", "world");
		return res;
	}
}
{% endhighlight java %}

To run it (using eclipse or equivalent shell command): it is a simple main app
{% highlight shell %}
java -jar myapp.jar
{% endhighlight %}


By default, it run on port 8080... and I can test it using shell curl command:
{% highlight shell %}
curl -X GET -H 'Accept: application/json' http://localhost:8080/app/helloParams
{% endhighlight shell %}

You can check springboot log file to see 
{% highlight text %}
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.0.BUILD-SNAPSHOT)

  INFO [main] f.a.t.TestSpringReactiveWebApplication   : Starting TestSpringReactiveWebApplication on ..  with PID 10944 (..)
  INFO [main] f.a.t.TestSpringReactiveWebApplication   : No active profile set, falling back to default profiles: default
  INFO [main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@46271dd6: startup date [Thu May 11 23:54:41 CEST 2017]; root of context hierarchy
  INFO [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8082 (http)
.. 
  INFO [main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/app/helloParams],methods=[GET]}" onto public java.util.Map<java.lang.String, java.lang.String> fr.an.tests.MyRestController.helloParams()
..
  INFO [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8082 (http)
  INFO [main] f.a.t.TestSpringReactiveWebApplication   : Started TestSpringReactiveWebApplication in 2.674 seconds (JVM running for 3.056)
{% endhighlight text %}



To run 2 instances on 2 different network ports, you need either to change jvm arguments 
{% highlight shell %}
java -jar myapp.jar --server.port=8081 &
java -jar myapp.jar --server.port=8082 &
{% endhighlight %}

or to change application.yaml
{% highlight yaml %}
server:
  port: 8081
{% endhighlight yaml %}
... but then I need anyway to have 2 different projects, or back to 2 different jvm argsuments
{% highlight shell %}
java -jar myapp.jar -Dspring.profiles.active=node1  # ... using application-node1.yaml
java -jar myapp.jar -Dspring.profiles.active=node2  # ... using application-node2.yaml
{% endhighlight shell %}


<H2>Preparing Traefik configuration</H2>

Download the 40Mo executable file from the website (or use Docker)

Edit the traefik.toml configuration file:
{% highlight text %}
defaultEntryPoints = ["http"]

[entryPoints]
  [entryPoints.http]
  address = ":8080"

[file]
# auto reload if config change  (necessary for conf .. otherwise server not scanned => 404... ??)

[frontends]
  [frontends.frontend1]
  backend = "backend1"
    [frontends.frontend1.routes.test_1]
    rule = "Host: localhost"
    # rule = "PathPrefix: /app"
    # rule = "Host: localhost; PathPrefix: /app"


[backends]
  [backends.backend1]
    [backends.backend1.LoadBalancer]
      method = "drr"
    [backends.backend1.healthcheck]
      path = "/app/health"
      interval = "3s"

    [backends.backend1.servers.server1]
    url = "http://127.0.0.1:8081"
    weight = 1

    [backends.backend1.servers.server2]
    url = "http://127.0.0.1:8082"
    weight = 1

{% endhighlight text %}

Cf below for healthcheck section explanation.


<H2>Run it</H2>

{% highlight shell %}
./traefik --configFile=config.toml
{% endhighlight shell %}



You can test ... it run OK!
You URL is now accessible from 8080, and will redirect internally to 8081 and 8082

Traefik does not have a lot of log using loglevel=INFO... Only 6 lines at startup:
{% highlight text %}
INFO[2017-05-12T00:01:20+02:00] Traefik version v1.3.0-rc1 built on 2017-05-05_01:42:07PM 
INFO[2017-05-12T00:01:20+02:00] Using TOML configuration file /home/arnaud/downloadTools/web/reverse-proxy/traefik/node1/config.toml 
INFO[2017-05-12T00:01:20+02:00] Preparing server http &{Network: Address::8080 TLS:<nil> Redirect:<nil> Auth:<nil> Compress:false} 
INFO[2017-05-12T00:01:20+02:00] Starting provider *file.Provider {"Watch":true,"Filename":"/home/arnaud/downloadTools/web/reverse-proxy/traefik/node1/config.toml","Constraints":null} 
INFO[2017-05-12T00:01:20+02:00] Starting server on :8080                     
INFO[2017-05-12T00:01:20+02:00] Server configuration reloaded on :8080       
{% endhighlight text %}

For example, if you don't write the empty section "[file]" in the traefik.toml, you have exactly the same logs at startup, so no warn message... but nothing works!

{% highlight text %}

[file]

# empty section
# auto reload if config change?
# but necessary for conf .. otherwise server not scanned => 404... ??
# Why Traefik, Why ???? You wanted me to loose few hours ...

{% endhighlight text %}



By changing to loglevel=DEBUG, you got more informations about the "backend servers routes":

{% highlight text %}
...
DEBU[2017-05-12T00:05:20+02:00] Creating frontend frontend1                  
DEBU[2017-05-12T00:05:20+02:00] Wiring frontend frontend1 to entryPoint http 
DEBU[2017-05-12T00:05:20+02:00] Creating route test_1 Host: localhost        
DEBU[2017-05-12T00:05:20+02:00] Creating backend backend1                    
DEBU[2017-05-12T00:05:20+02:00] Creating load-balancer drr                   
DEBU[2017-05-12T00:05:20+02:00] Creating server server1 at http://127.0.0.1:8081 with weight 1 
DEBU[2017-05-12T00:05:20+02:00] Creating server server2 at http://127.0.0.1:8082 with weight 1 
INFO[2017-05-12T00:05:20+02:00] Server configuration reloaded on :8080       
{% endhighlight text %}


<H3>Activating access.log file</H3>

Add this option 
{% highlight text %}
   --accesslogsfile=access.log
{% endhighlight text %}

In the access.log, you can see http requests received and processed successfully (both for html : 304 = OK not-modified, and for Rest/Json : 200 OK)
{% highlight text %}
::1 - - [12/May/2017:00:08:51 +0200] "GET /app/index.html HTTP/1.1" 304 0 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 14 "frontend1" "http://127.0.0.1:8082" 2ms
::1 - - [12/May/2017:00:08:51 +0200] "GET /app/index.html HTTP/1.1" 304 0 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 15 "frontend1" "http://127.0.0.1:8081" 2ms
::1 - - [12/May/2017:00:10:21 +0200] "GET /app/helloParams HTTP/1.1" 200 17 "" "curl/7.38.0" 16 "frontend1" "http://127.0.0.1:8082" 34ms
::1 - - [12/May/2017:00:10:23 +0200] "GET /app/helloParams HTTP/1.1" 200 17 "" "curl/7.38.0" 17 "frontend1" "http://127.0.0.1:8081" 32ms
::1 - - [12/May/2017:00:10:41 +0200] "GET /app/helloParams HTTP/1.1" 200 17 "" "curl/7.38.0" 18 "frontend1" "http://127.0.0.1:8082" 2ms
{% endhighlight text %}



You can have a "start-traefik.sh" shell script, basically to avoid remember optional debug arguments
{% highlight shell %}
ARGS=

# optionnal for verbose output
# ARGS="$ARGS --loglevel=DEBUG"
ARGS="$ARGS --loglevel=INFO"

# optionnal redirect logs in file
# ARGS="$ARGS --traefiklogsfile=traefik.log"

# optionnal, to trace all access
ARGS="$ARGS --accesslogsfile=access.log"

# optionnal to see admin console
# ARGS="$ARGS --web --web.address=8079"

./traefik --configFile=config.toml $ARGS
{% endhighlight shell %}



<H2>HealthCheck ... for detecting up/down nodes</H2>

Traefik does not mark nodes up/down based on tcp connection error!!
<BR/>
Traefik does not auto-retry on node2 if node1 failed to answer... Damned!
<BR/>
By using basic round-robin, and NO health-check, you obtain alternatively 50% request OK, and 50% ERROR!!
<BR/>

For the client app, you have 50% "Bad Gateway" http responses.
<BR/>
Maybe if your client is smart enough, it will retry few times, and hopefully (if it is alone), 
the round-robin will finish to choose the right node runnnig, so the client will have its answer and stop it's retries loop.


In the log (with loglevel=INFO), you see only WARN for 50% failing requests.. no messages for "marked down" node.
{% highlight text %}
WARN[2017-05-11T23:29:54+02:00] Error forwarding to http://127.0.0.1:8082, err: dial tcp 127.0.0.1:8082: getsockopt: connection refused 
WARN[2017-05-11T23:29:59+02:00] Error forwarding to http://127.0.0.1:8082, err: dial tcp 127.0.0.1:8082: getsockopt: connection refused 
WARN[2017-05-11T23:29:59+02:00] Error forwarding to http://127.0.0.1:8082, err: dial tcp 127.0.0.1:8082: getsockopt: connection refused 
{% endhighlight text %}
 
In the access.log, you see the 50% OK (http code 200) and 50% ERROR (http code 503):
{% highlight text %}
::1 - - [11/May/2017:23:29:58 +0200] "GET /app/index.html HTTP/1.1" 200 213 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 5 "frontend1" "http://127.0.0.1:8081" 3ms
::1 - - [11/May/2017:23:29:59 +0200] "GET /app/index.html HTTP/1.1" 502 11 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 6 "frontend1" "http://127.0.0.1:8082" 0ms
::1 - - [11/May/2017:23:29:59 +0200] "GET /app/index.html HTTP/1.1" 200 213 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 7 "frontend1" "http://127.0.0.1:8081" 2ms
::1 - - [11/May/2017:23:29:59 +0200] "GET /app/index.html HTTP/1.1" 502 11 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 8 "frontend1" "http://127.0.0.1:8082" 0ms
::1 - - [11/May/2017:23:30:00 +0200] "GET /app/index.html HTTP/1.1" 200 213 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 9 "frontend1" "http://127.0.0.1:8081" 2ms
::1 - - [11/May/2017:23:30:00 +0200] "GET /app/index.html HTTP/1.1" 502 11 "" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36" 10 "frontend1" "http://127.0.0.1:8082" 0ms
{% endhighlight text %}



The healthcheck section is necessary for Traefik to detect when your service is unavailable.<BR/>
So I have added a dummy endpoint "/app/health" that does nothing, and it is here pinged every 3 seconds by Traefik.

{% highlight text %}
[backends]
  [backends.backend1]
    [backends.backend1.LoadBalancer]
      method = "drr"
    [backends.backend1.healthcheck]
      path = "/app/health"
      interval = "3s"
{% endhighlight text %}


{% highlight text %}
WARN[2017-05-11T22:28:33+02:00] HealthCheck has failed [http://127.0.0.1:8081]: Remove from server list 
WARN[2017-05-11T22:28:36+02:00] HealthCheck is still failing [http://127.0.0.1:8081] 
WARN[2017-05-11T22:28:39+02:00] HealthCheck is still failing [http://127.0.0.1:8081] 
WARN[2017-05-11T22:28:42+02:00] HealthCheck is still failing [http://127.0.0.1:8081] 
{% endhighlight text %}

There is no log message for "server up"... but you have plenty of repeated messages "HealthCheck still failing" when one server is down.

Doing a ping with a frequency of 3 seconds is not CPU intensive... it is only a local socket open, and no complex action performed for "http GET /app/health".
It is not very realistic eithre to have a frequency of ~10ms ... 

During the 3seconds, the server can be estimated has "up" by Traefik, but be really "down"... 
Therefore, I can have an interruption of service during 3 seconds!  This is not 100% High Availability<BR/>


<H2>Conclusion</H2>

Static routing in Traefik is only a very limited use-case, compared to dynamic service-discovery that Traefik is able to perform using Mesos/Kubernetes/Consul/.. !!

It is a pity that Traefik does not automatically mark node down when TCP connection failed, 
and silently retry to other nodes when such a connection fail!

However, Traefik is really easy to install and use at start, and very powerfull to continue with.


