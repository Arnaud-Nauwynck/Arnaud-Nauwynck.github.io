---
layout: post
title:  "Lombokifier Eclipse Plugin"
date:   2017-01-18 23:40:00
categories: 
tags: eclipse pde jdt lombok refactoring
---

I wanted to write a simple eclipse plugin to "Lombokify classes"

<H1>Motivations</H1>

Lombok is a great project.
<BR/>
It allow java developper to write simpler code (small is beautifull), as in C#, Groovy, Kotlin BUT without switching to C#, Groovy, Kotlin...

<A href="https://projectlombok.org/">https://projectlombok.org/</A>

<p>
As as additional motivation, it is a subject I have just proposed to Devoxx France 2017 "Call For Paper".
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lombokify/screenshot-devoxx-cfp.png" />
<BR/>

<BR/>


<H1>Steps for using the magic "Lombokifier eclipse plugin"</H1>

Images worth thousands words.
<BR/>
Here is how to use it.
<BR/>

Select either Java Project(s), Java Package(s) or Java class(es), and right click for refactoring menu action: "recursive Lombokify" 
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lombokify/screenshot-Lombokify-selection.png" />
<BR/>

Choose some settings (currently supports only @Getter / @Setter refactoring)
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lombokify/screenshot-Lombokify-params.png" />
<BR/>

Click on Preview, then Ok
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lombokify/screenshot-Lombokify-preview.png" />
<BR/>

And here you are!
<BR/>
<img src="{{site.url}}/assets/posts/2017-01-19-eclipse-lombokify/screenshot-Lombokify-res.png" />
<BR/>




<H1>How it work</H1>

Come to Devoxx France 2017, if hopefully this talk will be selected.
<BR/>

Source code is extremely simple...
<BR/>
It is open-source, see here: <A href="https://github.com/Arnaud-Nauwynck/mytoolbox/tree/master/eclipse-plugins/fr.an.eclipse.tools.lombok">https://github.com/Arnaud-Nauwynck/mytoolbox .. fr.an.eclipse.tools.lombok</A> 


<BR/>
There is some boiler-plate... not very interresting to detail. (dependency to other plugins fr.an.eclipse.pattern, etc.)
<BR/>
Basically, it allows to handle user selection and recurse on each *.java file, then call the magic Lombokify transformation on each parsed AST, and save it back to java file.
<BR/>

The first code I wrote took me ~2 hours, for detecting getter/setter methods and changing them to @Getter and @Setter.<BR/>
It is very simple, representing ~90 lines of AST tree manipulation in eclipse.<BR/>
Here is and extract of the code:

{% highlight java %}
private static final String LOMBOK_PACKAGE = "lombok";
private static final String ANNOTATION_LOMBOK_GETTER_SIMPLENAME = "Getter";
private static final String ANNOTATION_LOMBOK_SETTER_SIMPLENAME = "Setter";

protected static class FieldGetterSetterDetection {
	final String fieldName;
	VariableDeclarationFragment fieldDecl;
	MethodDeclaration getterDecl;
	MethodDeclaration setterDecl;
	
	public FieldGetterSetterDetection(String fieldName) {
		this.fieldName = fieldName;
	}
}

@Override
protected void doRefactorUnit(CompilationUnit unit, Object refactoringInfoObject) {
	if (useGetterSetter) {
		doRefactorUnit_GetterSetter(unit);
	}
}

protected void doRefactorUnit_GetterSetter(CompilationUnit unit) {
	ASTVisitor vis = new ASTVisitor() {
		@Override
		public boolean visit(TypeDeclaration node) {
			doVisitRefactorGetterSetter(unit, node);
			return super.visit(node);
		}
	};
	unit.accept(vis);
}

private void doVisitRefactorGetterSetter(CompilationUnit unit, TypeDeclaration node) {
	List<BodyDeclaration> decls = node.bodyDeclarations();
	Map<String,FieldGetterSetterDetection> fieldInfos = new HashMap<>();
	// scan methods, detect getter/setter
	for(BodyDeclaration decl : decls) {
		if (decl instanceof MethodDeclaration) {
			MethodDeclaration m = (MethodDeclaration) decl;
			String methName = m.getName().getIdentifier();
			if (methName.startsWith("get") || methName.startsWith("is")) {
				String propName = MatchASTUtils.matchBasicGetterMeth(m);
				if (propName != null) {
					FieldGetterSetterDetection fo = getOrCreateField(fieldInfos, propName);
					fo.getterDecl = m;
				}
			} else if (methName.startsWith("set")) {
				String propName = MatchASTUtils.matchBasicSetterMeth(m);
				if (propName != null) {
					FieldGetterSetterDetection fo = getOrCreateField(fieldInfos, propName);
					fo.setterDecl = m;
				}
			}
		}
	}
	// scan fields, lookup corresponding getter/setter
	for(BodyDeclaration decl : decls) {
		if (decl instanceof FieldDeclaration) {
			FieldDeclaration f = (FieldDeclaration) decl;
			List<VariableDeclarationFragment> fieldDecls = f.fragments();
			if (fieldDecls.size() != 1) {
				continue; // unsupported yet! "private Type x, y;" ... only "private Type x;"
			}
			for(VariableDeclarationFragment fieldDecl : fieldDecls) {
				String fieldName = fieldDecl.getName().getIdentifier();
				FieldGetterSetterDetection fo = fieldInfos.get(fieldName);
				if (fo != null) {
					fo.fieldDecl = fieldDecl;
					
					if (fo.setterDecl != null) {
						// refactor: add lombok annotation @Setter .. delete setter method
						JavaASTUtil.addMarkerAnnotation(unit, f.modifiers(), 
								LOMBOK_PACKAGE, ANNOTATION_LOMBOK_SETTER_SIMPLENAME);
						fo.setterDecl.delete();
					}
					if (fo.getterDecl != null) {
						// refactor: add lombok annotation @Getter .. delete getter method
						JavaASTUtil.addMarkerAnnotation(unit, f.modifiers(), 
								LOMBOK_PACKAGE, ANNOTATION_LOMBOK_GETTER_SIMPLENAME);
						fo.getterDecl.delete();
					}
				}							
			}
		}
	}
}

protected static FieldGetterSetterDetection getOrCreateField(Map<String,FieldGetterSetterDetection> fieldInfos, String name) {
	FieldGetterSetterDetection res = fieldInfos.get(name);
	if (res == null) {
		res = new FieldGetterSetterDetection(name);
		fieldInfos.put(name, res);
	}
	return res;
}
{% endhighlight %}	


<H1>Day 2 ... see next post : <A href="{{site.url}}/2017/01/21/eclipse-lombokify-valvar.html">Lombokifier Eclipse Plugin (Day 2)</A></H1>


