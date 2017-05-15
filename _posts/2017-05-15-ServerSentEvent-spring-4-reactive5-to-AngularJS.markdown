---
layout: post
title:  "Using Server-Sent-Event from Spring4 and Reactive Spring5 to AngularJS"
date:   2017-05-15 23:30:00
categories: 
tags: web http SSE server-sent-event springboot spring5 springweb AngularJs
---

<H1>Web Technologies to do notifications</H1>  

Imagine you want to develop a mini real-time chat application using 2 web clients, connected to a web server.<BR/>
There are several possibilities to receive updates: 
<ul>
<li>Old-school: explicit Reload button...</li>
<li>Long polling with timer to auto-refresh</li>
<li>WebSocket</li>
<li>Server-Sent-Event</li>
</ul>

<H3>WebSocket</H3>
WebSocket is the most famous, but is a little overkill. <BR/>
It is protocol negociation between the browser and the server to change the Http connection to a TCP connection, 
and re-use it as a dual-channel binary communication.<BR/>
On a LAN it works great... On a WAN with HTTP Firewalls, Http Proxies and Http LoadBalancer (Stateless Non Session Sticky), it is much more difficult. <BR/>


<H3>SSE : Server-Sent-Event</H3>

Server-Sent-Event are much simpler.<BR/>
It is a regular Http GET request with media type "text-stream", which will read a stream of events as single line text, 
each line starting as "data: ...", "id: ", "event: ", or "retry: ".<BR/> 
Every "data: " is interpreted as a message event, which trigger a client-side javascript callback method.

{% highlight javascript %}
var eventSource = new EventSource("/chat/room/Default/sse");

eventSource.onmessage = function(event) {
    var eventData = JSON.parse(event.data);
    console.log("received event data", eventData);
};
{% endhighlight javascript %}


Typical streams of events for a chat: 
{% highlight text %}
event: chat
data: {"id":"123", "from":"bobby", "msg":"Hello John", "chatRoom":"Default", "time":"02:33:48" }

id: 124
data: {"id":"124", "from":"john", "msg":"Hi Bob", "chatRoom":"Default", "time": "02:40:10" }

id: 125
data: {"id":"125", "from": "bobby", "msg":"How are you?", "chatRoom":"Default", "time": "02:40:15" }
{% endhighlight %}

   
Looking at network level, it is plain-old Http protocol: it keep the Http socket connection opened for a while, and read results until deconnection.<BR/>
The server will close its connection, typically after 30s  (default on tomcat for example), 
and the client simply re-open a new one to possibly another server,
and re-establish a query passing the "last-event-id" Http header attribute to get newly emitted events since the last one already received.   


To test it, simply launch a curl command 
{% highlight shell %}
curl -H 'Accept: text/event-stream' http://localhost:8080/chat/room/Default/sse

data: {"id": "1", ..}
data: {"id": "2", ..}
data: {"id": "3", ..}
.. 
... ommited: receive all (recent) known events from server
..
data: {"id":"123", .. }

... waiting reading more
... connection closed after 30s
{% endhighlight shell %}

So more interrestingly, add the header "last-event-id", supposing you have already listen up to id "123":
{% highlight shell %}
curl -H 'Accept: text/event-stream' -H 'last-event-id: 123' http://localhost:8080/chat/room/Default/sse

data: {"id": "124", ..}
data: {"id": "125", ..}

... waiting reading more
... connection closed after 30s
{% endhighlight shell %}



<H1>Server implementation using springboot Spring4 : SseEmitter</H1>

{% highlight java %}
@RestController
@RequestMapping("/app")
public class MyRestController {
	
	private static final Logger LOG = LoggerFactory.getLogger(MyRestController.class);
	
	@Autowired
	private ChatHistoryService chatHistoryService;
	
	... 
	
	@GetMapping(path = "/chat/room/{chatRoom}/sseSpring4", produces = "text/event-stream")
	public SseEmitter subscribeChatMessages(
	 		@PathVariable("chatRoom") String chatRoom,
	   		@RequestHeader(name="last-event-id", required=false) String lastEventId
	   		) {
	    LOG.info("subscribeMessagesSpring4 lastEventId:" + lastEventId);
		ChatRoomEntry chatRoomEntry = chatHistoryService.getChatRoom(chatRoom);
	    return chatRoomEntry.subscribe(lastEventId);
	}
	
