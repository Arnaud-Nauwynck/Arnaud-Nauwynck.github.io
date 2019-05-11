---
layout: post
title:  "Maven Local Repository Files Spy, using GlowRoot"
date:   2019-05-11 17:00:00
categories: 
tags: maven jvm instrumentation agent glowroot
---

<H2>Introduction</H2>

I want to spy maven for all File creation/modification/deletion under its local repository dir.
<p>

Moreover, I want to have the stack trace (the source code) that performs such calls.
<p>

Let's launch maven instrumented under several conditions:
<ul>
<li> mvn install</li>
<li> mvn ...  downloading from remote repositories some pom.xml and jar files</li>
<li> mvn ...  downloading from an invalid Http Proxy some pom.xml and jar files, all files are corrupted html files (using default checkSumPolicy=warn)</li>
<li> mvn ...  downloading from an invalid Http Proxy some pom.xml and jar files, but forcing checkSumPolicy=fail</li>
</ul>

<H2>Using (see previous post) Glowroot Plugin on Maven itself</H2>

In the previous post, I have described a Glowroot plugin to instrument calls done on java.io.*
<p>

Now for instrumenting maven itself, let's use it.
Notice maven use some custom classloaders, and for it Bootstrap ClassLoader, it scans all jars in "${MAVEN_HOME}/boot/plexus-classworlds-*.jar"
<p>

Therefore, it is simpler to use the jvm argument "-Xbootclasspath/a:" to append a jar in the bootclasspath:
<p>

I use this script to launch an instrumented maven jvm:
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


<H2>Result File write acces for "mvn install"</H2>

Write access performed by maven are printed to stdout lines starting by '#### (glowroot-file) ', with the corresponding stack trace.
<p>

In this default plugin configuration, only pathes that start with '/home/' are printed, this can be changed by glooroot arguments and configuration files (-Dglowroot.conf.dir, config.json, config-default.json, ...)
<p>

Notice that default values from the META-INF/glowroot.plugin.json get cached in the glowroot/config.json !
<p>


