---
layout: post
title:  "Jekyll Edit Markdown Samples"
date:   2014-10-28 21:00:00
categories: jekyll
tags: jekyll
---


<h2>Jekyll file structure</h2>

Jekyll uses the Liquid ( https://github.com/Shopify/liquid/wiki ) templating language to process templates

Jekyll file starts with a triple dash  - - - , to define a Jekyll header

The header part is in Yaml syntax, until closing - - -, then there is the body part

{% highlight text %}
---
layout: post
title: "My Page"
otherVar1: value1
---
<h1>Sample Jekill body file</h1>
my file body text (.md, .html, ...)
goes on several line

... interpreted then by Jekill Liquid template engine
... using
   expanded _layouts/{ { page.layout } }.html layout file
   contextual page header variable  ... example: { { page.title } }
 + contextual parent variable       ... example: { { site.url } } 
 + Liquid tags { %for ...} { %if} ..{ %endif %}{ % endfor %}
...

cf next 
{% endhighlight %}


The body content is itself evaluated by Jekyll using the Liquid template engine, and passing a contextual list of variables=value.

This context inherits parent context values:
page -> parent _config.xml
layout fragment -> parent page -> parent _config.xml 


<h2>Jekyll directory project structure</h2>

files and dirs starting with underscores ("_") are interpreted by Jekyll

typical directory structure:

{% highlight text %}

./_config.yml

./_layouts
./_layouts/default.html
./_layouts/page.html
./_layouts/post.html

./_includes
./_includes/footer.html
./_includes/head.html
./_includes/header.html

./css
./css/main.scss

./_posts
./_posts/2014-10-27-welcome-to-jekyll.markdown
./_posts/2014-10-28-jekyll-edit-markdown-samples.markdown

./assets/image1.png
./assets/image2.png

./data/file1.json
./data/file2.json

./_collection1/file1.json
./_collection1/file2.yaml
./_collection1/file2.csv

./_collection2/file1.json
./_collection2/file2.yaml
./_collection2/file2.csv

./about.md
./index.html
./feed.xml

./.gitignore
./_site/** ... gitignored generated files 
{% endhighlight %}  

  

<h2>Liquid templating engine overview</h2>

Liquid https://github.com/Shopify/liquid  
defines itself as "
Liquid markup language. Safe, customer facing template language for flexible web apps.
http://liquidmarkup.org/"

See documentation : http://docs.shopify.com/themes/liquid-documentation/basics
See video http://www.youtube.com/watch?feature=player_embedded&v=tZLTExLukSg  (which last 6 minutes only)

It is a kind of text pre-processor in Ruby (like Velocity or ThymLeaf in Java, T4 in C#, Angular expression/directives in JavaScript, etc..)

Like AngularJS, it has a mustach-like syntax for variable, and like Velocity, it has begin-tag / end-tag for macros
 

A picture is worth a thousands words:

{% highlight liquid  %}{% raw %} 
<ul id="products">
  {% for product in products %}
   {% if product.enabled %}
    <li>
      <h2>{{ product.name }}</h2>
      Only {{ product.price | price }}
      {{ product.description | prettyprint | paragraph }}
    </li>
   {% endif %}
  {% endfor %}
</ul>
{% endraw %}{% endhighlight %}



More in details: 

* replaces variables by their value
{% highlight liquid  %}{% raw %} {{ variable }} {% endraw %}{% endhighlight %}

* filter expression and format
{% highlight liquid  %}{% raw %} {{ variable | filter1 | filter2 | formatter }} {% endraw %}{% endhighlight %}


* execute tag macro 
{% highlight liquid  %}{% raw %}
{% macro arg1 arg2.. %}
body text
{% endmacro %}
{% endraw %}{% endhighlight %}

Tag macro can be developped as plugin and added


* loops tag

{% highlight liquid %}{% raw %}
{% for product in products %}
  .. {{ product.name }}
{% endfor %}
{% endraw %}{% endhighlight %}

* if tag 
{% highlight liquid  %}{% raw %}
{% if product.enabled %}
  .. {{ product.name }}
{% endif %}
{% endraw %}{% endhighlight %}


* Include a file content, from _includes/subdir/file
{% highlight liquid %}
{% raw %}
{% include subdir/file %}
{% endraw %}
{% endhighlight %}

( notice: work only with path as variable:  
{% highlight liquid %}{% raw %}
{% include {{pathVar}} %}
{% endraw %}{% endhighlight %}

* Include a file content, relative to the current file : 
{% highlight liquid %}{% raw %}
{% include_relative subdir/file %}
{% endraw %}{% endhighlight %}



<h2>Jekyll > Liquid > Pygments : adding code snippets with syntax hilight in Jekyll pages</h2>

see |http://jekyllrb.com/docs/templates/|

Jekyll uses python Pygments librairy: http://pygments.org/

install using   
{% highlight bash %}
$ sudo apt-get install python-pip
$ sudo pip install Pygments
{% endhighlight %}

and also edit your Jekyll _config.yaml to add 
{% highlight yaml %}
highlighter: pygments
{% endhighlight %}


Pygments has itself tons of supported lexer for langages:
http://pygments.org/languages/

( detailed list: http://pygments.org/docs/lexers/ )



* Java
{% raw  %}
{% highlight java %}
public class Foo implements IFoo {
   @Arg(name="test")
   private boolean test;

   public static void main(String[] args) {
   }
}
{% endhighlight %}
{% endraw %}

{% highlight java %}
public class Foo implements IFoo {
   @Arg(name="test")
   private boolean test;

   public static void main(String[] args) {
   }
}
{% endhighlight %}

* Ruby  (Liquid is itself written in Ruby... so cited here)
{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}


* Xml (with tons of &amp;lt;tag/&amp;gt; for escaping &lt;tag/> in rendered html...)
{% highlight xml %}
<tagA attr="1">
	<tagB/>
	&c;
</tagA>
{% endhighlight %}


* Html (with tons of &amp;lt;tag/&amp;gt; for escaping &lt;tag/> in rendered html...):
{% highlight html %}
<div>
   <A href="mylink.png">lylink.png</A>
</div>
{% endhighlight %}


* Liquid 

{% raw %}
<pre><code class="language-liquid" data-lang="liquid"><span class="p">{%</span><span class="w"> </span><span class="nt">highlight</span><span class="w"> </span>liquid<span class="w"> </span><span class="p">%}</span>
<span class="p">{%</span><span class="w"> </span><span class="nt">raw</span><span class="w"> </span><span class="p">%}</span>

... {%if %} bla bla {{ var }} {% endif %}

{<span/>% endraw %}
{% endhighlight %}</code></pre>
{% endraw %}

This is a Meta-headache : Formating Liquid text to escape from Liquid code... (containing {% raw %}{% raw %}{% endraw %} ...)  


* Liquid Blog explained in blog ... Meta-Meta headache (for Advanced curious developpers only) 


Note: 
You may have noticed that to write it myself in this blog (Meta-Meta-Headake), I should myself double the wrapping with highlight/raw ../endraw/endhighlight !!! 
But I also had to cheat by adding a space (invisible &lt;span/> in html)  between { and % to disable the first endraw that was doubled ! 
Then finally, I took up the intermediate generated html, copy&pasted it in this blog by wrapping it by raw..endraw !! 


{% highlight liquid %}
{% raw %}

{% highlight liquid %}
{% raw %}

{% highlight liquid %}
{% raw %}

... {%if %} bla bla {{ var }} {% endif %}

{<span/>% endraw %} // <== Meta-Meta-Headake : cheating by adding an empty space or span  
{% endhighlight %}

{ % endraw %}
{% endhighlight %}

{% endraw %}
{% endhighlight %}

