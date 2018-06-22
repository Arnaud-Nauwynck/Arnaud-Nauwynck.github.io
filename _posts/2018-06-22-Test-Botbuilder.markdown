---
layout: post
title:  "Test Botbuilder"
date:   2018-06-22 08:30:00
categories: 
tags: bot chat
---


<H1>Setup project</H1>

https://tutorials.botsfloor.com/lets-make-a-chatbot-microsoft-bot-framework-node-js-7da211149c2f

{% highlight shell %}
$ mkdir test-botbuilder; cd test-botbuilder
$ git init; echo ".project" >> .gitignore; 
$ echo "node_modules" >> .gitignore; echo package-lock.json >> .gitignore

$ npm init
$ npm install --save botbuilder
$ npm install --save restify

$ git add . ; git ci -m "init"

$ gedit app.js &
$ node app.js

{% endhighlight %}

Edit the bot javascript code:
This bot simply respond "you said: XX", when you send a "XX" message
 
{% highlight javascript%}
var restify = require('restify'); 
var builder = require('botbuilder');  

// Setup Restify Server 
var server = restify.createServer(); 
server.listen(process.env.port || process.env.PORT || 3978, 
function () {    
    console.log('%s listening to %s', server.name, server.url);  
});  

// chat connector for communicating with the Bot Framework Service 
var connector = new builder.ChatConnector({     
    appId: process.env.MICROSOFT_APP_ID,     
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Listen for messages from users  
server.post('/api/messages', connector.listen());  

// Receive messages from the user and respond by echoing each message back (prefixed with 'You said:') 

var bot = new builder.UniversalBot(connector, function (session) {     
session.send("You said: %s", session.message.text); 
});
{% endhighlight %}


Install the BotEmulator:

Open url https://github.com/Microsoft/BotFramework-Emulator/releases/
see https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download

Download the AppImage, make it executable, and run it

{% highlight shell %}
$ wget https://github.com/Microsoft/BotFramework-Emulator/releases/download/v4.0.15-alpha/botframework-emulator-4.0.15-alpha-x86_64.AppImage

$ chmod u+x botframework-emulator-4.0.15-alpha-x86_64.AppImage
 
$ ./botframework-emulator-4.0.15-alpha-x86_64.AppImage &  

{% endhighlight %}


Remarks: when running the AppImage emulator, the image is unzipped and run locally under /tmp/.mount_botfra*
You can see it is a nodejs application:

{% highlight shell %}
$ cd/tmp/.mount_botfra8ivddr
$ tree
.
├── app
│   └── views_resources_200_percent.pak
...
├── AppRun
├── botframework-emulator.desktop
├── botframework-emulator.png -> usr/share/icons/hicolor/512x512/apps/botframework-emulator.png
86 directories, 219 files
{% endhighlight %}



<img src="{{site.url}}/assets/posts/2018-06-22-Test-Botbuilder/screenshot1-botbuilder.png" />
      


Go in the emulator window, and create a new bot configuration
Choose 
- name: test-bot1
- endpoint: http://localhost:3978/api/messages

<img src="{{site.url}}/assets/posts/2018-06-22-Test-Botbuilder/screenshot2-create-bot-config.png" />

The emulator will save it to a file called  
{% highlight json %}
{
    "name": "test-bot1",
    "description": "",
    "secretKey": "",
    "services": [
        {
            "appId": "",
            "id": "cab19150-75e7-11e8-841c-5921b383ac3f",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "http://localhost:3978/api/messages"
        }
    ]
}
{% endhighlight %}


You have now a botemulator dialog for sending/receiving message with your bot

<img src="{{site.url}}/assets/posts/2018-06-22-Test-Botbuilder/screenshot3-botemulator.png" />



Then go in the chat type area, type a first message, "hello"
You can see the message is sent... and the bot replied a message : "you said: hello" 

<img src="{{site.url}}/assets/posts/2018-06-22-Test-Botbuilder/screenshot4-send-receive-message.png" />

You can see the logs in your nodejs bot console:

{% highlight text %}
$ node app.js
restify listening to http://[::]:3978
WARN: ChatConnector: receive - emulator running without security enabled.
ChatConnector: message received.
The Bot State API is deprecated.  Please refer to https://aka.ms/I6swrh for details on how to replace with your own storage.
UniversalBot("*") routing "hello" from "emulator"
Library("*").findRoutes() explanation:
	ActiveDialog(0.1)
/ - waterfall() step 1 of 1
/ - Session.send()
/ - Session.sendBatch() sending 1 message(s)
The Bot State API is deprecated.  Please refer to https://aka.ms/I6swrh for details on how to replace with your own storage.
{% endhighlight %}



When clicking on the message you sent, you can see that the emulator is doing a http POST message with this content to your bot

<img src="{{site.url}}/assets/posts/2018-06-22-Test-Botbuilder/screenshot5-send-message-detail.png" />
 
{% highlight json%}
{
  "channelData": {
    "clientActivityId": "1529648584784.3610234534525605.0"
  },
  "entities": [
    {
      "requiresBotState": true,
      "supportsListening": true,
      "supportsTts": true,
      "type": "ClientCapabilities"
    }
  ],
  "from": {
    "id": "default-user",
    "name": "User"
  },
  "id": null,
  "locale": "en-US",
  "text": "hello",
  "textFormat": "plain",
  "timestamp": "2018-06-22T06:24:11.736Z",
  "type": "message"
}
{% endhighlight %}


Then when clicking on the bot response message, you can see that your bot sent this json response to the emulator:

{% highlight json%}
{
  "channelId": "emulator",
  "conversation": {
    "id": "ea9bbbd0-75e7-11e8-841c-5921b383ac3f|livechat"
  },
  "from": {
    "id": "cab19150-75e7-11e8-841c-5921b383ac3f",
    "name": "Bot",
    "role": "bot"
  },
  "id": "ed5a8db0-75e7-11e8-99e3-7f65f723bf79",
  "inputHint": "acceptingInput",
  "localTimestamp": "2018-06-22T08:46:01+02:00",
  "locale": "en-US",
  "recipient": {
    "id": "default-user",
    "role": "user"
  },
  "replyToId": "ed302250-75e7-11e8-99e3-7f65f723bf79",
  "text": "You said: hello",
  "timestamp": "2018-06-22T06:46:01.483Z",
  "type": "message"
}
{% endhighlight %}

<img src="{{site.url}}/assets/posts/2018-06-22-Test-Botbuilder/screenshot6-receive-message-detail.png" />