Results logs:
{% highlight text %}
2019-05-11 19:58:44.813 INFO  org.glowroot - Glowroot version: 0.13.4, built 2019-04-21 20:27:33 +0000
2019-05-11 19:58:44.815 INFO  org.glowroot - Java version: 1.8.0_131 (Oracle Corporation / Linux)
2019-05-11 19:58:44.816 INFO  org.glowroot - Java args: -javaagent:./glowroot/glowroot.jar -Xbootclasspath/a:./target/test-glowroot-0.0.1-SNAPSHOT.jar
#### (glowroot-file)  cinit class fr.an.test.glowroot.FileAspect
#### (glowroot-file)  logging only java.io.File modifications under path:'/home/' with pattern:.*
#### (glowroot-file) (1/1) FileOutputStream.open() '/home/arnaud/.oracle_jre_usage/db4f55ab9af3a5a2.timestamp'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileOutputStreamOpenAdvice.onBefore:56/FileOutputStream.open/<init>:213/<init>:162/UsageTrackerClient.registerUsage:434/setupAndTimestamp:300/access$000:78/UsageTrackerClient$4.run:322/run:317/AccessController.doPrivileged/UsageTrackerClient.run:317/PostVMInitHook.trackJavaUsage:29/run:21
2019-05-11 19:58:48.152 ERROR org.glowroot - Error binding to 127.0.0.1:4000, the UI is not available (will keep trying to bind...): Address already in use
#### (glowroot-file) File.mkdirs() '/home/arnaud/.m2/repository'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileMkdirsAdvice.onBefore:107/File.mkdirs/DefaultMaven.validateLocalRepository:360/doExecute:169/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< fr.an.tests:test-glowroot >----------------------
[INFO] Building test-glowroot 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
2019-05-11 19:58:49.155 ERROR org.glowroot - Error binding to 127.0.0.1:4000, the UI is not available (will keep trying to bind...): Address already in use
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ test-glowroot ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ test-glowroot ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ test-glowroot ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ test-glowroot ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ test-glowroot ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ test-glowroot ---
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ test-glowroot ---
[INFO] Installing /mnt/a_1tera2/homeData/arnaud/perso/devPerso/my-github/test-snippets.github/test-glowroot/target/test-glowroot-0.0.1-SNAPSHOT.jar to /home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT/test-glowroot-0.0.1-SNAPSHOT.jar
#### (glowroot-file) File.mkdirs() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileMkdirsAdvice.onBefore:107/File.mkdirs/TrackingFileManager.update:95/EnhancedLocalRepositoryManager.addRepo:193/addArtifact:171/add:149/DefaultInstaller.install:275/install:195/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
[INFO] Installing /mnt/a_1tera2/homeData/arnaud/perso/devPerso/my-github/test-snippets.github/test-glowroot/pom.xml to /home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT/test-glowroot-0.0.1-SNAPSHOT.pom
#### (glowroot-file) (2/5) FileOutputStream.open() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT/test-glowroot-0.0.1-SNAPSHOT.pom'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileOutputStreamOpenAdvice.onBefore:56/FileOutputStream.open/<init>:213/<init>:162/DefaultFileProcessor.copy:166/copy:150/DefaultInstaller.install:267/install:195/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
#### (glowroot-file) File.mkdirs() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileMkdirsAdvice.onBefore:107/File.mkdirs/TrackingFileManager.update:95/EnhancedLocalRepositoryManager.addRepo:193/addArtifact:171/add:149/DefaultInstaller.install:275/install:195/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
#### (glowroot-file) File.mkdirs() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileMkdirsAdvice.onBefore:107/File.mkdirs/MavenMetadata.write:115/merge:78/DefaultInstaller.install:302/install:205/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
#### (glowroot-file) (3/6) FileOutputStream.open() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot/0.0.1-SNAPSHOT/maven-metadata-local.xml'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileOutputStreamOpenAdvice.onBefore:56/FileOutputStream.open/<init>:213/<init>:162/XmlStreamWriter.<init>:60/WriterFactory.newXmlWriter:122/MavenMetadata.write:116/merge:78/DefaultInstaller.install:302/install:205/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
#### (glowroot-file) File.mkdirs() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileMkdirsAdvice.onBefore:107/File.mkdirs/MavenMetadata.write:115/merge:78/DefaultInstaller.install:302/install:205/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
#### (glowroot-file) (4/7) FileOutputStream.open() '/home/arnaud/.m2/repository/fr/an/tests/test-glowroot/maven-metadata-local.xml'
#### (glowroot-file) from stack: FileAspect.logCall:155/FileAspect$FileOutputStreamOpenAdvice.onBefore:56/FileOutputStream.open/<init>:213/<init>:162/XmlStreamWriter.<init>:60/WriterFactory.newXmlWriter:122/MavenMetadata.write:116/merge:78/DefaultInstaller.install:302/install:205/install:152/DefaultRepositorySystem.install:377/DefaultArtifactInstaller.install:107/InstallMojo.execute:115/DefaultBuildPluginManager.executeMojo:137/MojoExecutor.execute:208/execute:154/execute:146/LifecycleModuleBuilder.buildProject:117/buildProject:81/SingleThreadedBuilder.build:56/LifecycleStarter.execute:128/DefaultMaven.doExecute:305/doExecute:192/execute:105/MavenCli.execute:954/doMain:288/main:192/NativeMethodAccessorImpl.invoke0/invoke:62/DelegatingMethodAccessorImpl.invoke:43/Method.invoke:498/Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.859 s
[INFO] Finished at: 2019-05-11T19:58:49+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}


The print format for a stack trace element is 
<br/>
&nbsp;&nbsp;    "className.methodName:lineNumber/" 
<br/>
or simply 
<br/>
&nbsp;&nbsp;    "methodName:lineNumber/" to avoid repeating the same className as the one before.
<p>

