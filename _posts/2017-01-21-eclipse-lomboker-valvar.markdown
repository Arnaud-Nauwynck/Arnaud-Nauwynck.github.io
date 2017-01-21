---
layout: post
title:  "Lomboker Eclipse Plugin (Day 2)"
date:   2017-01-21 19:50:00
categories: 
tags: eclipse pde jdt lombok refactoring
---

This is the second post on my proof-of-concept "Lomboker Eclipse Plugin"<BR/>

See previous post <A href="{{site.url}}/2017/01/18/eclipse-lomboker.html">here</A> 

<BR/>
Yesterday, I have shown the first post and the plugin to a fiend of mine, and we had this discussion:

<blockquote>
My Friend&gt; is it all Lombok is about?<BR/>

Me&gt; Well, no Lombok also have the amazying val and var features, and much more<BR/>

My Friend&gt; It is not supported in your plugin ?<BR/>

Me&gt; Well, yes it is true that would be easy to do.<BR/>
  &nbsp; But sorry about that, I only started yesterday and I spent just 3 hours on it...<BR/> 
  &nbsp; The longer was to write the post on my github.io<BR/>
   
My Friend&gt; Ah, ok, I see  ... that's not so bad<BR/>
</blockquote>

 
Isn't it so encouraging to do the second step ?
 


<H1>Day 2 : Transforming val and var for local variable and foreach</H1>

On day 2, I worked again to improve this proof-of-concept plugin, and added ~3 more hours of work.

<p>
I have implemented the detections of "SomeLongTypeName localVar = new SomeLongTypeName();", and replacements with lombok "var localVar = new SomeLongTypeName();"

<p>
Isn't it true we ALL java developper are fed up typing things like ??
{% highlight java%}
Map<String,Map<Integer,List<Boolean>>> map = new HashMap<String,Map<Integer,List<Boolean>>>();
Map<Integer,List<Boolean>> elt1 = map.get("key1");
{% endhighlight %}
instead of 
{% highlight java%}
val map = new HashMap<String,Map<Integer,List<Boolean>>>();
val elt1 = map.get("key1");
{% endhighlight %}	

<P>
Of course, Java will continue to evolve (slowly as it always did), and there is some hope with this JEP (<A href="http://openjdk.java.net/jeps/286">http://openjdk.java.net/jeps/286</A>) that it will be built-in in the langage soon:
<BR/>
<A href="http://openjdk.java.net/jeps/286"><img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lomboker/jep286.png" /></A>


<P>
Back to my plugin development, this was pretty easy, but I had to avoid few nasty cases:
<ul>
<li> it is useless to replace already primitive types, and also String type
{% highlight java %}
int i = 10;
// should not replace by
var i = 10; 
{% endhighlight %}
</li>

<li> when using the jdk8 Diamond syntax, I had to preserve the type from the left hand side declaration, to put it to the right hand side!
{% highlight java %}
List<SomeLongTypeName> ls = new ArrayList<>();
// => in Lombok:
var ls = new ArrayList<SomeLongTypeName>();

// this also work..
Map<String,List<SomeLongTypeName>> map = new HashMap<>();
// => in Lombok:
var map = new HashMap<String, List<SomeLongTypeName>>();
{% endhighlight %}

To my opinion, this is a nasty mistake in java8 syntax, it is more logic for typed-inference langage to be like 
{% highlight java %}
List<> ls = new ArrayList<SomeLongTypeName>();
{% endhighlight %}
</li>

<li> final variable must be replaced by "val", whereas modifiable variable must be replaced by "var"</li>
<li> for explicitely "final" variable, the keyword modifier "final" must be removed
{% highlight java %}
final SomeLongTypeName x = new SomeLongTypeName();
// => in Lombok:
val x = new SomeLongTypeName();
// instead of ...   final var x = new SomeLongTypeName(); 
{% endhighlight %}
</li>

<li> I should also detect "effective final" variables (variables that not explicitely marked as final, but bever modified anyway)..  I did not implement it yet, so they still are replaced with lombok "var".</li>
</ul>


<H2>Enough speaking ... Demo</H2>


Now the settings dialog window has 2 checkboxes:
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lomboker/screenshot-Lomboker-params2.png" />
<BR/>

Click on Preview, to see how local variables are transformed<BR/>
(notice the different cases using primitive, final declaration, diamond syntax ..)<BR/>
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lomboker/screenshot-Lomboker-valvar.png" />
<BR/>

And also the same with foreach loops 
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lomboker/screenshot-Lomboker-foreach.png" />
<BR/>


<H1>Want to see how code is simple ?</H1>

Here is an extract of the transformation of foreach "var" ... It is only ~ 50 lines of code only !

{% highlight java%}
	protected void doRefactorUnit_ValVar(CompilationUnit unit) {
		RequiredLombokImports requiredImports = new RequiredLombokImports();
		ASTVisitor vis = new ASTVisitor() {
			@Override
			public boolean visit(EnhancedForStatement node) {
				SingleVariableDeclaration forVarDecl = node.getParameter();
				boolean needReplaceByVal = needReplaceTypeNameToVar(forVarDecl.getType());
				if (needReplaceByVal) {
					// do replace "for(Type varName : ..) .." => "for(var varName : ..) .."
					AST ast = unit.getAST();
					forVarDecl.setType(ast.newSimpleType(ast.newName(LOMBOK_VAL)));
					requiredImports.setUseVal();
				}				
				return super.visit(node);
			}
		};
		unit.accept(vis);
		requiredImports.addImports(unit);
	}

	private static boolean needReplaceTypeNameToVar(Type varDeclType) {
		String varDeclTypeName = MatchASTUtils.matchSimpleTypeName(varDeclType);
		if (varDeclTypeName != null && (varDeclTypeName.equals(LOMBOK_VAL) || varDeclTypeName.equals(LOMBOK_VAR))) {
			return false; // ok, already a lombok type!
		}
		if (varDeclType.isPrimitiveType() || "String".equals(varDeclTypeName)) {
			// do not replace already primitive type int, double, boolean, and also String...  
			return false;
		}
		return true;
	}
	
	private static class RequiredLombokImports {
		private boolean useLombokVal;
		public void setUseVal() {
			useLombokVal = true;
		}
		public void addImports(CompilationUnit unit) {
			if (useLombokVal) {
				JavaASTUtil.addImport(unit, "lombok.val");
			}
		}
	}
{% endhighlight %}
	
	

<H1>Conclusion</H1>


I am ready, and I would be proud to present it at the great Devoxx France 2017 conference.

<p>
Maybe somebody will notice it, and this plugin will be improved and added to the excellent Lombok tool suite.
</p>

<p>
Notice that there was already an official "delomboker" tools, to transform lombok code to plain-old-java code, as required for source-code analyser tools like GWT and Sonar.<BR/>
Now there would be an official "lomboker" tool.
</p>

<p>
For the presentation title, I have chosen "Chérie, J'ai Lomboké les Classes" (french translation of "Honey, I Shrunk the Classes"), which is a reference to the film "Honey, I Shrunk the Kids".
</p>

Reminder, it is open-source, fork it on GitHub <A href="https://github.com/Arnaud-Nauwynck/mytoolbox/tree/master/eclipse-plugins/fr.an.eclipse.tools.lombok">https://github.com/Arnaud-Nauwynck/mytoolbox .. fr.an.eclipse.tools.lombok</A> 



 
