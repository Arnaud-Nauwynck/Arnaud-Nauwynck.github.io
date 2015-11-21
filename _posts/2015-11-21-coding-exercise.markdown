---
layout: post
title:  "Live Coding - Java Coding Exercise: a tool for synchronizing 2 dirs (backup) using SHA comparisons"
date:   2015-11-21 17:00:00
categories: 
tags: live-coding exercise video backup
---



Link of the Live Coding video lasting 1 hour  
 <A href="{{site.url}}/assets/posts/2015-11-21-coding-exercise/live-coding.mkt">live-coding.mkt</A>  



<h1>Introduction : Live Coding - Student Exercise Correction</h1>

I asked my students to code in Java a "File Synchronisation Tool", using SHA-1 comparisons.

Basically, after the course on java.util.List, java.util.HashMap and java.io.* , I wanted to give them an exercise to practise what they had learned.<BR/> 

Students level in java are varying from begniners to intermediate. 
They are 20-21 years old, and working part-time in a company, and part time at university (3 years after french graduate "baccalaureat")

They had approximatly 1 month to give me their work, as an eclipse source code project, and including tests.


<h1>Description of the Synchronizing Tool</h1>

The "File Synchronisation Tool" must compare recursively 2 directories (aka sourceDir and destinationDir), and compute the list of copy/removal of files to get the destination directory be in sync with the source directory.

For example, suppose you want to compare source directory "/data/dir1" to destination directory "/data/dir1.backup", 
and you have some files added in source "/data/dir1/a/foo-new.txt", modified "/data/dir1/a/bar-modified.txt", and deleted "/data/dir1/a/baz-deleted.txt" (does not exists in source dir, only in "/data/dir1.backup/a/baz-deleted.txt")   

The tool ouput should be a list of commands to synchronize from source to dir:
{% highlight text %}
cp '/data/dir1/a/foo-new.txt'       '/data/dir1.backup/a/foo-new.txt'
cp '/data/dir1/a/bar-modified.txt'  '/data/dir1.backup/a/bar-modified.txt'
rm                                  '/data/dir1.backup/a/baz-deleted.txt'
{% endhighlight text %}



<BR/>
The synchronisation tool must work in a "3-way" algorithm style, by first computing all SHA-1 signatures of source files, then SHA-1 signatures of destination files... then find matches and mismatches between source and destinations.
It could(should) also warn on duplicate file contents found in source, so possibly use symlink on destination instead of wasting disk copy.

It was explicitly stated that the internal datastructure to use was a Map of List, whose key is the SHA-1 of file content, and values are the related paths of all files having the SHA.

{% highlight java %}
Map<String/*SHA*/, List<String/*Path*/>> sha2RelativePaths = 
   new HashMap<String, List<String>>();   
{% endhighlight java %}
 

As as example, suppose you have a directory with the following content
{% highlight text %}
/
/a/foo.txt             with SHA1= a1b2c...  512 hexa chars code
/a/foo-copy.txt                 = (idem)
/a/b/bar.txt                SHA1= e7f8a...
{% endhighlight text %}

then the data signature of theses files is a Map with the following content:
{% highlight text %}