Moreover, only the short classname are printed... it is pretty easy to find out the fully qualified name for the short name.
<p>

The equivalent stack trace vertically is ..
{% highlight shell %}
FileAspect$FileOutputStreamOpenAdvice.onBefore:46
FileOutputStream.open/<init>:213/<init>:162
XmlStreamWriter.<init>:60
WriterFactory.newXmlWriter:122
MavenMetadata.write:116/merge:78
DefaultInstaller.install:302/install:205/install:152
DefaultRepositorySystem.install:377
DefaultArtifactInstaller.install:107
InstallMojo.execute:115
DefaultBuildPluginManager.executeMojo:137
MojoExecutor.execute:208/execute:154/execute:146
LifecycleModuleBuilder.buildProject:117/buildProject:81
SingleThreadedBuilder.build:56
LifecycleStarter.execute:128
DefaultMaven.doExecute:305/doExecute:192/execute:105
MavenCli.execute:954
MavenCli.doMain:288
MavenCli.main:192
NativeMethodAccessorImpl.invoke0
NativeMethodAccessorImpl.invoke:62
DelegatingMethodAccessorImpl.invoke:43
Method.invoke:498
Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
{% endhighlight %}


<H2>Test again .. log stacktrace for files downloaded and written to local repo</H2>

Here is another test execution, to detect the files downloaded by maven and written to local repository


{% highlight shell %}
rm -rf ~/.m2/repository/org/glowroot/glowroot-agent-plugin-api/
./mvn-glowroot-file.sh package
{% endhighlight %}

Results:
written files:
<ul>
<li> .pom.part</li>
<li> .pom.sha1-$(uuid).tmp</li>
<li> .pom         &nbsp;&nbsp;(.pom.part renamedTo)</li>
<li> .pom.sha1    &nbsp;&nbsp;(.pom.sha1..tmp renamedTo)</li>
<li> .jar.part</li>
<li> .jar.sha1-$(uuid).tmp </li>
<li> .jar         &nbsp;&nbsp;(.jar.part renamedTo)</li>
<li> .jar.sha1    &nbsp;&nbsp;(.pom.sha1 renamedTo)</li>
</ul>

All following stack traces have in common this stack fragment, ommited next

{% highlight text %}
    MojoExecutor.execute:202/execute:156/execute:148
    LifecycleModuleBuilder.buildProject:117/buildProject:81
    SingleThreadedBuilder.build:56
    LifecycleStarter.execute:128
    DefaultMaven.doExecute:305/doExecute:192/execute:105
    MavenCli.execute:956/doMain:288/main:192
    NativeMethodAccessorImpl.invoke0:62
    DelegatingMethodAccessorImpl.invoke:43
    Method.invoke:498
    Launcher.launchEnhanced:289/launch:229/mainWithExitCode:415/main:356
{% endhighlight %}




{% highlight text %}
..

Downloading from central: https://repo.maven.apache.org/maven2/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom
#### (glowroot-file) FileOutputStream.open() '..\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.part 
#### (glowroot-file) from stack:
    FileOutputStream.open/<init>:213/<init>:162
    LazyFileOutputStream.initialize:154
    LazyFileOutputStream.write:126
    AbstractWagon.transfer:581/getTransfer:372/getTransfer:315/getTransfer:284
    StreamWagon.getIfNewer:97/get:61
    WagonTransporter$GetTaskRunner.run:567
    WagonTransporter.execute:435/get:412
    BasicRepositoryConnector$GetTaskRunner.runTask:456
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.resolveCachedArtifactDescriptor:530/getArtifactDescriptorResult:513/
        processDependency:402/processDependency:356/process:344/collectDependencies:247
    DefaultRepositorySystem.collectDependencies:269
    DefaultProjectDependenciesResolver.resolve:169
    LifecycleDependencyResolver.getDependencies:243/resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248
        