	// handle normal "Async timeout", to avoid logging warn messages every 30s per client...
	@ExceptionHandler(value = AsyncRequestTimeoutException.class)  
    public String asyncTimeout(AsyncRequestTimeoutException e){  
        return null; // "SSE timeout..OK";  
    }
	
}

... 
public class ChatRoomEntry {

	private List<SseEmitter> emitters = new ArrayList<>();

	@Override
	public void onPostMessage(ChatMessageEntry msg) {
		if (emitters.isEmpty()) {
			return;
		}
		SseEventBuilder evtBuilder = SseEmitter.event()
				.id(Integer.toString(msg.id))
				.name("chat")
				.data(msg);
		for(SseEmitter emitter : emitters) {
			try {
				emitter.send(evtBuilder);
			} catch (IOException ex) {
				LOG.error("Failed to send msg to emitter", ex);
			}
		}
	}
	
	public SseEmitter subscribe(String lastEventIdText) {
		SseEmitter emitter = new SseEmitter();
        Integer lastId = lastEventIdText != null? Integer.parseInt(lastEventIdText) : null;
		List<ChatMessageEntry> replayMsgs = (lastId != null)?  
				chatRoom.listMessagesSinceLastId(lastId) : chatRoom.listMessages();
		for(ChatMessageEntry msg : replayMsgs) {
			if (lastId != null && msg.id <= lastId) {
				continue;
			}
			try {
				emitter.send(msg);
			} catch (IOException ex) {
				LOG.error("Failed to re-send msg to emitter, ex:", ex.getMessage() + " => complete with error ... remove,disconnect");
				emitter.completeWithError(ex);
			}
		}
		
        // note: should synchronised replayMessage ... add()!
        emitters.add(emitter);
        
        emitter.onCompletion(() -> {
        	LOG.debug("onCompletion -> remove emitter"); // log as DEBUG... logged every 30s per client! 
        	emitters.remove(emitter);
        });

        emitter.onTimeout(() -> {
        	LOG.debug("onTimeout -> remove emitter");  // log as DEBUG... logged every 30s per client!
        	emitters.remove(emitter);
        });

        return emitter;
	}

{% endhighlight java %}
    
Notice that you can change the configuration timeout which is 30s by default: edit file config/application.yaml
{% highlight yaml %}
spring:
  mvc:
    async:
      # default: 30 seconds... should be higher..
      # for test only= > 15s!
      request-timeout: 15000
{% endhighlight yaml %}


<H1>Server implementation using springboot Spring5 : Flux&lt;ServerSentEvent&lt;..&gt;&gt;</H1>

Spring5 is currently in SNAPSHOT as of 2017-05, you need to use maven SNAPSHOT versions (+maven SNAPSHOTS repositories).
See from <A href="https://start.spring.io/">https://start.spring.io/</A>, by choosing the top-level right combo-box version "with springboot version:2.0.0.BUILD-SNAPSHOT" instead of "1.5.3" 

{% highlight xml %}
    <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.0.BUILD-SNAPSHOT</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
..
	<dependencies>
		..
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webflux</artifactId>
		</dependency>
	
{% endhighlight xml %}    

Then the implementation is much shorter

{% highlight java %}
@RestController
@RequestMapping("/app")
public class MyRestController {
	
	private static final Logger LOG = LoggerFactory.getLogger(MyRestController.class);
	
	@Autowired
	private ChatHistoryService chatHistoryService;
	
	... 

	@GetMapping(path = "/chat/room/{chatRoom}/sseSpring5", produces = "text/event-stream")
    public Flux<ServerSentEvent<ChatMessageEntry>> subscribeChatMessages_spring5(
    		@PathVariable("chatRoom") String chatRoom,
    		@RequestHeader(name="last-event-id", required=false) String lastEventId
    		) {
        ChatRoomEntry chatRoomEntry = chatHistoryService.getChatRoom(chatRoom);
        if (chatRoomEntry == null) {
        	return null;
        }
        LOG.info("subscribeMessagesSpring5 lastEventId:" + lastEventId);
        return chatRoomEntry.subscribe(lastEventId);
    }


... 
public class ChatRoomEntry {

	private ReplayProcessor<ServerSentEvent<ChatMessageEntry>> replayProcessor = 
		ReplayProcessor.<ServerSentEvent<ChatMessageEntry>>create(100);
	
