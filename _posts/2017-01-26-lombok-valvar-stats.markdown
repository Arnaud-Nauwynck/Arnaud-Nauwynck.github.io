---
layout: post
title:  "Lomboker Eclipse Plugin (Day 3)"
date:   2017-01-26 20:50:00
categories: 
tags: eclipse pde jdt lombok refactoring
---

<H1>Result of applying Lomboker to a java project : PlantUML</H1>


PlantUML is a java project (chosen randomly while reading doc from the excellent JHipster project).
It has source-code, and is simple and pure java-based, with simple pom.xml

Very basic stats on the project : 
{% highlight shell %}
echo Number of java files: $(find . -name \*.java | wc -l )
echo Number of lines of code: $(find . -name \*.java -exec cat {} \; | wc -l )
echo Number of bytes of code: $(find . -name \*.java -exec cat {} \; | wc -c )

Number of java files: 2558
Number of lines of code: 392 819
Number of bytes of code: 14 558 590
{% endhighlight %}

After the lomboker transformation (and without counting "import lombok" lines), we got: 
{% highlight shell %}
echo Number of lines of code: $(find . -name \*.java -exec grep -v 'import lombok' {} \; | wc -l)
echo Number of bytes of code: $(find . -name \*.java -exec grep -v 'import lombok' {} \; | wc -c)

Number of lines of code: 391 491
Number of bytes of code: 14 517 252
{% endhighlight %}

So basically, we have<BR/>
2819 - 1491 = 1328 lines less  (eventhough @Getter and @Setter are written on separated lines)<BR/>
58590 - 17252 = 41338 bytes less<BR/>


<P>
To check that it still compiles the same: 

I have added
{% highlight xml %}
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.16.12</version>
	<scope>provided</scope>
</dependency>
{% endhighlight %}

Then I have added file support for lombok experimental "var" (only "val" is built-in yet):
{% highlight text %}
lombok.var.flagUsage = ALLOW
{% endhighlight %}

Then I have compiled it with "mvn clean install" ... 

<P>
Unfortunatly, I got 3 compile errors with the transformation:
<ul>
<li>'val' is not compatible with array initializer expressions</li>
<li> Cannot use 'var' here because initializer expression does not have a representable type: Type cannot be resolved</li>
</ul>
The first is really a bug in the transformer, which would be easy to fix.
It occured only in a code 
{% highlight java %}
float[] style = { dash1, dash2 };          // before lombok
val style = { dash1, dash2 };              // with transformation BUG
val style = new float[] { dash1, dash2 };  // fixed
{% endhighlight %}

The second occured here, with array dimension written on local variable name (C-style) instead of Type:
{% highlight java %}
Point2DInt result[] = new Point2DInt[nb];  // before lombok
val result[] = new Point2DInt[nb];         // with transformation BUG
val result = new Point2DInt[nb];           // fixed
{% endhighlight %}

The second is because the old code was using the identifier string "var", which is now a reserved word for lombok, to be replaced for example with "v". 

{% highlight text %}
[ERROR] src/net/sourceforge/plantuml/geom/Box.java:[121,6] error: Cannot use 'var' here because initializer expression does not have a representable type: Type cannot be resolved
[ERROR] /mnt/a_1tera2/homeData/arnaud/downloadTools/UML/plantuml/plantuml-8054/src/org/stathissideris/ascii2image/text/CellSet.java:[648,7] error: Cannot use 'var' here because initializer expression does not have a representable type: Type cannot be resolved
[ERROR] src/net/sourceforge/plantuml/ugraphic/g2d/DriverLineG2d.java:[75,7] error: 'val' is not compatible with array initializer expressions. Use the full form (new int[] { ... } instead of just { ... })
[ERROR] error: Lombok visitor handler class lombok.javac.handlers.HandleVal failed: java.lang.NullPointerException
[ERROR] src/net/sourceforge/plantuml/jasic/Jasic.java:[143,12] error: 'var' is not compatible with array initializer expressions. Use the full form (new int[] { ... } instead of just { ... })
{% endhighlight %}



<H1>Horror Stories using experimental "lombok.experimental.var" and Eclipse</H1>

I got ~ 1500 ERRORS while trying to compile in Eclipse, and waste several hours trying to fix them.<BR/>
It is mostly because the support of "lombok.experimental.var" is VERY VERY BAD in Eclipse, worse than in pure (maven-)java-lombok (which is also bugged).

<P>
The problem is that the compilation in pure maven does not gives you the exact location of the internal crash ... And in Eclipse, you have thousands error, much more than in command line. But at least you can fix them because you know the file+line number for each error, so you can manually revert it.

<P>
In 99% of the cases, var can be substituted with "val" ... and you should do!
For that, it is quite easy to detect from the Eclipse refactoring tool where a local variable is "effective final". I have added these automated detection in the refactoring, but there are still many internal errors for "val" also !!


<H1>Horror Stories using "lombok.val" in Eclipse</H1>

The support of "val" in Eclipse is also full of BUGS, much WORSE than native (maven-)javac-lombok.
It is really surprising that the type inference is so bad. 

<P>
I have noticed for example that double affectation are sometimes not typed-inferred correctly in Eclipse, in particular with template types !!

{% highlight java %}
val ls1 = new ArrayList<Integer>(); // <= ok, type-inference works
val ls2 = ls1;   // <= sometimes, type-inference is broken in Eclipse ?!! 
                 //   .. but ok in native javac-lombok
{% endhighlight %}




<H1>Horror Stories : internal compiler errors on "lombok.val" compiling with built-in javac-Lombok</H1>


However, some "lombok.val" also cause internal error, even in built-in lombok launched from maven command line
  
{% highlight text %}
Exception while resolving: NODE LOCAL (class com.sun.tools.javac.tree.JCTree$JCVariableDecl) @val()
final ___Lombok_VAL_Attrib__ result = new UPath()
java.lang.NullPointerException
	at com.sun.tools.javac.comp.Check$Validator.visitTypeIdent(Check.java:1331)
	at com.sun.tools.javac.tree.JCTree$JCPrimitiveTypeTree.accept(JCTree.java:2078)
	at com.sun.tools.javac.comp.Check$Validator.validateTree(Check.java:1350)
	at com.sun.tools.javac.comp.Check$Validator.visitTypeArray(Check.java:1245)
	at com.sun.tools.javac.tree.JCTree$JCArrayTypeTree.accept(JCTree.java:2104)
	at com.sun.tools.javac.comp.Check$Validator.validateTree(Check.java:1350)

[INFO] -------------------------------------------------------------
[ERROR] COMPILATION ERROR : 
[INFO] -------------------------------------------------------------
[ERROR] error: Lombok visitor handler class lombok.javac.handlers.HandleVal failed: java.lang.NullPointerException
[ERROR] error: Lombok visitor handler class lombok.javac.handlers.HandleVal failed: java.lang.NullPointerException
[INFO] 2 errors 
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
{% endhighlight %}

The problem is that there is absolutely NO clue of where the problem occurs!! no file or line number !


<H1>Conclusion - Use It .. but not everywhere</H1>

Basically, The refactoring is NOT applicable as a fully automated tool ...<BR/> 
You have to manually undo some of the automated refactoring because they cause errors both in maven and in Eclipse. <BR/>
You can still retain 90% of these automated refactorings, but it will take you few hours to select them.