#### (glowroot-file) FileOutputStream.open() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.sha1-77ce64e42112642211906003866.tmp 
#### (glowroot-file) from stack:
    FileOutputStream.open:213/<init>:162
    LazyFileOutputStream.initialize:154/write:126
    AbstractWagon.transfer:581/getTransfer:372/getTransfer:315/getTransfer:284
    StreamWagon.getIfNewer:97/get:61
    WagonTransporter$GetTaskRunner.run:567
    WagonTransporter.execute:435/get:412
    BasicRepositoryConnector$GetTaskRunner.fetchChecksum:423
    ChecksumValidator.validateExternalChecksums:157/validate:103
    BasicRepositoryConnector$GetTaskRunner.runTask:459
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.resolveCachedArtifactDescriptor:530/getArtifactDescriptorResult:513/
        processDependency:402/processDependency:356/process:344/collectDependencies:247
    DefaultRepositorySystem.collectDependencies:269
    DefaultProjectDependenciesResolver.resolve:169
    LifecycleDependencyResolver.getDependencies:243/resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248

#### (glowroot-file) (1/1) File.renameTo() dest: '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom' src:  '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.part' 
#### (glowroot-file) from stack:
    File.renameTo
    DefaultFileProcessor.move:249
    BasicRepositoryConnector$GetTaskRunner.runTask:481
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.resolveCachedArtifactDescriptor:530/getArtifactDescriptorResult:513
        /processDependency:402/processDependency:356/process:344
    DefaultDependencyCollector.collectDependencies:247
    DefaultRepositorySystem.collectDependencies:269
    ProjectDependenciesResolver.resolve:169
    LifecycleDependencyResolver.getDependencies:243
    LifecycleDependencyResolver.resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248

#### (glowroot-file) (2/2) File.renameTo() dest: '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.sha1' 
    src:  '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.sha1-ad24d7b01489881715896332211.tmp'
#### (glowroot-file) from stack:
    File.renameTo
    DefaultFileProcessor.move:249
    ChecksumValidator.commit:244
    BasicRepositoryConnector$GetTaskRunner.runTask:484
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.resolveCachedArtifactDescriptor:530/getArtifactDescriptorResult:513
        /processDependency:402/processDependency:356/process:344/collectDependencies:247
    DefaultRepositorySystem.collectDependencies:269
    ProjectectDependenciesResolver.resolve:169
    LifecycleDependencyResolver.getDependencies:243
    LifecycleDependencyResolver.resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248

Downloaded from central: https://repo.maven.apache.org/maven2/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom : https://repo.maven.apache.org/maven2/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.jar

#### (glowroot-file) FileOutputStream.open() ~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.jar.part 
#### (glowroot-file) from stack:
    FileOutputStream.open:213/<init>:162
    LazyFileOutputStream.initialize:154/write:126
    AbstractWagon.transfer:581/getTransfer:372/getTransfer:315/getTransfer:284
    StreamWagon.getIfNewer:97/get:61
    WagonTransporter$GetTaskRunner.run:567
    WagonTransporter.execute:435/get:412
    BasicRepositoryConnector$GetTaskRunner.runTask:456
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215
    DefaultRepositorySystem.resolveDependencies:325
    DefaultProjectDependenciesResolver.resolve:202
    LifecycleDependencyResolver.getDependencies:243/resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248
    
#### (glowroot-file) FileOutputStream.open() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.jar.sha1-cebc165f7113521320898028675.tmp' 
#### (glowroot-file) from stack:
    FileOutputStream.open:213/<init>:162
    LazyFileOutputStream.initialize:154/write:126
    AbstractWagon.transfer:581/getTransfer:372/getTransfer:315/getTransfer:284
    StreamWagon.getIfNewer:97/get:61
    WagonTransporter$GetTaskRunner.run:567
    WagonTransporter.execute:435/get:412
    BasicRepositoryConnector$GetTaskRunner.fetchChecksum:423
    ChecksumValidator.validateExternalChecksums:157
    ChecksumValidator.validate:103
    BasicRepositoryConnector$GetTaskRunner.runTask:459
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215
    DefaultRepositorySystem.resolveDependencies:325
    DefaultProjectDependenciesResolver.resolve:202
    LifecycleDependencyResolver.getDependencies:243/resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248
    
