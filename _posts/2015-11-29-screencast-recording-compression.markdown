---
layout: post
title:  "Screencast Recording & Dedicated Computer Graphics Video Compression"
date:   2015-11-28 23:00:00
categories: 
tags: video compression screencast
---

<h1> Previous Post: Live Coding with Video Recording </h1>
 
This POST started 10 days ago with a Lived Coding recording.<BR/>
See this previous post: <A href="{{site.url}}/2015/11/21/coding-exercise.html">Live Coding - Java Coding Exercise: a tool for synchronizing 2 dirs (backup) using SHA comparisons</A>



<h1>Live Coding Video Recording</h1>

For preparing 30 minutes of live-coding, it actually took me more than 10 hours of work: 
<ul>
<li> 1 hour to peek tools in linux to do video screen capture, and key-press/mouse-click display</li>
<li> 1 hour to code (first recording)</li>
<li> 1  hour to convert the video file to a compressed but standard file format</li>
<li> 3  hours to write this post on github.io</li>
<li> 4 hours to repeat faster code recording</li>
</ul>

And because I am curious about everything in computer sciences, I have spent the last few days searching for tools, documentations, research papers on the subject of "screencast video compression" !
  

At first, the longest was to find the tool to show the key-press/mouse-clicks and to do a video capture of all.
<BR/>
I tend to avoid mouse-clicking wherever possible, because it is just slower... and more harmfull for your "carpal tunnel" (you might get surgical operation on your hand sometimes otherwise...). see <a href="https://en.wikipedia.org/wiki/Carpal_tunnel">wikipedia: carpal tunnel</a>

For displaying key-press and mouse-click, I have chosen to install and launched  "key-mon" tool

<img src="{{site.url}}/assets/posts/2015-11-21-coding-exercise/key-mon-screenshot.png" />

When watching at the video, you can see that I hardly ever use the mouse, but I use few eclipse shortcuts here and there.

I should also (but did not) have configured "MouseFeed", which is and Eclipse plugin that shows you when you mouse-click instead of using shortcuts. This plugin is very cool to learn important shortcuts, and force you improving your Eclipse efficiency.
<BR/>
For example, for fast switching from java source code editer to corresponding Test code, I use "CTRL+J" which is defined in an eclipse plugin called "MoreUnit".


<BR/>

Finally, here we are, I recorder me coding for 30 minutes, converted the file to .MKV file format (=4.6 Mo) , and here it is:

   <A href="{{site.url}}/assets/posts/2015-11-21-coding-exercise/live-coding.mkv">live-coding.mkv</A>  


For training myself, I might redo and redo again this exercise:
<ul>
<li> try to reduce time from 1 hour to 45 minutes ?</li>
<li> try to reduse code size from 200 lines to 150 ? (using shorter code style, java 8 lambda, ..)</li>
<li> try not to use the mouse at all (from ~ 20 clicks to 0?)</li>
<li> try to do do less typo errors (edit then delete 1 char)</li>
<li> try to do less do-write and undo-write of codes (type directly the final code, when choosing local variables names, method names, if-tehn-else / for / try-catch structures) </li>
<li> less hesitations </li>
<li> always write Tests code first before Code (TDD=Test Driven Development, Agile practise), </li>
<li> etc. </li>
</ul>

An interesting measurment would be to see the number of "key stroke", and the number of "Delete" or "Up arrow" pressed.<BR/>
Even better, I would like to see the speed of "key strokes", as a function of time, to see when coding is slowing.   



<h2>More on screen capture video compression file formats</h2>  


To do screencast video capture, I have searched a tool for my Debian Linux workstation...
After googling it, there was a dozen of candidates. <BR/>

Strangely (or not!!), most of them dit not work at all (crashed when launched !!!), where unusable, or very un-intuitive to use.
I am a busy guy, I don't want to read 600 pages of manual for that kind of task.
I repeated 4 times:
<ul>
<li> get a soft candidate from google</li>
<li> apt-get install ..</li>
<li> launch it </li>
<li> click on the big green button if found it, and if it seems to work</li>
<li> trash it, and get another one</li>
</li>
</ul>
<BR/>
  
I finally choose a very simple and efficient one from github (using java source-code for it!)
It is called "screen-recorder", and there is also a tool "screen-player", and "screen-converter"

<BR/>

<a href="https://github.com/bspkrs/java-screen-recorder.git">https://github.com/bspkrs/java-screen-recorder.git</a>

<BR/>

For recording my video, I launched "screen-recorder.sh"  which internally was  
{% highlight text %}
java -cp screen-recorder.jar com.wet.wired.jsr.recorder.JRecorder
{% endhighlight text %}
<BR/>