	@Override
	public void onPostMessage(ChatMessageEntry msg) {
		ServerSentEvent<ChatMessageEntry> event = ServerSentEvent.builder(msg)
				.event("chat")
				.id(Integer.toString(msg.id)).build();
		replayProcessor.onNext(event);
	}

	public Flux<ServerSentEvent<ChatMessageEntry>> subscribe(String lastEventId) {
		Integer lastId = (lastEventId != null)? Integer.parseInt(lastEventId) : null;
		return replayProcessor.filter(x -> lastId == null || x.data().get().id > lastId);
	}
}
{% endhighlight java %} 	


<H1>Implementation on client-side using AngularJS</H1>

The code on client-side is really small.

Here is a "rich" debugging screen with options to show connection status, 
choose between spring4 and spring5 server implementation to call,
and format messages.  

<img src="{{site.url}}/assets/posts/2017-05-15-ServerSentEvent-spring-4-reactive5-to-AngularJS/screenshot-test-spring-reactive-web.svg"/>

{%highlight javascript %}
'use strict';
angular.module('myapp', [ ])
.controller('MyController', function($scope, $http) {
    var self = this;

    self.useImpl = "spring5";
    self.from = "me";
    self.chatRoom = "Default";
    self.msgToSend = "";
    self.formattedMessages = '';
    
    self.eventSource = null;
    self.lastEventId = 0;
    self.connStatus = '';
    
    self.onInit = function() {
        // $scope.on("destroy", function() { self.onDispose(); });
        self.connectEventSource();
    };
    
    self.onDispose = function() {
        console.log("onDispose");
        self.eventSource.close();
    };
    
    self.connectEventSource = function() {
        console.log("subscribe chatRoom: " + self.chatRoom + " useImpl:" + self.useImpl + " lastEventId:" + self.lastEventId);
        self.eventSource = new EventSource('/app/chat/room/' + self.chatRoom + '/subscribeMessagesSpring' + (self.useImpl === 'spring4'? '4' : '5'),
                { id: self.lastEventId });
        
        if (self.lastEventId > 0) {
            self.eventSource.id = self.lastEventId;
        }
        
        self.eventSource.addEventListener('open', function(e) {
            console.log("onopen", e);
            self.connStatus = 'connected';
            $scope.$apply();
        }, false);

        self.eventSource.addEventListener('error', function(e) {
            if (e.eventPhase == EventSource.CLOSED) {
              console.log('connection closed (..reconnect)', e);
              self.connStatus = 'connection closed (..auto reconnect in 3s)';
            } else {
              console.log("onerror", e);
              self.connStatus = 'error\n';
            }
            $scope.$apply();
          }, false);
        
        self.eventSource.addEventListener('message', function(e) {
            console.log("onmessage", e);
            self.handleMessageEvent(e);
            $scope.$apply();
        }, false);
        
        self.eventSource.addEventListener("chat", function(e) {
            // never called? .. cf onmessage !
            console.log("on event 'chat'", e);
            self.handleMessageEvent(e);
            $scope.$apply();
        });
    }

    self.handleMessageEvent = function(e) {
        var msg = JSON.parse(e.data);
        self.lastEventId = msg.id;
        self.eventSource.id = self.lastEventId;
        self.addFormattedEvent(msg.id, new Date(msg.date), msg.from, msg.msg);
    };
    
    self.onClickSendMsg = function () {
        console.log("send");
        var msg = self.msgToSend;
        // self.addFormattedEvent(new Date(), self.from, msg);
        
        var req = { onBehalfOf: self.from, msg };
        self.sending = true;
        $http.post("/app/chat/room/" + self.chatRoom, req)
        .then(function() {
          self.sending = false;
        }, function(err) {
          self.addFormattedEvent(-1, new Date(), self.from, "Failed to send " + msg + ":" + err);
          self.sending = false;
        });
    };
    
    self.addFormattedEvent = function(id, date, from, msg) {
       self.formattedMessages += "[" + id + "] " + date.getHours() + ":" + date.getMinutes() + ":" + date.getSeconds() + " " +  
           ((self.from === from)? "" : "(" + from + ") ") + 
           msg + "\n\n";
    }
    
    self.onClickReconnect = function() {
        self.formattedMessages += "** Click RECONNECT **";
        self.eventSource.close();
        self.connectEventSource();
    };
    
    self.onInit();
});
{%endhighlight javascript %}


<H1>Testing It...</H1>

Run it: start main class from springboot within your IDE  (or mvn spring-boot:run)
<BR/>

Testing using web client: open 2 browsers on url 
   http://localhost:8081/chat-angularjs/index.html

<img src="{{site.url}}/assets/posts/2017-05-15-ServerSentEvent-spring-4-reactive5-to-AngularJS/screenshot-web-client.png"/>
<BR/>


Testing using curl shell commands:

testing SSE GET request (with timeout after 30s)

<img src="{{site.url}}/assets/posts/2017-05-15-ServerSentEvent-spring-4-reactive5-to-AngularJS/screenshot-curl-last-event-id.png"/>

Test posting message directly from curl command  (with verbose mode):
<img src="{{site.url}}/assets/posts/2017-05-15-ServerSentEvent-spring-4-reactive5-to-AngularJS/screenshot-post-message-verbose.png"/>


<H1>Details analysis of timed-events logs</H1>

Here are commented logs from both client and server:
 
Logs on server:

<img src="{{site.url}}/assets/posts/2017-05-15-ServerSentEvent-spring-4-reactive5-to-AngularJS/screenshot-logs-server.png"/>

{%highlight text %}
fr.an.tests.MyRestController             : msg #1: "(BOT) server start"
  // at server startup, msg#1 is published (no client connected yet)
  // on server-side, List of recent msg: [ msg#1 ], 
  // msg sequence number incremented to 2  

fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:null  
  // <= initial connection from client "bob"
  // not "last-event-id" known from client => client receive all recent msgs from server: [ msg#0 ]
  // client1 as now "last-event-id: 1" 