#### (glowroot-file) File.renameTo() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.jar' 
    src: '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.jar.part'
#### (glowroot-file) from stack:
    File.renameTo
    DefaultFileProcessor.move:249
    BasicRepositoryConnector$GetTaskRunner.runTask:481
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215
    DefaultRepositorySystem.resolveDependencies:325
    DefaultProjectDependenciesResolver.resolve:202
    LifecycleDependencyResolver.getDependencies:243/resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248)

#### (glowroot-file) File.renameTo() dest: '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.jar.sha1'
    src: '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.jar.sha1-5c65fc31805141540408522583.tmp'
#### (glowroot-file) from stack:
    File.renameTo
    DefaultFileProcessor.move:249
    ChecksumValidator.commit:244
    BasicRepositoryConnector$GetTaskRunner.runTask:484
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215
    DefaultRepositorySystem.resolveDependencies:325
    DefaultProjectDependenciesResolver.resolve:202
    LifecycleDependencyResolver.getDependencies:243/resolveProjectDependencies:147
    MojoExecutor.ensureDependenciesAreResolved:248
    
{% endhighlight %}



<H2>Test again .. downloaded invalid checkSum files, with default checkSumPolicy=warn</H2>

I am using here a dummy http repository server, that always answer http 200, with a welcome html page. Therefore, all pom, jar and sh1 files are invalid.


For this, I use an additionnal maven profile "-Pdummy-repo"

In the ~/.m2/settings.xml:
{% highlight xml %}
    <profile>
        <id>dummy-repo</id>
        <repositories>
            <repository>
                <id>central</id>
                <url>http://localhost:8090/repo</url>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>central</id>
                <url>http://localhost:8090/repo</url>
            </pluginRepository>
        </pluginRepositories>
    </profile>
{% endhighlight %}


{% highlight shell %}
rm -rf ~/.m2/repository/org/glowroot/glowroot-agent-plugin-api/
./mvn-glowroot-file.sh -Pdummy-repo package
{% endhighlight %}

Now, we can see maven is still downloading pom.part, pom.sha1, and then does renameTo the same way !!!
By debugging it, this is because by default, repositories have checkSumPolicy="warn", instead of "fail"

{% highlight text %}
[WARNING] Checksum validation failed, expected <!DOCTYPE but is 28db9960ddfec8ed449bb87ec4d17c139d9301fd from central for http://localhost:8090/repo/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom

[WARNING] Could not validate integrity of download from http://localhost:8090/repo/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom: Checksum validation failed, expected <!DOCTYPE but is 28db9960ddfec8ed449bb87ec4d17c139d9301fd
                   
[WARNING] Checksum validation failed, expected <!DOCTYPE but is 28db9960ddfec8ed449bb87ec4d17c139d9301fd from central for http://localhost:8090/repo/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom

#### (glowroot-file) (1/1) File.renameTo() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom'
    src:'~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.part'
#### (glowroot-file) from stack:
    File.renameTo
    DefaultFileProcessor.move:249
    BasicRepositoryConnector$GetTaskRunner.runTask:481
    BasicRepositoryConnector$TaskRunner.run:363)
    RunnableErrorForwarder$1.run:75)
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.resolveCachedArtifactDescriptor:530


#### (glowroot-file) (2/2) File.renameTo() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.sha1' 
    src:'~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.sha1-1461f17f5117003776646361438.tmp'
#### (glowroot-file) from stack:
    File.renameTo
    DefaultFileProcessor.move:249
    ChecksumValidator.commit:244
    BasicRepositoryConnector$GetTaskRunner.runTask:484
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489
    DefaultArtifactResolver.resolve:390
{% endhighlight %}