It worked like a charm. Presenting me a minimalistic and efficient app with only a "start" button.
Internally, it writes to a temporary file... And once started, you have a button "Stop", and "Save" (which rename the temp file to the destination you ask).
<BR/>

The huge advantage of it is that it writes the video file on the fly in a proprietary(!) file format ".cap", but it is very very efficiently, using an internal GZIP stream of bitmaps... simple but efficient!
Then you can replay it,  either normal speed, or fast speed... and convert it to ".mov" standard format.
<BR/>

For 28mn of video capture of 1900x1080,  the .cap file is 31Mo ONLY  ... while the converted .mov file is 518 Go, and the .MKV file is 30% smaller: 22 Mo
<BR/>
(For another recording lasting 1 hour, I got bigger files: 36M of .CAP, 13M of .MKV, and 1.5 Go of .MOV !) 
<BR/>

{% highlight text %}
$ ls -lh
  22M  live-coding.h264
  22M  live-coding.mkv
  22M  live-coding.m4v
  23M  live-coding.webm
  31M  live-coding.cap
  36M  live-coding.mp4
  65M  live-coding.wmv
  65M  live-coding.asf
  69M  live-coding.avi
  86M  live-coding.flv
  86M  live-coding.swf
 518M  live-coding.mov
  25G  live-coding.yuv


{% endhighlight text %}

This is the main reason why I put on this website the .cap file ... if you are interrested in seing the video!
Someday, I might find a converter to a much better compressed and standard file format. 



To play the video in the recorded proprietary .CAP format ... I have launched
{% highlight text %}
java -cp java-screen-player.jar com.wet.wired.jsr.player.JPlayer live-coding.cap
{% endhighlight text %}
<BR/>

Then to convert it to .MOV, I launched
{% highlight text %}
java -cp screen-converter.jar:lib/jmf.jar  com.wet.wired.jsr.converter.RecordingConverter live-coding.cap
{% endhighlight text %}
<BR/>
Unfortunatly, .MOV file is huge, something like 20x bigger than .CAP file.

So, I converted it to a more efficient and standard format: .MKV, using "handbrake" UI conversion tool.



MOV file compression algorithm is based on lossy JPEG (Fourier cosinus transformationon using float numbers and roundings), so it is clearly not adpated to compress RGB integers image generated from computer screen (using only horizontal/vertical rectangles, Fonts display and few other vector drawing primitives)
<BR/>
   
Basically, a video of someone typing text in an editor is a video with a very LOW BIT RATE information between each frame!
We could describe that images are combinations of these low level primitives:
<ul>
<li> Only the few pixels around the cursor are changing from one frame image to the next one.</li>
 The cursor itself may not part of the image in the screen capture recorder, it could be computed and redrawn above</li> 
<li> around the cursor change, image block are often shifted to the left or to the right</li>
<li> most changing blocks in the image are often copy&paste image blocks from already played frames (example: click => change color, then revert to previous color)</li>
<li> many image fragments comes from graphical drawing primitives: line, rectangle, text fonts, icons</li> 
</ul>
<BR/>

Screen capture and teleconferencing to share a screen is SO MUCH common, it is used in Citrix applications, YouTube tutorials, Team work programming, Chats ...
There should be extremely efficient algorithms to use the knowledge of image and video structures, and compress them much more efficiently than a regular camera based film (where .MOV is efficient).
    
<BR/>

I did few searches with Google (and DuckDuckGo), and I found that lossless .MKV, .WMV, .VP9 might be interresting candidates.

My first try was to convert the .mov to .mkv, which I did using "handbrake" application.<BR/>

The result I got confirmed my thoughts... .MKT file is 3x to 10x smaller than .cap file !

It is a pity than the project java-screen-converter does not convert directly to .MKV file format, but produces hugly intermediate .MOV  file (which is huge, and loss quality with JPEG computations) ... then recompressing it with .MKV is not optimal... it should be even much better with a direct .CAP to .MKV conversion.   
 
Someone interrested to contribute to <a href="https://github.com/bspkrs/java-screen-recorder.git">github: java-screen-recorder.git</a> for doing this ?



<H1> Yet Another Github project : screencast-compression-tool.git</H1>

After playing/refactoring a little with https://github.com/bspkrs/java-screen-recorder.git,
I felt the need to do more image analysis algorithm on it.
I also wanted to have better video file format conversion or tool supports, so I have tested humble.io project. 
<BR/>

I have created a new repository, not cloning the previous one (I have reused only 2 files from java-screen-recorder, but restarting other all from the beginning).
<BR/>

<A href="https://github.com/Arnaud-Nauwynck/screencast-compression-tool">https://github.com/Arnaud-Nauwynck/screencast-compression-tool</A>


more things to come ...

