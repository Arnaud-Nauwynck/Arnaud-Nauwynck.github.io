---
layout: post
title:  "Json Jackson Tips &amp; Tricks: Challenge Solving Errors"
date:   2017-01-25 21:50:00
categories: 
tags: json jackson junit
---

<H1>Java + Json = Jackson</H1>

Jackson is the de-facto standard for serializing/deserializing java objects to/from json text.

To use it, simply add it in you maven pm.xml dependencies.
{% highlight xml %}
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.7.1</version>
</dependency>
{% endhighlight %}

This transitively use jackson-core for the pure parser, and jackson-annotation for annotations: 
{% highlight text %}
$ mvn dependency:tree
..
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.7.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.7.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.7.1:compile
..
{% endhighlight %}

You can enrich your Data Transfer Objects (your api jars) with Jackson (and JaxB) annotations, this has no  impacts for users of your jar libraries, who are not obliged to use the parser + databind jar.

<P>
The usage is pretty simple, almost everything is accessible from the facade class ObjectMapper, which can be used like that:

{% highlight java %}
public static final ObjectMapper objectMapper = new ObjectMapper();

public static String toJson(Object obj) {
	ByteArrayOutputStream buffer = new ByteArrayOutputStream();
	try {
		objectMapper.writeValue(buffer, obj);
	} catch (Exception ex) {
		throw new RuntimeException("Failed obj->json", ex);
	}
	String jsonText = buffer.toString();
	return jsonText;
}

public static <T> T fromJson(String json, Class<T> clss) {
	try {
		T res = objectMapper.readValue(json, clss);
		return res;
	} catch (Exception ex) {
		throw new RuntimeException("Failed json->obj", ex);
	}
}
{% endhighlight %}


<H1>Challenge : Fix these JsonMappingException(s)</H1>

I have written a series of JUnit tests, to show some common mistakes on Jackson.
<P>
These tests almost all fails, but some other surprisingly works while they "should" not..
<P>
You will be surprised to see which Tests are OK, and which one are not, just by reading the source-code and trying to understand them.

<P>
The challenge for you is to fix these tests to make them work.


The code is here 
<A href="https://github.com/Arnaud-Nauwynck/test-snippets/blob/master/test-jackson/src/test/java/fr/an/tests/jackson/JacksonTest.java">JacksonTest.java</A>





	