<H2>Test again .. downloaded invalid checkSum files, with forced checkSumPolicy=fail</H2>

For this, I now use maven profile "-Pdummy-repo-fail"

In the ~/.m2/settings.xml:
{% highlight xml %}
    <profile>
        <id>dummy-repo-fail</id>
        <repositories>
            <repository>
                <id>central</id>
                <url>http://localhost:8090/repo</url>
                <releases>
                    <checksumPolicy>fail</checksumPolicy>
                </releases>
                <snapshots>
                    <checksumPolicy>fail</checksumPolicy>
                </snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>central</id>
                <url>http://localhost:8090/repo</url>
                <releases>
                    <checksumPolicy>fail</checksumPolicy>
                </releases>
                <snapshots>
                    <checksumPolicy>fail</checksumPolicy>
                </snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>
{% endhighlight %}


{% highlight shell %}
rm -rf ~/.m2/repository/org/glowroot/glowroot-agent-plugin-api/
./mvn-glowroot-file.sh -Pdummy-repo-fail package
{% endhighlight %}

Now maven correctly detects a build failure.

{% highlight text %}
Downloading from central: http://localhost:8090/repo/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom
[WARNING] Checksum validation failed, expected <!DOCTYPE but is 28db9960ddfec8ed449bb87ec4d17c139d9301fd from central for http://localhost:8090/repo/org/glowroot/glowroot-agent-plugin-api/0.13.4/glowroot-agent-plugin-api-0.13.4.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.224 s
[INFO] Finished at: 2019-05-11T15:42:35+02:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project test-glowroot: Could not resolve dependencies for project fr.an.tests:test-glowroot:jar:0.0.1-SNAPSHOT: Failed to collect dependencies at org.glowroot:glowroot-agent-plugin-api:jar:0.13.4: Failed to read artifact descriptor for org.glowroot:glowroot-agent-plugin-api:jar:0.13.4: Could not transfer artifact org.glowroot:glowroot-agent-plugin-api:pom:0.13.4 from/to central (http://localhost:8090/repo): Checksum validation failed, expected <!DOCTYPE but is 28db9960ddfec8ed449bb87ec4d17c139d9301fd -> [Help 1]
{% endhighlight %}

And we can check that only ".part" or ".tmp" files are written, and they are deleted correctly.

{% highlight text  %}
#### (glowroot-file) (2/2) File.delete() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.part'
#### (glowroot-file) from stack:
    at java.io.File.delete:342)
    BasicRepositoryConnector$GetTaskRunner.runTask:489
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.collectDependencies:247

#### (glowroot-file) (3/3) File.delete() '~\.m2\repository\org\glowroot\glowroot-agent-plugin-api\0.13.4\glowroot-agent-plugin-api-0.13.4.pom.part.lock'
#### (glowroot-file) from stack:
    File.delete
    PartialFile$LockFile.close:236
    PartialFile.close:349
    BasicRepositoryConnector$GetTaskRunner.runTask:489
    BasicRepositoryConnector$TaskRunner.run:363
    RunnableErrorForwarder$1.run:75
    BasicRepositoryConnector$DirectExecutor.execute:642
    BasicRepositoryConnector.get:262
    DefaultArtifactResolver.performDownloads:489/resolve:390/resolveArtifacts:215/resolveArtifact:192
    DefaultArtifactDescriptorReader.loadPom:240/readArtifactDescriptor:171
    DefaultDependencyCollector.collectDependencies:247

{% endhighlight %}



<H2>Conclusion</H2>

Maven is unfortunatly badly configured by default for checkSumPolicy...<br/>
It uses "warn", but it should rather use "fail" <br/>
In particular if you are working behind a badly configured http proxy.
<p>

Glowroot is very helpfull to understand deep inside a jvm what is going on, without touching the program source code 
<p>

View the full source-code 
<A href="https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-glowroot">https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-glowroot</A>



