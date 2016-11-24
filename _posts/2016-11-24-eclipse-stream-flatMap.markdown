---
layout: post
title:  "Stream-map-flatMap .. bad type inference in Eclipse Neon"
date:   2016-11-18 09:28:00
categories: 
tags: eclipse jdk8 stream
---

<H1>Stream-map-flatMap  ... complexe lambda code for doing simple cartesian product</H1>


Here is a simple cartesian product
{% highlight java %}
	@Test public void test2() {
		List<Integer> ls1 = range(1,2);
		List<Integer> ls2 = range(3,4);
		List<String> basicRes = new ArrayList<>();
		for(int x1 : ls1) {
			for(int x2: ls2) {
				basicRes.add(x1 + "-" + x2);
			}
		}

		List<String> expectedRes = Arrays.asList("1-3", "1-4", "2-3", "2-4");
		Assert.assertEquals(expectedRes, res);
	}

	private static List<Integer> range(int start, int end) {
		List<Integer> res = new ArrayList<>();
		for(int i = start; i <= end; i++) {
			res.add(i);
		}
		return res;
	}
{% endhiglight %}

Using Jdk8 Lambda ... you can do much more complex (but not more compact/readable..)

This-one is still readable..
{% highlight java %}
		List<String> foreachRes = new ArrayList<>();
		ls1.forEach(x1 -> {
			ls2.forEach(x2 -> {
				foreachRes.add(x1 + "-" + x2);
			});
		});
		Assert.assertEquals(expectedRes, foreachRes);
{% endhiglight %}

Then this one:
{% highlight java %}
		List<Integer> ls1 = range(1,2);
		List<Integer> ls2 = range(3,4);
		List<String> res = ls1.stream().flatMap(x1 -> 
			ls2.stream().map(x2 -> 
					x1 + "-" + x2))
				.collect(Collectors.toList());
		Assert.assertEquals(expectedRes, foreachRes);
{% endhighlight %}

Also the non-indented version, make it a good candidate for Obfuscated Code Contest (JCC as CCC ??)
{% highlight java %}
		List<String> res = ls1.stream().flatMap(x1 -> ls2.stream().map(x2 -> x1 + "-" + x2)).collect(Collectors.toList());
{% endhighlight %}
		

As far, as Good


<H1>Little more ... Eclipse Bug in type inference</H1>

Doing the same with 3 cartesian product instead of 2 ... it compiles in maven/javac but does not compile anymore in Eclipse !!

{% highlight java %}
	List<String> res = ls1.stream().flatMap(x1 -> 
		ls2.stream().flatMap(x2 -> 
			ls3.stream().map(x3 -> 
				"" + x1 + "-" + x2 + "-" + x3)))
			.collect(Collectors.toList());
{% endhighlight %}

Eclipse says Compile Error: 			
"Type mismatch: cannot convert from List<Object> to List<String>"


This bug is already referenced in eclipse Bugzilla.
For example here <A href="https://bugs.eclipse.org/bugs/show_bug.cgi?id=502158">Eclipse Bugzilla #502158</A>
 
A workaround is pretty easy, just add an explicit cast to the map() argument:

{% highlight java %}
	List<String> castRes = ls1.stream().flatMap((Function<Integer,Stream<String>>) x1 -> 
			ls2.stream().flatMap(x2 -> 
				ls3.stream().map(x3 -> 
					"" + x1 + "-" + x2 + "-" + x3)))
				.collect(Collectors.toList());
{% endhighlight %}



You can find this source code on my github here:
<A href="https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-map-flatMap">https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-map-flatMap</A>

				