{
   { key1: "a1b2c...",  value: List[ "/a/foo.txt",  "/a/foo-copy.txt"  ] }
   { key2: "e7f8a...",  value: List[ "/a/b/bar.txt" ]
}
{% endhighlight text %}
 

<h1>Correction : Code Difficulty and Coding Speed Performance</h1>

The code is pretty simple, as the following shell command on the correction source code shows it:

{% highlight text %}
$ wc -l ./src/fr/iut/tp/backupsha/BackupSHAMain.java ./src/fr/iut/tp/backupsha/BackupSHAMainTest.java 
 134 ./src/fr/iut/tp/backupsha/BackupSHAMain.java
  54 ./src/fr/iut/tp/backupsha/BackupSHAMainTest.java
 188 total
{% endhighlight text %}



The main part takes ~100 lines of code, in a total of less than 200 lines of codes, JUnit tests included.

You might wonder that it is not so much, and it should take ~ 30 seconds per line to code, so ~1 hour to get it...<BR/> 
Indeed, it took me ~1 hour to code it.<BR/>
Being fast is possible only because you repeat something you already knows, or knows how to do it... It is just the repetition and training that makes you becoming fast, like every athletes.


It should take you ~1 hour to code, if you are also an experimented developper, but for my students it was their first coding of such tool ... I received by email the first projects from my students (by asking several times) after more than 15 days, but I think they have spend only few (~3 ?) days on it.<BR/>
Unfortunatly, only 4 out of 18 students had done something, and the results submitted were FAR FAR from expected, both in term of results, performance optimisations and code readability/quality !!<BR/>

I had to reply to them several times, to explain what was wrong in their programs<BR/>
Basically, the forgot to do a recursive scan of the directory, they only listed 1 level of directory. Then they failed to understand that the SHA1 was computed on the file content, that it was to converted to hexa string (or whatever array class in java implementing equals() and hashCode() ), and that they had handle the possibility of file content duplicated several times ...  




<h1>Correction - Live Coding Video</h1>

For preparing 1hour of live-coding, it actually took me more than 3 hours of work: 
<ul>
<li> 1 hour to peek tools in linux to do video screen capture, and key-press/mouse-click display</li>
<li> 1 hour to code</li>
<li> 1  hour to write this post on github.io</li>
</ul>

The longest was to find the tool to show the key-press/mouse-clicks and to do a video capture of all.
I tend to avoid mouse-clicking wherever possible, because it is just slower... and more harmfull for your "carpal tunnel" (you might get surgical operation on your hand sometimes otherwise...). see <a href="https://en.wikipedia.org/wiki/Carpal_tunnel">wikipedia: carpal tunnel</a>

For displaying key-press and mouse-click, I have chosen to install and launched  "key-mon" tool

<img src="key-mn.png" />

I should also (but did not) have configured "MouseFeed", which is and Eclipse plugin that shows you when you mouse-click instead of using shortcuts. This plugin is very cool to learn important shortcut, and force you improving your Eclipse efficiency.

<BR/>

Finally, here we are, I recorder me coding for 1 hour, converted the file to .MKT file format (=13 Mo) , and here it is:

   <A href="{{site.url}}/assets/posts/2015-11-21-coding-exercise/live-coding.mkt">live-coding.mkt</A>  





<h2>More on screen capture video compression file formats</h2>  


To do screen video cpature, I have searched a tool for my Debian Linux workstation...
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
  
I finally choose a very simple and efficient one from github (using java source-code for it!)
It is called "screen-recorder", and there is also a tool "screen-player", and "screen-converter"

<BR/>

<a href="https://github.com/bspkrs/java-screen-recorder.git">https://github.com/bspkrs/java-screen-recorder.git</a>

<BR/>

For recording my video, I launched "screen-recorder.sh"  which internally was  
{% highlight text %}
java -cp screen-recorder.jar com.wet.wired.jsr.recorder.JRecorder
{% endhighlight text %}

It worked like a charm. Presenting me a minimalistic and efficient app with only a "start" button.
Internally, it writes to a temporary file... And once started, you have a button "Stop", and "Save" (which rename the temp file to the destination you ask).
<BR/>

The huge advantage of it is that it writes the video file on the fly in a proprietary(!) file format ".cap", but it is very very efficiently, using an internal GZIP stream of bitmaps... simple but efficient!
Then you can replay it,  either normal speed, or fast speed... and convert it to ".mov" standard format.
<BR/>

For 1 hour of video capture of 1900x1080,  the .cap file is 36Mo ONLY  ... while the converted .mov file is 1.5 Go, and the .MKT file is 3x smaller: 13Mo !!! 
<BR/>

{% highlight text %}
$ ls -lh live*
-rw-r--r-- 1 arnaud arnaud  36M Nov 21 16:40 live-coding.cap
-rw-r--r-- 1 arnaud arnaud  13M Nov 21 22:32 live-coding.mkv
-rw-r--r-- 1 arnaud arnaud 1.5G Nov 21 17:20 live-coding.mov
{% endhighlight text %}

This is the main reason why I put on this website the .cap file ... if you are interrested in seing the video!
Someday, I might find a converter to a much better compressed and standard file format. 



To play the video in the recorded proprietray .CAP format, download the 36Mo <A href="{{site.url}}/assets/posts/2015-11-21-coding-exercise/live-coding.cap">live-coding.cap</A> file, 
and download the <A href="{{site.url}}/assets/posts/2015-11-21-coding-exercise/screen-player.jar">screen-player.jar</A> ... Launch it:

{% highlight text %}
java -cp java-screen-player.jar com.wet.wired.jsr.player.JPlayer live-coding.cap
{% endhighlight text %}
 


It is clear that 36Mo is still a big file, even it is much smaller than .mov file (1.5Go).<BR/>

Mov file compression algorithm is based on lossy JPEG (Fourier consinus transformationon using real number), so it is clearly not adpated to compress RGB integers image generated from computer screen (using only horizontal/vertical rectangles, and Fonts : vector graphics, not raster bitmap)
<BR/>
   
Basically, a video of someone typing text in an editor is a video with a very LOW BIT RATE information between each frame!
We could descrbe that images are combinations of these low level primitives:
<ul>
<li> Only the few pixels around the cursor are changing from one frame image to the next one.</li>
 The cursor itself may not part of the image in the screen capture recorder, it could be computed and redrawn above</li> 
<li> around the cursor change, image block are often shifted to the left or to the right</li>
<li> most changing blocks in the image are often copy&paste image blocks from already played frames (example: click => change color, then revert to previous color)</li>
<li> bitmap data ar epixel, but in fact comes from graphic drawing primitives: line, rectangle, text fonts, icons</li> 
<BR/>

Screen capture and teleconferencing to share a screen is SO MUCH common, it is used in Citrix applications, YouTube tutorials, Team work programming, Chats ...

There should be extremely efficient algorithms to use the knowledge of image and video structures, and compress them much more efficiently than a regular camera based film.
    
<BR/>

I did few searches with Google (and DuckDuckGo), and I found that lossless .MKV, .WMV, .VP9 might be interresting candidates.

My first try was to convert the .mov to .mkt, which I did using "handbrake" application.<BR/>

The result I got confirmed my thoughts... .MKT file is 13mo, so 3x smaller than .cap file !

It is a pity than the project java-screen-converter does not convert directly to .MKT file format, but produces hugly intermediate .MOV  file (which is huge, and loss quality with JPEG computations) ... then recompressing it with .MKT is not optimal... it should be even much better (perhaps less than 10Mo) with a direct .CAP to .MKT conversion.   
 
Someone interrested to contribute to  
<a href="https://github.com/bspkrs/java-screen-recorder.git">github: java-screen-recorder.git</a> for doing this ?



