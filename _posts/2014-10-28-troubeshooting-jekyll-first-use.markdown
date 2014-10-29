---
layout: post
title:  "Troubleshooting Jekyll errors on first use"
date:   2014-10-28 23:00:00
categories: jekyll
tags: jekyll
---
<h1>Troubles shooting error encountered on my first use of Jekyll page editing</h1>


<h2>Encountered ERROR 1 : on error ... server may stopped and page are not generated any more</h2>

Also note that in case of error in a Jekyll file, the server will either crash or just log error message, and your web site is not generated anymore in real-time.

In this case, you are stuck on the last version that used to compile !!!  
It occured to me at the first use, and I was a bit confused on what was going one ... (I had put .png file in the _posts/ dir, and Jekyll complained ) 

It is a pitty that there is no warning message in the rendered html, so you could notice by browing the page that this one is not any more generated !! 

The worst is that Jekyll by default doesn't even log at which line number the problem occured, but you only knows the file having errors, or your see a helpfull message to restart with --trace args !!

{% highlight bash %}
jekyll build --trace 
{% endhighlight %}



<h2>Encountered ERROR 2 : Don't even try to browse from local generated file file:/// ... /_site/index.html </h2>

The look & feel is completely broken ... the CSS resource is not found anymore !

<img src="{{site.url}}/assets/posts/2014-10-27-jekyll/jekyll-error1-static-file-index.png" />

you could check that the index.html contains
{% highlight html %}
<link rel="stylesheet" href="/css/main.css">
{% endhighlight html %}

But however, this line without the absolute "/" would work (but don't edit a file that is generated, you might loose your time !)
{% highlight html %}
<link rel="stylesheet" href="css/main.css">
{% endhighlight html %}

As a prefered optin, use "http://localhost/4000", and pretend that the tool will indeed generate "static html page" that's work, but only when deployed on the web server configured as specified in your config.yaml "site.url: http://...." 

  
<h2>Adding an image</h2>

<h3>Encountered ERROR 3 : adding the binary .png file ... under /asset sub dir </h3>
While writing my first Jekyll page for this post, I encountered this problem: How to include a picture?... (I didn't think it could be a challenge, nor a problem, as I am very acustomed to Confluence wiki) 


First I began to copy my image.png file near to my _posts/yyy-mm-dd-mypost.md file  ... ERROR!

Then immediately after, I received my first error in the console:  

{% highlight text %}
      Regenerating: 1 files at 2014-10-27 01:26:05 Error reading file /home/username/jekyll-test/_posts/2014-10-27-jekyll/img.png: invalid byte sequence in UTF-8 
...error:
             Error: invalid byte sequence in UTF-8
             Error: Run jekyll build --trace for more information.
{% endhighlight text %}

Apparently, this is because Jekyll fails to interpret the header marker expected at the beginning of the file : - - -, but it gets a BOM marker, then a png binary encoding.  

Instead, you have to put your file in a special directory under "./assets/"

<h3>syntax for displaying image in markdown ? ... as html &lt;img src=".."/> </h3>
I fell a bit lost with the markup langage, because it lacks documentation... I thought it would be like a rich confluence or Jive application, to hav every rich markdown tags ... but apparently not! There are still a lot of markdown, but it is not a Wiki at all ... It is a general purpose templating engine, for layout, and "programming" your rendering with data.

Just to include an image, and higlight some text... I realized that typing html directly inside the markup langage could work !!
so typing  

<h3>Encountered ERROR 4 :  404 file not found   for image ... site.url != localhost:4000</h3>

It is recommended in documentation to write image source using 
{% highlight Liquid %}
{% raw %}
  "{{ site.url }}/asset/sub-dir/your-image.png"
{% endraw %}
{% endhighlight %}
  
But url source of image are not working as expected ...
Check the result URL with "image > Right Click > Copy Image Location" or from your favorite html Web Console debugger.
You can see that Jekyll has replaced your target "site.url" by the real remote website "http://username.github.io" instead of "http://localhost:4000", and you have not uploaded the .png file yet !


So 2 possibilities: 1/ upload the image right know...  or 2/ ...  

If you simply replace by 
{% highlight Liquid %}
{% raw %}
  "/asset/sub-dir/your-image.png"
{% endraw %}
{% endhighlight %}

Then it seems to works fine ... 

But consider it as a temporary workwaround that works, and might not work anymore when your real web site will be deployed to your web hosting provider. 

Indeed, your base url might not be the root of the remote site "http://host:port/", but maybe "http://host:port/subapp/subcontext/yourRootWevSite"  !! 

So your picture would be uploaded to ""http://host:port/a/b/yourRootSite/asset/sub-dir/your-image.png", but eroneously referenced as "http://host:port/yourRootWevSite/asset/sub-dir/your-image.png"

In your case, as a temporary workaround, re-edit your _config.yaml configuration file, 
and re-edit (but do not commit) 

{% highlight Yaml %}
# ** temporary ***  DO NOT COMMIT:
site.url: http://localhost:4000
# *** use this real value instead !
site.url: http://username.github.io
{% endhighlight %}

Once your files will have been uploaded to your target web site, you would not requires this temporary hack anymore


<h3>Encountered ERROR 5 :  404 file not found   for target web site image http://username.github.io/ ... ?? </h3>

unresolved yet the second day I used Jekyll while writing this post !

