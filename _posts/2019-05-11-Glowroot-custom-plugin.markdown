---
layout: post
title:  "Test Glowroot Custom Plugin Jvm Instrumentation"
date:   2019-05-11 18:00:00
categories: 
tags: jvm instrumentation agent glowroot
---

# Test Glowroot Custom Plugin Jvm Instrumentation

This is a test for capturing some methods calls performed by an instrumentated jvm.
<br/>
It used internally a custom plugin writen with Glowroot  

<br/>
A sample usage is here to detect all file write access performed by maven in "/home" sub dirs.
</br/>

Internally, the plugin instrument all calls to "java.io.FileOutputStream.open(" and "java.io.File.renameTo()",
filter if the path starts with "/home/", and print the corresponding stack trace with the count/total of write access.


# Discover Glowroot
[https://github.com/glowroot]

Glowroot is an APM (Application Performance Management) like many other expensive closed-source tools, but it is free and Open Source..

The performance measurment offer a rich centralised server and a web UI, but this is not required. Glowroot can run as embedded server, or no server.

Moreover, glowroot is internally based on a jvm instrumentation engine, is modular by plugins, and can be used without the performance api parts.  


# Step 1: download and unzip GlowRoot agent

{% highlight shell %}
wget https://github.com/glowroot/glowroot/releases/download/v0.13.4/glowroot-0.13.4-dist.zip
unzip glowroot-0.13.4-dist.zip
{% endhighlight %}


# Step 2: Create a custom Glowroot plugin

Create the pom.xml with the following dependencies
{% highlight xml %}

{% highlight %}

Then code in java the instrumentation.
<br/>
Here, I am not interrested in performance traces, but only adding a "System.out.println()" for all java.io.* calls that modify files:

- java.io.FileOutputStream.open(String,boolean)
- java.io.File.mkdir()
- java.io.File.mkdirs()
- java.io.File.renameTo(File)
- java.io.File.delete()
- java.io.File.deleteOnExit()


{% highlight java %}
package fr.an.test.glowroot;

import java.io.File;

import org.glowroot.agent.plugin.api.weaving.BindParameter;
import org.glowroot.agent.plugin.api.weaving.BindReceiver;
import org.glowroot.agent.plugin.api.weaving.OnBefore;
import org.glowroot.agent.plugin.api.weaving.Pointcut;

public class FileAspect {    

    /**
     * Instrument calls to <code>java.io.FileOutputStream.open()</code>
     */
    @Pointcut(className = "java.io.FileOutputStream", methodName = "open", methodParameterTypes = {"java.lang.String", "boolean"})
    public static class FileOutputStreamOpenAdvice {
        @OnBefore
        public static void onBefore(@BindParameter String path) {
            System.out.println("***** FileOutputStream.open() '" + path + "' ... ");
        }
    }

    // more... truncated    
}
{% endhighlight %}
    

Compile it:
{% highlight shell %}
mvn package
{% endhighlight %}



# Step 3: Run a jvm with the Glowroot plugin and agent

execute a java process, by adding jvm arguments
{% highlight shell %}
-javaagent:./glowroot/glowroot.jar -Xbootclasspath/a:./glowroot-plugin.jar"
{% endhighlight %}


For example for instrumenting maven itself 

{% highlight shell %}
./mvn-glowroot-file.sh install
{% endhighlight %}

cf detailed script:
{% highlight shell %}
#!/bin/bash
# file: mvn-glowroot-file.sh
# script similar to "mvn", that add glowroot file write plugin instrumentation

PROJECT_VERSION=$(mvn -q -DforceStdout help:evaluate -Dexpression=project.version)
PLUGIN_JAR="./target/test-glowroot-${PROJECT_VERSION}.jar"

MAVEN_OPTS="${MAVEN_OPTS} -javaagent:./glowroot/glowroot.jar"
MAVEN_OPTS="${MAVEN_OPTS} -Xbootclasspath/a:${PLUGIN_JAR}"

# MAVEN_OPTS="${MAVEN_OPTS} -Dglowroot.debug.printClassLoading=true"

export MAVEN_OPTS

mvn $@ 
{% endhighlight %}



# Step 4: Analyse logs

Here for the mvn execution with the custom glowroot agent, I 
get these result logs.

{% highlight text %}
2019-05-11 18:48:54.056 INFO  org.glowroot - Glowroot version: 0.13.4, built 2019-04-21 20:27:33 +0000                                                           
2019-05-11 18:48:54.056 INFO  org.glowroot - Java version: 1.8.0_202 (Oracle Corporation / Windows 10)                                                           
2019-05-11 18:48:54.056 INFO  org.glowroot - Java args: -javaagent:./glowroot/glowroot.jar -Xbootclasspath/a:./target/test-glowroot-0.0.1-SNAPSHOT.jar           
**** cinit class fr.an.test.glowroot.FileAspect                                                                                                                  
**** logging only java.io.File modifications under path:'' with pattern:.*                                                                                       
***** (1/1) FileOutputStream.open() 'C:\ProgramData\Oracle\Java\.oracle_jre_usage\8f6ba24f3ed3fb62.timestamp'                                                    
***** File.mkdirs() 'C:\cygwin64\tmp'                                                                                                                            
***** (2/2) FileOutputStream.open() 'D:\arn\devPerso\mygithub\test-snippets\test-glowroot\glowroot\data\data.lock.db'                                            
2019-05-11 18:49:02.814 INFO  org.glowroot - UI listening on 127.0.0.1:4000 (to access the UI from remote machines, change the bind address to 0.0.0.0, either in
 the Glowroot UI under Configuration > Web or directly in the admin.json file, and then restart JVM to take effect)                                              
***** File.mkdirs() 'C:\Users\arnau\.m2\repository'                                                                                                              
[INFO] Scanning for projects...                                                                                                                                  
[INFO]                                                                                                                                                           
[INFO] ---------------------< fr.an.tests:test-glowroot >----------------------                                                                                  
[INFO] Building test-glowroot 0.0.1-SNAPSHOT                                                                                                                     
[INFO] --------------------------------[ jar ]---------------------------------                                                                                  
[INFO]                                                                                                                                                           
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ test-glowroot ---                                                                          
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources, i.e. build is platform dependent!                                                
[INFO] Copying 1 resource                                                                                                                                        
[INFO]                                                                                                                                                           
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ test-glowroot ---                                                                             
***** (3/3) FileOutputStream.open() 'D:\arn\devPerso\mygithub\test-snippets\test-glowroot\target\maven-status\maven-compiler-plugin\compile\default-compile\input
Files.lst'                                                                                                                                                       
[INFO] Nothing to compile - all classes are up to date                                                                                                           
[INFO]                                                                                                                                                           
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ test-glowroot ---                                                                  
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources, i.e. build is platform dependent!                                                
[INFO] skip non existing resourceDirectory D:\arn\devPerso\mygithub\test-snippets\test-glowroot\src\test\resources                                               
[INFO]                                                                                                                                                           
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ test-glowroot ---                                                                     
[INFO] No sources to compile                                                                                                                                     
[INFO]                                                                                                                                                           
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ test-glowroot ---                                                                                  
[INFO] Tests are skipped.                                                                                                                                        
[INFO]                                                                                                                                                           
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ test-glowroot ---                                                                                            
***** File.mkdirs() 'D:\arn\devPerso\mygithub\test-snippets\test-glowroot\target'                                                                                
[INFO]                                                                                                                                                           
[INFO] --- maven-install-plugin:2.4:install (default-install) @ test-glowroot ---                                                                                
[INFO] Installing D:\arn\devPerso\mygithub\test-snippets\test-glowroot\target\test-glowroot-0.0.1-SNAPSHOT.jar to C:\Users\arnau\.m2\repository\fr\an\tests\test-
glowroot\0.0.1-SNAPSHOT\test-glowroot-0.0.1-SNAPSHOT.jar                                                                                                         
***** File.mkdirs() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\0.0.1-SNAPSHOT'                                                                     
[INFO] Installing D:\arn\devPerso\mygithub\test-snippets\test-glowroot\pom.xml to C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\0.0.1-SNAPSHOT\test-glo
wroot-0.0.1-SNAPSHOT.pom                                                                                                                                         
***** (4/4) FileOutputStream.open() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\0.0.1-SNAPSHOT\test-glowroot-0.0.1-SNAPSHOT.pom'                    
***** File.mkdirs() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\0.0.1-SNAPSHOT'                                                                     
***** File.mkdirs() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\0.0.1-SNAPSHOT'                                                                     
***** (5/5) FileOutputStream.open() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\0.0.1-SNAPSHOT\maven-metadata-local.xml'                            
***** File.mkdirs() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot'                                                                                    
***** (6/6) FileOutputStream.open() 'C:\Users\arnau\.m2\repository\fr\an\tests\test-glowroot\maven-metadata-local.xml'                                           
[INFO] ------------------------------------------------------------------------                                                                                  
[INFO] BUILD SUCCESS                                                                                                                                             
[INFO] ------------------------------------------------------------------------                                                                                  
[INFO] Total time:  2.859 s                                                                                                                                      
[INFO] Finished at: 2019-05-11T18:49:07+02:00                                                                                                                    
[INFO] ------------------------------------------------------------------------                                                                                  
***** (1/1) File.delete() 'D:\arn\devPerso\mygithub\test-snippets\test-glowroot\glowroot\data\data.lock.db'                                                      
***** (2/2) File.delete() 'D:\arn\devPerso\mygithub\test-snippets\test-glowroot\glowroot\tmp\.lock'                                                              
{% endhighlight %}


# Conclusion

Such logs are even better when combined with stack traces and statistic counts.

It is very helpfull to understand deep inside a jvm what is going on, without touching the program source code 

View the full source-code [https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-glowroot]
