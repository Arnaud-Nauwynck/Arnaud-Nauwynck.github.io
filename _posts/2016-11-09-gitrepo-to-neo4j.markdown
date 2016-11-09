---
layout: post
title:  "Git Repo to Neo4j"
date:   2016-11-09 23:00:00
categories: 
tags: jgit neo4j
---

<h1>Git Repo to Neo4J</h1>

Here is a little project (~ 2 days of development), playing around with both JGit and Neo4J (and springboot-data-neo4j).

It read Git Commits/RevTree/Person/... graph from a local Git repository, and update the corresponding Graph objects in Neo4J.
<BR/>
So you can display it graphically, and perform Cypher requests on it.

<BR/>
I have tested it on the JGit repository itself ... 
<img src="{{site.url}}/assets/posts/2016-11-09-gitrepo-to-neo4j/screenshot-git2neo4j.png"></img>

The source code repository "gitrepo-to-neo4j" is open-source, and you can found it here:
<A href="https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/gitrepo-to-neo4j">Github test-snippets/gitrepo-to-neo4j</A>
<BR/>


You could try displaying All objects...but it is too much for one screen.


<H2>Java Libraries</H2>

<H3>JGit</H3>
JGit = a java library for accessing a Git repository

<H3>Neo4J</H3>
Neo4J = a graph database  (with java driver client, rich Web graphical client (d3s..), a Rest api, ..)
<BR/>
 The fact is that Neo4J server is also written in java make it a breeze to download+unzip+start. 

<H3>Springboot-data-neo4j</H3>
springboot-data-neo4j = springboot magic for Neo4j ...  


<H2>Motivations...</H2>

It is fun to code, at least more fun that playing stupid video games, like many students I know...

More seriously, at work there are complex reporting tools trying to check informations between SVN/GIT and Jira, and then between Jira and a deployment tool (XLDeploy from XebiaLabs)...

some One of the  goals is to check that a developped feature as specified in a Jira Issue, is deployed on Developments/Tests/Prod environments.
<BR/>

Internally, a top level Jira feature request has many corresponding Jira sub-issues per deployable components, which corresponds to many GIT commits (the Jira issue id is mandatory in the git commits message ..). 
Then jenkins builds are linked to Git commits, and produce deployable versions... which are deployed using the deployment tool... 

These reporting tool is quite a mess.
<BR/>

It was fun re-looking at it as a unification Graph database of both GIT commits graph and Jira Issues graph.
This was the subject of a team lunch-time Coding DOJO, as I organised them every week at work.
 


<H2>Getting started with Neo4J</H2>

<ul>
<li>Download</li>
<li>Start
just type "./bin/neo4j start" ... or if you are running on windows, and not a Windows local administrator, (cannot start a windows Service) => just use ".\bin\neo4j.bat console"
</li>
<li>open your browser on http://localhost:7474/browser
All you have to do is to change your password on first login
</li>
<li>there is also a nice running demo embedded in the UI</li>
</ul>


Some links:
<ul>
<li> <A href="https://neo4j.com/">https://neo4j.com/</A> </li>
<li> demo in http://localhost:7474/browser </li>
<li> <A href="https://neo4j.com/docs/cypher-refcard/current/">Cypher Cheat Sheet</A> </li>
</ul>


Some Example of Cypher commands:
{% highlight %}

match(r:SymbolicRepoRef) return r

match(c: RevCi) return c limit 100

match(u:Person) return u limit 100

match(c: RevCi)-[r]-(t) return c,r,t limit 100

match(c: RevCi)-[p:parent]-(c2: RevCi)  return c,p,c2 limit 100

{% endhighlight %}


Here is an extract example of Java code for inserting / reading Person in Neo4J graph database 
{% highlight java %}

@NodeEntity(label="Person")
public class PersonIdentEntity {

	private Long id;
	
	private String name;
	private String emailAddress;
	
	// getter-setter ommited .. use Lombok some day
}

import org.springframework.data.neo4j.repository.GraphRepository;
import fr.an.tools.git2neo4j.domain.PersonIdentEntity;

/** DAO=Repository ... magically created/injected by springboot-data */ 
public interface PersonDAO extends GraphRepository<PersonIdentEntity> {
}

@Component
public class Git2Neo4JSyncService {

	@Autowired
	private PersonDAO personDAO;

	@Transactional
	public void syncRepo(Git git) {
	    // find all persons from Neo4j graph db
		Iterable<PersonIdentEntity> personEntities = personDAO.findAll();
		
		// traverse jgit RevCommit graphs
		org.eclipse.jgit.lib.RevCommit revCommit = ..   
		org.eclipse.jgit.lib.PersonIdent commitAuthor = revCommit.getAuhor();
		
		// if already created, find personEntities by email
		PersonIdentEntity personEntity = ..  
		if (personEntity == null) {
		  // create corresponding Neo4j person entity
		  personEntity = createPersonEntity(commitAuthor);
		}
	}	
	
	protected PersonIdentEntity createPersonEntity(PersonIdent src) {
		PersonIdentEntity res = new PersonIdentEntity();
		res.setEmailAddress(email);
		res.setName(src.getName());

		return personDAO.save(res);
	}
}

{% endhighlight java %}



<H2>Getting started with JGit</H2>

Of course, it is mandatory to understand GIT Internals, before trying to use JGit.
I recommend 
<ul>
<li>
excellent reading books (like <A href="https://github.s3.amazonaws.com/media/progit.en.pdf">Pro-GIT.df<</A>)
</li>
<li>online <A href="http://ndpsoftware.com/git-cheatsheet.html">Visual Git Cheat Sheet</A> </li>
<li><A href="http://onlywei.github.io/explain-git-with-d3/">explain-git-with-d3</A> </li>
<li> <A href="https://marklodato.github.io/visual-git-guide/index-en.html">visual-git-guide<A> </li> 
<li> and presentations (including one of mine : <A href="http://arnaud.nauwynck.free.fr/CoursIUT/CoursIUT-IntroGIT.pdf">CoursIUT-IntroGIT.pdf</A>) </li>
</ul>


{% highlight java %}

{% endhighlight java %}