  // after 30s, client1 GET request is timeouting
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:1     
  // reconnect from client1 => re-susbscrive to events, using "last-event-id: 1"
  // no new messages since #1 => no events sent, connection pending
  
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:1     
  // again .. GET Timeout + reconnect
  
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:null
  // <= initial connection with last-event-id from client2 "john"
  // not "last-event-id" known from client2 => client receive all recent msgs from server: [ msg#0 ]
  // client2 as now "last-event-id: 1" 
  
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:1
  // timeout+reconnect from client (1 or 2?)

fr.an.tests.MyRestController             : receive msg chatRoom:Default from:john msg: Hello bob
  // POST msg#2 from john => published to both connected client1 & client2 
  // client1 connected => status:  received msgs:[msg#1, msg#2]  last-event-id: 2
  // client2 connected => status:  received msgs:[msg#1, msg#2]  last-event-id: 2
  
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:2
  // client1 timeout .. reconnect after 3s using "last-event-id: 2"
  // no new messages received
  
  // client2 timeout ... will reconnect next in 3s..
  
fr.an.tests.MyRestController             : receive msg chatRoom:Default from:bob msg: Hi john
  // POST msg #3 to client1   (client2 currently disconnected)
  
  /// status:
  // client1: connected,    received msgs:[msg#1, msg#2, msg#3]  last-event-id: 3
  // client2: disconnected, received msgs:[msg#1, msg#2]  last-event-id: 2
  
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:2
  // reconnection from client2 using last-event-id: 2
  // msg #3 was NOT received during 3s re-connection delay => client2 received missing msg #3
  // status
  // client1: connected, received msgs:[msg#1, msg#2, msg#3]  last-event-id: 3
  // client2: connected, received msgs:[msg#1, msg#2, msg#3]  last-event-id: 3
   
   
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:3
  // reconnection from client2...msg #3 already received

fr.an.tests.MyRestController             : receive msg chatRoom:Default from:john msg: How are you ?
fr.an.tests.MyRestController             : receive msg chatRoom:Default from:bob msg: fine
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:4
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:5
fr.an.tests.MyRestController             : subscribeMessagesSpring5 lastEventId:5
{%endhighlight text %}


Logs from client-side using chorme dev tools:
<img src="{{site.url}}/assets/posts/2017-05-15-ServerSentEvent-spring-4-reactive5-to-AngularJS/screenshot-logs-chrome.png"/>



<H1>Conclusion</H1>

Server-Sent-Event works great, and are really simple !
<BR/>

SpringBoot and new Spring5 Reactor are awesome.
<BR/>

The full repository code is available here:
<A href="https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-spring-reactive-web">https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-spring-reactive-web</A>



