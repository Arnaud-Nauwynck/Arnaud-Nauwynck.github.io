---
layout: post
title:  "Discovering Jekyll Static Page Generator & Github.io  (Jekyll) web page hosting"
date:   2014-10-27 00:20:18
categories: jekyll
tags: jekyll
---

I was searching a static blog page generator... <br/>
with google search for "blog static", result rank number 1 is Jekyll (as of 2014-10-27)
<br/>

<img src="{{site.url}}/assets/posts/2014-10-27-jekyll/google-blog-static.png"/>


so here we go: [http://jekyllrb.com]
<br/>


installation is straightforward
{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

... but it worked only under root account !
... And I also needed to install ruby-dev   (error when only ruby is installed !) 
 

creating the "Hello Word" site is also simple and easy (as for every others "Hello World") 

{% highlight bash %}
$ jekyll new .
$ jekyll serve
$ echo ... open your browser to http://localhost:4000
{% endhighlight %}

Open your browser http://localhost:4000, and admire your "Hello Blog World" web site.

<img src="{{site.url}}/assets/posts/2014-10-27-jekyll/jekyll-new-localhost-4000.png"/>

Note that it is regenerated in real-time when you edit any input file in Jekyll

(except the _config.yaml itself, which requires a stop "Ctrl C" + restart "jekyll serve")



As a developper, I recommend using a SCM for everything you do ... so it becomes

{% highlight bash %}
$ git init .   
# ... or git clone git@github.com:username/username.github.io.git

$ echo _site > .gitignore
# ... you might have nothing all this dir "./_site" now contains all your generated static web site!

$ git add .
$ git commit -m "initial commit"
{% endhighlight %}


 

... So now, we have our site and great tools to edit it, but what's next?



<h2>Hosting your Jekyll web site result ... natively supported in github.io</h2>
Then I discovered in Jekyll documentation that one preferred way of publishing blog pages was github.io, and that a user having an account "user", can create a github repository called "user.github.io", and automatically, the master branch of the repository will be exported as a web site in "http://user.github.io"


I have created my repo, cloned it locally, and pushed my index.html containing "Hello World" ... but nothing appeared immediately!
It took hours for my website to be up&running ... now it is: 
<A>http://arnaud-nauwynck.github.io</A>
<img src="{{site.url}}/assets/posts/2014-10-27-jekyll/arnaud-nauwynck.github.io.png"/> 


At first, I didn't know if it was better to put my website jekyll source code in my github repo, or simply put the result generated in directory _site/   ... and in fact, Google support jekyll natively, and show the .gitignored files _site/* when browsing the http://user.gihtub.io instead of the real content of the repo http://gihtub.com/user/user.github.io   !!!

   

