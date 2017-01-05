---
layout: post
title:  "Building Eclipse Platform"
date:   2017-01-05 21:42:00
categories: 
tags: eclipse pde tycho
---

This may be a strange idea, but I wanted to build myself the Eclipse Platform from source code.


<H1>Steps for Building Eclipse</H1>

Everything is described in details here :
<A href="https://wiki.eclipse.org/Platform-releng/Platform_Build">https://wiki.eclipse.org/Platform-releng/Platform_Build</A>

First, check you have ~13 Go of free disk space ... 

The only re-requisite is to have  have jdk8 + maven 3 installed. 
For example in a "setenv-jdk8-mvn3.sh" to source from your ~/.bashrc, you can add something like 
{% highlight text %}
export JAVA_HOME=/opt/devtools/jdk/jdk1.8.0
export PATH=$JAVA_HOME/bin:$PATH

export M2_HOME=/opt/devtools/maven-3.3.9
export PATH=$M2_HOME/bin:$PATH

export MAVEN_OPTS="-Xmx2048m -Declipse.p2.mirrors=false"
{% endhighlight %}


Then proceed to getting source code + compiling (~3 hours, depending of your internet connection)

{% highlight text %}
git clone -b master --recursive git://git.eclipse.org/gitroot/platform/eclipse.platform.releng.aggregator.git
cd eclipse.platform.releng.aggregator

git pull --recurse-submodules
git submodule update
# ...WAIT...

# launch the build!
mvn verify

# ...WAIT...
{% endhighlight %}

Other possible variations for build command line are, 
with skip tests: "-DskipTests", and with build native libs "-Dnative=gtk.linux.x86_64"


I have launched
{% highlight text %}
mvn verify

...
thousands lines of logs ommitted
...

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
..
{% endhighlight %}

It just worked right from the first trial (This should not be a surprise to you given Eclipse code quality)
<BR/>
... But it took a long time ... 
<BR/>
downloading from maven / eclipse repositories ... executing tests  ... building several native distributions (like solaris+windows+apple+... my dear linux)!
<BR/>
Notice that re-building a second time, is faster because there is no more downloading (all in local maven / p2 repository...) ... It took ~ 45 minutes on my PC.


The documentation says build results are located here:
<ul>
<li>SDK zip: org.eclipse.releng.tychoeclipsebuilder/sdk/target/products/</li>
<li>P2 repository : org.eclipse.repository/target/repository </li>
<li>Source bundles : org.eclipse.repository/target/repository/plugins</li>
</ul>

Proof of successfull build:

{% highlight text %}
cd eclipse.platform.releng.tychoeclipsebuilder/platform/target/products
ls -lh

total 1.1G
drwxr-xr-x 8 arnaud arnaud 4.0K Jan  5 23:22 org.eclipse.platform.ide
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-aix.gtk.ppc64.zip
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-aix.gtk.ppc.zip
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-hpux.gtk.ia64.zip
-rw-r--r-- 1 arnaud arnaud  72M Jan  5 23:22 org.eclipse.platform.ide-linux.gtk.ppc64le.tar.gz
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:22 org.eclipse.platform.ide-linux.gtk.ppc64.tar.gz
-rw-r--r-- 1 arnaud arnaud  72M Jan  5 23:22 org.eclipse.platform.ide-linux.gtk.ppc.tar.gz
-rw-r--r-- 1 arnaud arnaud  72M Jan  5 23:23 org.eclipse.platform.ide-linux.gtk.s390.tar.gz
-rw-r--r-- 1 arnaud arnaud  72M Jan  5 23:23 org.eclipse.platform.ide-linux.gtk.s390x.tar.gz
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:22 org.eclipse.platform.ide-linux.gtk.x86_64.tar.gz
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:22 org.eclipse.platform.ide-linux.gtk.x86.tar.gz
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-macosx.cocoa.x86_64.tar.gz
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-solaris.gtk.sparcv9.zip
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-solaris.gtk.x86_64.zip
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-win32.win32.x86_64.zip
-rw-r--r-- 1 arnaud arnaud  73M Jan  5 23:23 org.eclipse.platform.ide-win32.win32.x86.zip
{% endhighlight %}
As you can see ... eclipse is packaged for 15 different platforms, consuming 1G just for the final build results!

Curious about total disk space:

{% highlight text %}
eclipse.platform.releng.aggregator$ du -sh
12G	.
{% endhighlight %}

Even curious about code size ... there is ~ 55k java files (35k source java files + 20k test java files), representing 11 Million lines (8.5 Millions line of java code + 3 millions lines of java test )!!

{% highlight text %}
$ find . -name \*.java | wc -l 
54670

$ find . -name \*.java -exec cat {} \; | wc -l 
11152995

$ find . -name \*.java -not -path \*test\* -exec cat {} \; | wc -l
8581122

{% endhighlight %}




Here is the end of maven log, giving a preview of all maven sub modules and the time for compiling each.

{% highlight text %}

...
thousands lines of logs ommitted
...

[INFO] ------------------------------------------------------------------------
[INFO] Building platform-aggregator 4.7.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] eclipse-platform-parent ............................ SUCCESS [  0.267 s]
[INFO] eclipse-sdk-prereqs ................................ SUCCESS [  0.105 s]
[INFO] eclipse.jdt ........................................ SUCCESS [  0.006 s]
[INFO] rt.equinox.framework ............................... SUCCESS [  0.005 s]
[INFO] org.eclipse.osgi ................................... SUCCESS [ 13.754 s]
[INFO] org.eclipse.osgi.services .......................... SUCCESS [  3.780 s]
[INFO] rt.equinox.bundles ................................. SUCCESS [  0.005 s]
[INFO] org.eclipse.equinox.common ......................... SUCCESS [  3.633 s]
[INFO] eclipse.platform.ui ................................ SUCCESS [  0.004 s]
[INFO] org.eclipse.core.commands .......................... SUCCESS [  3.951 s]
[INFO] eclipse.platform.runtime ........................... SUCCESS [  0.004 s]
[INFO] org.eclipse.core.jobs .............................. SUCCESS [  3.303 s]
[INFO] org.eclipse.equinox.registry ....................... SUCCESS [  3.608 s]
[INFO] org.eclipse.equinox.preferences .................... SUCCESS [  1.932 s]
[INFO] org.eclipse.core.contenttype ....................... SUCCESS [  1.864 s]
[INFO] org.eclipse.core.databinding.observable ............ SUCCESS [  2.609 s]
[INFO] org.eclipse.core.databinding.property .............. SUCCESS [  2.345 s]
[INFO] org.eclipse.core.databinding ....................... SUCCESS [  4.027 s]
[INFO] org.eclipse.equinox.app ............................ SUCCESS [  3.114 s]
[INFO] org.eclipse.core.runtime ........................... SUCCESS [  3.141 s]
[INFO] org.eclipse.core.expressions ....................... SUCCESS [  3.206 s]
[INFO] org.eclipse.e4.core.di.annotations ................. SUCCESS [  2.430 s]
[INFO] org.eclipse.e4.core.di ............................. SUCCESS [  2.731 s]
[INFO] org.eclipse.e4.core.contexts ....................... SUCCESS [  2.810 s]
[INFO] org.eclipse.equinox.util ........................... SUCCESS [  3.053 s]
[INFO] org.eclipse.equinox.ds ............................. SUCCESS [  3.640 s]
[INFO] org.eclipse.e4.core.services ....................... SUCCESS [  2.957 s]
[INFO] org.eclipse.e4.core.commands ....................... SUCCESS [  2.801 s]
[INFO] org.eclipse.e4.core.di.extensions .................. SUCCESS [  2.530 s]
[INFO] org.eclipse.e4.emf.xpath ........................... SUCCESS [  2.995 s]
[INFO] eclipse.platform.swt ............................... SUCCESS [  0.006 s]
[INFO] org.eclipse.swt .................................... SUCCESS [  1.527 s]
[INFO] org.eclipse.equinox.bidi ........................... SUCCESS [  3.267 s]
[INFO] org.eclipse.swt.gtk.linux.x86 ...................... SUCCESS [ 21.222 s]
[INFO] org.eclipse.swt.gtk.linux.x86_64 ................... SUCCESS [ 12.033 s]
[INFO] org.eclipse.swt.gtk.linux.ppc ...................... SUCCESS [ 14.630 s]
[INFO] org.eclipse.swt.gtk.linux.ppc64 .................... SUCCESS [ 10.968 s]
[INFO] org.eclipse.swt.gtk.linux.ppc64le .................. SUCCESS [ 10.913 s]
[INFO] org.eclipse.swt.gtk.linux.s390 ..................... SUCCESS [ 13.771 s]
[INFO] org.eclipse.swt.gtk.linux.s390x .................... SUCCESS [ 12.835 s]
[INFO] org.eclipse.swt.win32.win32.x86 .................... SUCCESS [ 16.183 s]
[INFO] org.eclipse.swt.win32.win32.x86_64 ................. SUCCESS [ 12.151 s]
[INFO] org.eclipse.swt.cocoa.macosx.x86_64 ................ SUCCESS [ 10.792 s]
[INFO] org.eclipse.swt.gtk.solaris.x86_64 ................. SUCCESS [ 10.311 s]
[INFO] org.eclipse.swt.gtk.solaris.sparcv9 ................ SUCCESS [ 10.134 s]
[INFO] org.eclipse.swt.gtk.hpux.ia64 ...................... SUCCESS [ 10.323 s]
[INFO] org.eclipse.swt.gtk.aix.ppc ........................ SUCCESS [  9.079 s]
[INFO] org.eclipse.swt.gtk.aix.ppc64 ...................... SUCCESS [  6.170 s]
[INFO] org.eclipse.jface .................................. SUCCESS [  4.260 s]
[INFO] org.eclipse.e4.ui.bindings ......................... SUCCESS [  3.016 s]
[INFO] org.eclipse.e4.ui.css.core ......................... SUCCESS [  3.567 s]
[INFO] org.eclipse.e4.ui.css.swt .......................... SUCCESS [  3.566 s]
[INFO] org.eclipse.e4.ui.css.swt.theme .................... SUCCESS [  2.775 s]
[INFO] org.eclipse.e4.ui.di ............................... SUCCESS [  2.763 s]
[INFO] org.eclipse.e4.ui.model.workbench .................. SUCCESS [  2.170 s]
[INFO] org.eclipse.equinox.event .......................... SUCCESS [  1.572 s]
[INFO] org.eclipse.e4.ui.services ......................... SUCCESS [  2.899 s]
[INFO] org.eclipse.e4.ui.widgets .......................... SUCCESS [  2.533 s]
[INFO] org.eclipse.e4.ui.workbench ........................ SUCCESS [  2.066 s]
[INFO] org.eclipse.jface.databinding ...................... SUCCESS [  2.391 s]
[INFO] org.eclipse.e4.ui.workbench3 ....................... SUCCESS [  2.650 s]
[INFO] org.eclipse.e4.ui.workbench.swt .................... SUCCESS [  3.611 s]
[INFO] org.eclipse.e4.ui.workbench.renderers.swt .......... SUCCESS [  3.514 s]
[INFO] org.eclipse.e4.ui.workbench.addons.swt ............. SUCCESS [  1.713 s]
[INFO] eclipse.platform.ua ................................ SUCCESS [  0.003 s]
[INFO] org.eclipse.help ................................... SUCCESS [  3.743 s]
[INFO] org.eclipse.equinox.security ....................... SUCCESS [  3.223 s]
[INFO] eclipse.platform.team .............................. SUCCESS [  0.006 s]
[INFO] org.eclipse.core.net ............................... SUCCESS [  2.993 s]
[INFO] eclipse.platform.debug ............................. SUCCESS [  0.003 s]
[INFO] org.eclipse.core.variables ......................... SUCCESS [  2.754 s]
[INFO] eclipse.platform ................................... SUCCESS [  0.004 s]
[INFO] org.eclipse.ant.core ............................... SUCCESS [  3.355 s]
[INFO] org.eclipse.equinox.http.servlet ................... SUCCESS [  1.767 s]
[INFO] org.eclipse.equinox.http.jetty ..................... SUCCESS [  2.736 s]
[INFO] org.eclipse.help.base .............................. SUCCESS [  1.992 s]
[INFO] org.eclipse.ui.workbench ........................... SUCCESS [  7.802 s]
[INFO] org.eclipse.ui ..................................... SUCCESS [  4.204 s]
[INFO] org.eclipse.ui.forms ............................... SUCCESS [  4.471 s]
[INFO] org.eclipse.ui.intro ............................... SUCCESS [  4.154 s]
[INFO] org.eclipse.help.ui ................................ SUCCESS [  4.208 s]
[INFO] org.eclipse.ui.cheatsheets ......................... SUCCESS [  3.918 s]
[INFO] org.eclipse.jdt .................................... SUCCESS [  0.038 s]
[INFO] eclipse.platform.resources ......................... SUCCESS [  0.003 s]
[INFO] org.eclipse.core.filesystem ........................ SUCCESS [  3.090 s]
[INFO] org.eclipse.core.resources ......................... SUCCESS [  5.960 s]
[INFO] org.eclipse.debug.core ............................. SUCCESS [  5.358 s]
[INFO] org.eclipse.compare.core ........................... SUCCESS [  3.350 s]
[INFO] eclipse.platform.text .............................. SUCCESS [  0.008 s]
[INFO] org.eclipse.text ................................... SUCCESS [  2.320 s]
[INFO] org.eclipse.team.core .............................. SUCCESS [  4.735 s]
[INFO] eclipse.jdt.core ................................... SUCCESS [  0.003 s]
[INFO] org.eclipse.jdt.core ............................... SUCCESS [  7.700 s]
[INFO] eclipse.jdt.debug .................................. SUCCESS [  0.003 s]
[INFO] org.eclipse.jdt.debug .............................. SUCCESS [  2.345 s]
[INFO] org.eclipse.jdt.launching .......................... SUCCESS [  2.187 s]
[INFO] org.eclipse.core.externaltools ..................... SUCCESS [  2.786 s]
[INFO] org.eclipse.ant.launching .......................... SUCCESS [  3.408 s]
[INFO] org.eclipse.equinox.p2.core ........................ SUCCESS [  3.040 s]
[INFO] org.eclipse.equinox.p2.metadata .................... SUCCESS [  2.228 s]
[INFO] org.eclipse.equinox.p2.repository .................. SUCCESS [  3.577 s]
[INFO] org.eclipse.equinox.p2.metadata.repository ......... SUCCESS [  3.222 s]
[INFO] org.eclipse.equinox.p2.engine ...................... SUCCESS [  3.863 s]
[INFO] org.eclipse.jface.text ............................. SUCCESS [  7.499 s]
[INFO] org.eclipse.ui.views ............................... SUCCESS [  3.591 s]
[INFO] org.eclipse.ui.ide ................................. SUCCESS [  4.031 s]
[INFO] org.eclipse.ui.workbench.texteditor ................ SUCCESS [  2.525 s]
[INFO] org.eclipse.core.filebuffers ....................... SUCCESS [  3.422 s]
[INFO] org.eclipse.ui.editors ............................. SUCCESS [  2.313 s]
[INFO] org.eclipse.ui.console ............................. SUCCESS [  3.669 s]
[INFO] org.eclipse.debug.ui ............................... SUCCESS [  3.536 s]
[INFO] org.eclipse.ui.externaltools ....................... SUCCESS [  3.044 s]
[INFO] org.eclipse.compare ................................ SUCCESS [  6.093 s]
[INFO] eclipse.jdt.ui ..................................... SUCCESS [  0.003 s]
[INFO] org.eclipse.ltk.core.refactoring ................... SUCCESS [  4.639 s]
[INFO] org.eclipse.jdt.core.manipulation .................. SUCCESS [  2.256 s]
[INFO] org.eclipse.ui.navigator ........................... SUCCESS [  4.714 s]
[INFO] org.eclipse.team.ui ................................ SUCCESS [  7.199 s]
[INFO] org.eclipse.ltk.ui.refactoring ..................... SUCCESS [  4.570 s]
[INFO] org.eclipse.search ................................. SUCCESS [  4.564 s]
[INFO] org.eclipse.ui.views.properties.tabbed ............. SUCCESS [  3.532 s]
[INFO] org.eclipse.ui.navigator.resources ................. SUCCESS [  3.732 s]
[INFO] org.eclipse.jdt.ui ................................. SUCCESS [  8.551 s]
[INFO] org.eclipse.jdt.debug.ui ........................... SUCCESS [  2.420 s]
[INFO] org.eclipse.equinox.frameworkadmin ................. SUCCESS [  2.807 s]
[INFO] org.eclipse.equinox.frameworkadmin.equinox ......... SUCCESS [  2.932 s]
[INFO] org.eclipse.equinox.simpleconfigurator ............. SUCCESS [  3.098 s]
[INFO] org.eclipse.equinox.simpleconfigurator.manipulator . SUCCESS [  2.737 s]
[INFO] org.eclipse.jdt.junit.runtime ...................... SUCCESS [  3.019 s]
[INFO] org.eclipse.jdt.junit.core ......................... SUCCESS [  2.119 s]
[INFO] org.eclipse.jdt.junit .............................. SUCCESS [  4.471 s]
[INFO] org.eclipse.ant.ui ................................. SUCCESS [  5.352 s]
[INFO] org.eclipse.jdt.apt.core ........................... SUCCESS [  2.229 s]
[INFO] org.eclipse.jdt.compiler.tool ...................... SUCCESS [  2.217 s]
[INFO] org.eclipse.jdt.compiler.apt ....................... SUCCESS [  2.310 s]
[INFO] org.eclipse.jdt.apt.pluggable.core ................. SUCCESS [  2.737 s]
[INFO] org.eclipse.jdt.apt.ui ............................. SUCCESS [  2.978 s]
[INFO] org.eclipse.equinox.p2.jarprocessor ................ SUCCESS [  3.135 s]
[INFO] org.eclipse.equinox.p2.artifact.repository ......... SUCCESS [  3.220 s]
[INFO] org.eclipse.equinox.p2.director .................... SUCCESS [  3.443 s]
[INFO] org.eclipse.equinox.p2.director.app ................ SUCCESS [  3.219 s]
[INFO] org.eclipse.equinox.p2.garbagecollector ............ SUCCESS [  2.655 s]
[INFO] org.eclipse.equinox.p2.publisher ................... SUCCESS [  2.958 s]
[INFO] org.eclipse.equinox.p2.publisher.eclipse ........... SUCCESS [  1.951 s]
[INFO] org.eclipse.equinox.p2.repository.tools ............ SUCCESS [  3.629 s]
[INFO] org.eclipse.equinox.p2.touchpoint.eclipse .......... SUCCESS [  3.354 s]
[INFO] org.eclipse.equinox.p2.updatesite .................. SUCCESS [  3.468 s]
[INFO] org.eclipse.update.configurator .................... SUCCESS [  1.470 s]
[INFO] eclipse.pde.build .................................. SUCCESS [  0.007 s]
[INFO] org.eclipse.pde.build .............................. SUCCESS [  4.253 s]
[INFO] eclipse.pde.ui ..................................... SUCCESS [  0.003 s]
[INFO] org.eclipse.pde.core ............................... SUCCESS [  6.576 s]
[INFO] org.eclipse.equinox.concurrent ..................... SUCCESS [  2.704 s]
[INFO] eclipse.platform.common ............................ SUCCESS [  0.004 s]
[INFO] org.eclipse.platform.doc.isv ....................... SUCCESS [ 52.589 s]
[INFO] org.eclipse.jdt.doc.isv ............................ SUCCESS [ 11.922 s]
[INFO] org.eclipse.jdt.doc.user ........................... SUCCESS [ 13.517 s]
[INFO] org.eclipse.jdt.annotation ......................... SUCCESS [  2.778 s]
[INFO] org.eclipse.jdt.annotation ......................... SUCCESS [  1.934 s]
[INFO] org.eclipse.jdt.junit4.runtime ..................... SUCCESS [  2.801 s]
[INFO] org.eclipse.jdt.launching.macosx ................... SUCCESS [  2.529 s]
[INFO] org.eclipse.jdt.launching.ui.macosx ................ SUCCESS [  2.669 s]
[INFO] org.eclipse.jdt .................................... SUCCESS [  0.194 s]
[INFO] eclipse.platform.releng ............................ SUCCESS [  0.003 s]
[INFO] org.eclipse.test.performance ....................... SUCCESS [  2.778 s]
[INFO] org.eclipse.ui.ide.application ..................... SUCCESS [  3.526 s]
[INFO] org.eclipse.equinox.launcher ....................... SUCCESS [  3.094 s]
[INFO] org.eclipse.jdt.core.tests.compiler ................ SUCCESS [  1.088 s]
[INFO] org.eclipse.jdt.compiler.tool.tests ................ SUCCESS [  1.033 s]
[INFO] org.eclipse.jdt.core.tests.builder ................. SUCCESS [  0.482 s]
[INFO] org.eclipse.jdt.compiler.apt.tests ................. SUCCESS [  0.380 s]
[INFO] org.eclipse.jdt.core.tests.model ................... SUCCESS [  5.129 s]
[INFO] eclipse.jdt.core.binaries .......................... SUCCESS [  0.003 s]
[INFO] org.eclipse.jdt.core.tests.binaries ................ SUCCESS [ 46.578 s]
[INFO] org.eclipse.jdt.core.tests.performance ............. SUCCESS [  0.450 s]
[INFO] org.eclipse.jdt.apt.pluggable.tests ................ SUCCESS [  1.078 s]
[INFO] org.eclipse.jdt.apt.tests .......................... SUCCESS [  0.665 s]
[INFO] org.eclipse.jdt.debug.tests ........................ SUCCESS [  1.322 s]
[INFO] org.eclipse.core.filebuffers.tests ................. SUCCESS [  0.897 s]
[INFO] org.eclipse.text.tests ............................. SUCCESS [  1.243 s]
[INFO] org.eclipse.jface.text.tests ....................... SUCCESS [  0.756 s]
[INFO] org.eclipse.jdt.ui.tests ........................... SUCCESS [ 10.640 s]
[INFO] org.eclipse.jdt.text.tests ......................... SUCCESS [  7.016 s]
[INFO] org.eclipse.ltk.core.refactoring.tests ............. SUCCESS [  1.141 s]
[INFO] org.eclipse.ltk.ui.refactoring.tests ............... SUCCESS [  0.942 s]
[INFO] org.eclipse.jdt.ui.tests.refactoring ............... SUCCESS [  9.893 s]
[INFO] org.eclipse.pde.build.tests ........................ SUCCESS [  1.804 s]
[INFO] org.eclipse.pde.api.tools .......................... SUCCESS [  7.396 s]
[INFO] org.eclipse.pde.api.tools.annotations .............. SUCCESS [  2.506 s]
[INFO] org.eclipse.pde.api.tools.ee.cdcfoundation10 ....... SUCCESS [  1.134 s]
[INFO] org.eclipse.pde.api.tools.ee.cdcfoundation11 ....... SUCCESS [  0.831 s]
[INFO] org.eclipse.pde.api.tools.ee.j2se12 ................ SUCCESS [  1.803 s]
[INFO] org.eclipse.pde.api.tools.ee.j2se13 ................ SUCCESS [  1.599 s]
[INFO] org.eclipse.pde.api.tools.ee.j2se14 ................ SUCCESS [  2.140 s]
[INFO] org.eclipse.pde.api.tools.ee.j2se15 ................ SUCCESS [  2.108 s]
[INFO] org.eclipse.pde.api.tools.ee.javase16 .............. SUCCESS [  2.457 s]
[INFO] org.eclipse.pde.api.tools.ee.javase17 .............. SUCCESS [  2.332 s]
[INFO] org.eclipse.pde.api.tools.ee.jre11 ................. SUCCESS [  1.149 s]
[INFO] org.eclipse.pde.api.tools.ee.osgiminimum10 ......... SUCCESS [  1.034 s]
[INFO] org.eclipse.pde.api.tools.ee.osgiminimum11 ......... SUCCESS [  1.028 s]
[INFO] org.eclipse.pde.api.tools.ee.osgiminimum12 ......... SUCCESS [  0.894 s]
[INFO] org.eclipse.pde.api.tools.ee.javase18 .............. SUCCESS [  3.210 s]
[INFO] org.eclipse.pde.api.tools.ee.feature ............... SUCCESS [  0.621 s]
[INFO] org.eclipse.equinox.p2.operations .................. SUCCESS [  4.339 s]
[INFO] org.eclipse.equinox.security.ui .................... SUCCESS [  4.549 s]
[INFO] org.eclipse.equinox.p2.ui .......................... SUCCESS [  3.513 s]
[INFO] org.eclipse.pde.launching .......................... SUCCESS [  4.598 s]
[INFO] org.eclipse.ui.views.log ........................... SUCCESS [  4.444 s]
[INFO] org.eclipse.ui.trace ............................... SUCCESS [  4.223 s]
[INFO] org.eclipse.pde.ui ................................. SUCCESS [ 17.379 s]
[INFO] org.eclipse.pde.api.tools.ui ....................... SUCCESS [  5.349 s]
[INFO] org.eclipse.pde.api.tools.tests .................... SUCCESS [ 31.448 s]
[INFO] org.eclipse.pde.ds.core ............................ SUCCESS [  3.961 s]
[INFO] org.eclipse.pde.ds.tests ........................... SUCCESS [  0.658 s]
[INFO] org.eclipse.pde.ds.ui .............................. SUCCESS [  4.463 s]
[INFO] org.eclipse.pde.ds.annotations ..................... SUCCESS [  4.104 s]
[INFO] org.eclipse.pde.ua.core ............................ SUCCESS [  4.101 s]
[INFO] org.eclipse.pde.ua.ui .............................. SUCCESS [  5.476 s]
[INFO] org.eclipse.pde.ua.tests ........................... SUCCESS [  0.245 s]
[INFO] org.eclipse.pde .................................... SUCCESS [  0.211 s]
[INFO] org.eclipse.pde.junit.runtime ...................... SUCCESS [  3.635 s]
[INFO] org.eclipse.pde.runtime ............................ SUCCESS [  4.869 s]
[INFO] org.eclipse.pde.ui.templates ....................... SUCCESS [  4.847 s]
[INFO] org.eclipse.jsch.core .............................. SUCCESS [  4.270 s]
[INFO] org.eclipse.team.cvs.core .......................... SUCCESS [  5.726 s]
[INFO] org.eclipse.pde.ui.tests ........................... SUCCESS [  5.253 s]
[INFO] org.eclipse.pde.doc.user ........................... SUCCESS [ 16.702 s]
[INFO] org.eclipse.pde .................................... SUCCESS [  1.634 s]
[INFO] org.eclipse.update.core ............................ SUCCESS [  1.833 s]
[INFO] org.eclipse.ant.tests.core ......................... SUCCESS [  0.969 s]
[INFO] org.eclipse.ant.tests.ui ........................... SUCCESS [  0.436 s]
[INFO] org.eclipse.platform ............................... SUCCESS [  0.177 s]
[INFO] org.eclipse.sdk .................................... SUCCESS [  0.115 s]
[INFO] org.eclipse.core.databinding.beans ................. SUCCESS [  4.947 s]
[INFO] org.eclipse.jface.examples.databinding ............. SUCCESS [  2.460 s]
[INFO] org.eclipse.ui.examples.contributions .............. SUCCESS [  1.546 s]
[INFO] org.eclipse.ui.examples.fieldassist ................ SUCCESS [  1.577 s]
[INFO] org.eclipse.ui.examples.multipageeditor ............ SUCCESS [  1.210 s]
[INFO] org.eclipse.ui.examples.propertysheet .............. SUCCESS [  1.838 s]
[INFO] org.eclipse.ui.examples.readmetool ................. SUCCESS [  1.687 s]
[INFO] org.eclipse.ui.examples.undo ....................... SUCCESS [  1.428 s]
[INFO] org.eclipse.ui.examples.views.properties.tabbed.article SUCCESS [  1.425 s]
[INFO] org.eclipse.e4.demo.contacts ....................... SUCCESS [  0.223 s]
[INFO] org.eclipse.jface.snippets ......................... SUCCESS [  0.514 s]
[INFO] org.eclipse.ui.examples.job ........................ SUCCESS [  0.105 s]
[INFO] org.eclipse.ui.examples.navigator .................. SUCCESS [  0.368 s]
[INFO] org.eclipse.ui.forms.examples ...................... SUCCESS [  0.131 s]
[INFO] org.eclipse.ui.examples.rcp.mail ................... SUCCESS [  0.084 s]
[INFO] org.eclipse.e4.ui.dialogs .......................... SUCCESS [  4.025 s]
[INFO] org.eclipse.e4.ui.progress ......................... SUCCESS [  0.583 s]
[INFO] org.eclipse.e4.ui.workbench.renderers.swt.cocoa .... SUCCESS [  3.756 s]
[INFO] org.eclipse.ui.browser ............................. SUCCESS [  4.663 s]
[INFO] org.eclipse.ui.monitoring .......................... SUCCESS [  4.203 s]
[INFO] org.eclipse.ui.themes .............................. SUCCESS [  1.501 s]
[INFO] org.eclipse.osgi.compatibility.state ............... SUCCESS [  0.266 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.x86 ......... SUCCESS [  0.981 s]
[INFO] org.eclipse.equinox.console ........................ SUCCESS [  0.162 s]
[INFO] org.eclipse.e4.ui.swt.gtk .......................... SUCCESS [  1.206 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.x86_64 ...... SUCCESS [  0.760 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.ppc ......... SUCCESS [  0.783 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.ppc64 ....... SUCCESS [  0.800 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.ppc64le ..... SUCCESS [  0.993 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.s390 ........ SUCCESS [  0.816 s]
[INFO] org.eclipse.equinox.launcher.gtk.linux.s390x ....... SUCCESS [  0.975 s]
[INFO] org.eclipse.equinox.launcher.win32.win32.x86 ....... SUCCESS [  0.604 s]
[INFO] org.eclipse.equinox.launcher.win32.win32.x86_64 .... SUCCESS [  0.550 s]
[INFO] org.eclipse.equinox.launcher.cocoa.macosx.x86_64 ... SUCCESS [  0.564 s]
[INFO] org.eclipse.equinox.launcher.gtk.solaris.x86_64 .... SUCCESS [  0.562 s]
[INFO] org.eclipse.equinox.launcher.gtk.solaris.sparcv9 ... SUCCESS [  0.661 s]
[INFO] org.eclipse.equinox.launcher.gtk.hpux.ia64 ......... SUCCESS [  0.663 s]
[INFO] org.eclipse.equinox.launcher.gtk.aix.ppc ........... SUCCESS [  1.480 s]
[INFO] org.eclipse.equinox.launcher.gtk.aix.ppc64 ......... SUCCESS [  1.439 s]
[INFO] org.eclipse.e4.rcp ................................. SUCCESS [ 10.746 s]
[INFO] org.eclipse.ui.win32 ............................... SUCCESS [  3.863 s]
[INFO] org.eclipse.ui.cocoa ............................... SUCCESS [  1.330 s]
[INFO] eclipse.platform.ui.tests .......................... SUCCESS [  0.003 s]
[INFO] org.eclipse.jface.tests.databinding.conformance .... SUCCESS [  1.167 s]
[INFO] org.eclipse.jface.tests.databinding ................ SUCCESS [  2.398 s]
[INFO] org.eclipse.ui.monitoring.tests .................... SUCCESS [  0.371 s]
[INFO] eclipse.platform.runtime.tests ..................... SUCCESS [  0.002 s]
[INFO] org.eclipse.core.tests.harness ..................... SUCCESS [  0.842 s]
[INFO] org.eclipse.ui.tests.harness ....................... SUCCESS [  0.974 s]
[INFO] org.eclipse.ui.tests ............................... SUCCESS [  4.989 s]
[INFO] org.eclipse.ui.tests.forms ......................... SUCCESS [  0.965 s]
[INFO] org.eclipse.ui.tests.navigator ..................... SUCCESS [  1.727 s]
[INFO] org.eclipse.ui.tests.performance ................... SUCCESS [  1.586 s]
[INFO] org.eclipse.ui.tests.rcp ........................... SUCCESS [  0.983 s]
[INFO] org.eclipse.ui.tests.views.properties.tabbed ....... SUCCESS [  1.448 s]
[INFO] org.eclipse.ui.ide.application.tests ............... SUCCESS [  0.392 s]
[INFO] org.eclipse.e4.ui.bindings.tests ................... SUCCESS [  0.652 s]
[INFO] org.eclipse.e4.core.commands.tests ................. SUCCESS [  0.651 s]
[INFO] org.eclipse.e4.ui.tests ............................ SUCCESS [  0.479 s]
[INFO] org.eclipse.e4.ui.tests.css.core ................... SUCCESS [  0.670 s]
[INFO] org.eclipse.e4.ui.tests.css.swt .................... SUCCESS [  0.924 s]
[INFO] org.eclipse.e4.ui.workbench.addons.swt.test ........ SUCCESS [  0.378 s]
[INFO] org.eclipse.platform.doc.user ...................... SUCCESS [ 16.431 s]
[INFO] org.eclipse.debug.examples.core .................... SUCCESS [  4.121 s]
[INFO] org.eclipse.debug.examples.memory .................. SUCCESS [  2.644 s]
[INFO] org.eclipse.debug.examples.mixedmode ............... SUCCESS [  2.649 s]
[INFO] org.eclipse.debug.examples.ui ...................... SUCCESS [  4.786 s]
[INFO] org.eclipse.debug.tests ............................ SUCCESS [  1.607 s]
[INFO] org.eclipse.core.filesystem.hpux.ia64 .............. SUCCESS [  0.550 s]
[INFO] org.eclipse.core.filesystem.linux.ppc .............. SUCCESS [  0.541 s]
[INFO] org.eclipse.core.filesystem.linux.ppc64 ............ SUCCESS [  0.438 s]
[INFO] org.eclipse.core.filesystem.linux.ppc64le .......... SUCCESS [  0.435 s]
[INFO] org.eclipse.core.filesystem.linux.x86 .............. SUCCESS [  0.482 s]
[INFO] org.eclipse.core.filesystem.linux.x86_64 ........... SUCCESS [  0.429 s]
[INFO] org.eclipse.core.filesystem.macosx ................. SUCCESS [  0.432 s]
[INFO] org.eclipse.core.filesystem.solaris.sparc .......... SUCCESS [  0.068 s]
[INFO] org.eclipse.core.filesystem.win32.x86 .............. SUCCESS [  0.546 s]
[INFO] org.eclipse.core.filesystem.win32.x86_64 ........... SUCCESS [  0.548 s]
[INFO] org.eclipse.core.filesystem.aix.ppc ................ SUCCESS [  0.495 s]
[INFO] org.eclipse.core.filesystem.aix.ppc64 .............. SUCCESS [  0.432 s]
[INFO] org.eclipse.core.resources.win32.x86 ............... SUCCESS [  0.846 s]
[INFO] org.eclipse.core.resources.win32.x86_64 ............ SUCCESS [  0.675 s]
[INFO] org.eclipse.core.tests.filesystem.feature .......... SUCCESS [  0.067 s]
[INFO] org.eclipse.core.tests.resources ................... SUCCESS [  3.324 s]
[INFO] org.eclipse.core.tests.resources.saveparticipant1 .. SUCCESS [  0.095 s]
[INFO] org.eclipse.core.tests.resources.saveparticipant2 .. SUCCESS [  0.107 s]
[INFO] org.eclipse.core.tests.resources.saveparticipant3 .. SUCCESS [  0.095 s]
[INFO] org.eclipse.core.tests.resources.saveparticipant ... SUCCESS [  0.120 s]
[INFO] org.eclipse.equinox.core.feature ................... SUCCESS [  0.040 s]
[INFO] org.eclipse.core.runtime.feature ................... SUCCESS [  0.058 s]
[INFO] com.google.code.atinject.tck ....................... SUCCESS [  0.601 s]
[INFO] org.eclipse.core.expressions.tests ................. SUCCESS [  0.696 s]
[INFO] org.eclipse.core.tests.runtime ..................... SUCCESS [  2.300 s]
[INFO] org.eclipse.e4.core.tests .......................... SUCCESS [  1.090 s]
[INFO] org.eclipse.core.contenttype.tests ................. SUCCESS [  0.097 s]
[INFO] org.eclipse.swt.fragments.localbuild ............... SUCCESS [  0.424 s]
[INFO] org.eclipse.swt.tools.base ......................... SUCCESS [  0.808 s]
[INFO] org.eclipse.swt.tools.spies ........................ SUCCESS [  0.858 s]
[INFO] org.eclipse.swt.tools .............................. SUCCESS [  1.583 s]
[INFO] org.eclipse.swt.examples ........................... SUCCESS [  5.023 s]
[INFO] org.eclipse.swt.examples.browser.demos ............. SUCCESS [  2.199 s]
[INFO] org.eclipse.swt.examples.launcher .................. SUCCESS [  1.378 s]
[INFO] org.eclipse.swt.examples.ole.win32 ................. SUCCESS [  1.251 s]
[INFO] org.eclipse.swt.examples.views ..................... SUCCESS [  1.132 s]
[INFO] org.eclipse.swt.tests.fragments.feature ............ SUCCESS [  0.287 s]
[INFO] org.eclipse.swt.tests .............................. SUCCESS [ 10.925 s]
[INFO] org.eclipse.swt.tools.feature ...................... SUCCESS [  0.828 s]
[INFO] org.eclipse.swt.gtk.linux.aarch64 .................. SUCCESS [ 11.621 s]
[INFO] org.eclipse.swt.gtk.linux.arm ...................... SUCCESS [ 13.451 s]
[INFO] eclipse.platform.swt.binaries ...................... SUCCESS [  0.003 s]
[INFO] org.eclipse.compare.win32 .......................... SUCCESS [  3.988 s]
[INFO] org.eclipse.jsch.ui ................................ SUCCESS [  4.121 s]
[INFO] org.eclipse.team.cvs.ssh2 .......................... SUCCESS [  3.695 s]
[INFO] org.eclipse.team.cvs.ui ............................ SUCCESS [  8.280 s]
[INFO] org.eclipse.ui.net ................................. SUCCESS [  3.992 s]
[INFO] org.eclipse.compare.examples ....................... SUCCESS [  1.002 s]
[INFO] org.eclipse.compare.examples.xml ................... SUCCESS [  1.747 s]
[INFO] org.eclipse.team.examples.filesystem ............... SUCCESS [  2.182 s]
[INFO] org.eclipse.cvs .................................... SUCCESS [  0.059 s]
[INFO] org.eclipse.cvs .................................... SUCCESS [  0.130 s]
[INFO] org.eclipse.core.net.linux.x86 ..................... SUCCESS [  0.823 s]
[INFO] org.eclipse.core.net.linux.x86_64 .................. SUCCESS [  0.623 s]
[INFO] org.eclipse.core.net.win32.x86 ..................... SUCCESS [  0.856 s]
[INFO] org.eclipse.core.net.win32.x86_64 .................. SUCCESS [  0.641 s]
[INFO] org.eclipse.compare.tests .......................... SUCCESS [  1.038 s]
[INFO] org.eclipse.core.tests.net ......................... SUCCESS [  3.312 s]
[INFO] org.eclipse.jsch.tests ............................. SUCCESS [  3.150 s]
[INFO] org.eclipse.team.tests.core ........................ SUCCESS [  0.829 s]
[INFO] org.eclipse.team.tests.cvs.core .................... SUCCESS [  5.017 s]
[INFO] org.eclipse.search.tests ........................... SUCCESS [  0.872 s]
[INFO] org.eclipse.ui.editors.tests ....................... SUCCESS [  0.669 s]
[INFO] org.eclipse.ui.workbench.texteditor.tests .......... SUCCESS [  0.662 s]
[INFO] org.eclipse.ui.examples.javaeditor ................. SUCCESS [  1.633 s]
[INFO] org.eclipse.ui.genericeditor ....................... SUCCESS [  2.129 s]
[INFO] org.eclipse.ui.genericeditor.tests ................. SUCCESS [  0.715 s]
[INFO] org.eclipse.ui.genericeditor.examples .............. SUCCESS [  1.162 s]
[INFO] org.eclipse.equinox.http.registry .................. SUCCESS [  3.946 s]
[INFO] org.eclipse.equinox.jsp.jasper ..................... SUCCESS [  3.757 s]
[INFO] org.eclipse.equinox.jsp.jasper.registry ............ SUCCESS [  3.522 s]
[INFO] org.eclipse.help.webapp ............................ SUCCESS [ 37.170 s]
[INFO] org.eclipse.ui.intro.quicklinks .................... SUCCESS [  3.977 s]
[INFO] org.eclipse.ui.intro.universal ..................... SUCCESS [  6.543 s]
[INFO] org.eclipse.ua.tests ............................... SUCCESS [  2.770 s]
[INFO] org.eclipse.ua.tests.doc ........................... SUCCESS [  0.738 s]
[INFO] org.eclipse.ui.intro.quicklinks.examples ........... SUCCESS [  0.071 s]
[INFO] org.eclipse.ui.intro.solstice.examples ............. SUCCESS [  0.071 s]
[INFO] eclipse.platform.ui.tools .......................... SUCCESS [  0.003 s]
[INFO] org.eclipse.e4.tools ............................... SUCCESS [  1.541 s]
[INFO] org.eclipse.e4.tools.services ...................... SUCCESS [  0.088 s]
[INFO] org.eclipse.e4.tools.compat ........................ SUCCESS [  1.048 s]
[INFO] org.eclipse.e4.tools.emf.ui ........................ SUCCESS [  4.598 s]
[INFO] org.eclipse.e4.tools.emf.editor3x .................. SUCCESS [  1.216 s]
[INFO] org.eclipse.e4.tools.jdt.templates ................. SUCCESS [  1.031 s]
[INFO] org.eclipse.e4.core.tools.feature .................. SUCCESS [  0.123 s]
[INFO] org.eclipse.equinox.cm ............................. SUCCESS [  2.635 s]
[INFO] org.eclipse.equinox.coordinator .................... SUCCESS [  2.782 s]
[INFO] org.eclipse.equinox.device ......................... SUCCESS [  3.745 s]
[INFO] org.eclipse.equinox.io ............................. SUCCESS [  3.734 s]
[INFO] org.eclipse.equinox.ip ............................. SUCCESS [  3.798 s]
[INFO] org.eclipse.equinox.metatype ....................... SUCCESS [  2.873 s]
[INFO] org.eclipse.equinox.useradmin ...................... SUCCESS [  3.702 s]
[INFO] org.eclipse.equinox.wireadmin ...................... SUCCESS [  3.752 s]
[INFO] org.eclipse.equinox.compendium.sdk ................. SUCCESS [  0.055 s]
[INFO] org.eclipse.equinox.supplement ..................... SUCCESS [  5.053 s]
[INFO] org.eclipse.equinox.console.jaas.fragment .......... SUCCESS [  0.529 s]
[INFO] org.eclipse.equinox.console.ssh .................... SUCCESS [  0.127 s]
[INFO] org.eclipse.equinox.transforms.xslt ................ SUCCESS [  3.449 s]
[INFO] org.eclipse.equinox.transforms.hook ................ SUCCESS [  2.629 s]
[INFO] org.eclipse.osgi.util .............................. SUCCESS [  1.105 s]
[INFO] org.eclipse.equinox.region ......................... SUCCESS [  1.476 s]
[INFO] org.eclipse.equinox.weaving.hook ................... SUCCESS [  4.042 s]
[INFO] org.eclipse.equinox.weaving.caching ................ SUCCESS [  3.485 s]
[INFO] org.eclipse.equinox.weaving.caching.j9 ............. SUCCESS [  3.743 s]
[INFO] org.eclipse.equinox.security.win32.x86 ............. SUCCESS [  3.764 s]
[INFO] org.eclipse.equinox.security.win32.x86_64 .......... SUCCESS [  3.991 s]
[INFO] org.eclipse.equinox.security.macosx ................ SUCCESS [  3.711 s]
[INFO] org.eclipse.equinox.core.sdk ....................... SUCCESS [  0.079 s]
[INFO] org.eclipse.equinox.executable ..................... SUCCESS [ 13.043 s]
[INFO] org.eclipse.equinox.servletbridge .................. SUCCESS [  2.948 s]
[INFO] org.eclipse.equinox.http.servletbridge ............. SUCCESS [  3.422 s]
[INFO] org.eclipse.equinox.p2.console ..................... SUCCESS [  2.708 s]
[INFO] org.eclipse.equinox.p2.touchpoint.natives .......... SUCCESS [  4.159 s]
[INFO] org.eclipse.equinox.p2.transport.ecf ............... SUCCESS [  3.731 s]
[INFO] org.eclipse.equinox.server.core .................... SUCCESS [  0.040 s]
[INFO] org.eclipse.equinox.server.jetty ................... SUCCESS [  0.039 s]
[INFO] org.eclipse.equinox.server.p2 ...................... SUCCESS [  0.064 s]
[INFO] org.eclipse.equinox.serverside.sdk ................. SUCCESS [  0.060 s]
[INFO] org.eclipse.equinox.sdk ............................ SUCCESS [  0.060 s]
[INFO] org.eclipse.equinox.server.simple .................. SUCCESS [  0.034 s]
[INFO] org.eclipse.equinox.bidi.tests ..................... SUCCESS [  0.664 s]
[INFO] org.eclipse.equinox.cm.test ........................ SUCCESS [  0.060 s]
[INFO] org.eclipse.osgi.compatibility.plugins ............. SUCCESS [  0.729 s]
[INFO] org.eclipse.osgi.tests ............................. SUCCESS [  7.185 s]
[INFO] org.eclipse.equinox.compendium.tests ............... SUCCESS [  0.128 s]
[INFO] org.eclipse.equinox.ds.tests ....................... SUCCESS [  2.161 s]
[INFO] org.eclipse.equinox.security.tests ................. SUCCESS [  0.610 s]
[INFO] org.eclipse.equinox.http.servlet.tests ............. SUCCESS [  0.355 s]
[INFO] org.eclipse.equinox.servletbridge.template ......... SUCCESS [  0.064 s]
[INFO] org.eclipse.equinox.slf4j.stub ..................... SUCCESS [  0.065 s]
[INFO] org.eclipse.osgi.compatibility.plugins.feature ..... SUCCESS [  0.519 s]
[INFO] ie.wombat.jbdiff ................................... SUCCESS [  0.197 s]
[INFO] ie.wombat.jbdiff.test .............................. SUCCESS [  0.193 s]
[INFO] org.eclipse.equinox.frameworkadmin.test ............ SUCCESS [  4.385 s]
[INFO] org.eclipse.equinox.p2.sar ......................... SUCCESS [  3.157 s]
[INFO] org.eclipse.equinox.p2.artifact.optimizers ......... SUCCESS [  2.858 s]
[INFO] org.eclipse.equinox.p2.artifact.processors ......... SUCCESS [  2.936 s]
[INFO] org.eclipse.equinox.p2.directorywatcher ............ SUCCESS [  3.714 s]
[INFO] org.eclipse.equinox.p2.discovery ................... SUCCESS [  3.672 s]
[INFO] org.eclipse.equinox.p2.discovery.compatibility ..... SUCCESS [  3.855 s]
[INFO] org.eclipse.equinox.p2.extensionlocation ........... SUCCESS [  4.873 s]
[INFO] org.eclipse.equinox.p2.installer ................... SUCCESS [  3.905 s]
[INFO] org.eclipse.equinox.p2.reconciler.dropins .......... SUCCESS [  4.067 s]
[INFO] org.eclipse.equinox.p2.ui.importexport ............. SUCCESS [  4.353 s]
[INFO] org.eclipse.equinox.p2.updatechecker ............... SUCCESS [  3.625 s]
[INFO] org.eclipse.equinox.p2.ui.admin .................... SUCCESS [  4.159 s]
[INFO] org.eclipse.equinox.p2.ui.admin.rcp ................ SUCCESS [  3.649 s]
[INFO] org.eclipse.equinox.p2.ui.discovery ................ SUCCESS [  2.809 s]
[INFO] org.eclipse.equinox.p2.core.feature ................ SUCCESS [  2.081 s]
[INFO] org.eclipse.equinox.p2.extras.feature .............. SUCCESS [  0.114 s]
[INFO] org.eclipse.equinox.p2.discovery.feature ........... SUCCESS [  0.120 s]
[INFO] org.eclipse.equinox.p2.ui.sdk ...................... SUCCESS [  4.505 s]
[INFO] org.eclipse.equinox.p2.ui.sdk.scheduler ............ SUCCESS [  4.158 s]
[INFO] org.eclipse.equinox.p2.sdk ......................... SUCCESS [  0.151 s]
[INFO] org.eclipse.equinox.p2.rcp.feature ................. SUCCESS [  0.108 s]
[INFO] org.eclipse.equinox.p2.user.ui ..................... SUCCESS [  0.111 s]
[INFO] org.eclipse.equinox.p2.tests.reconciler.product .... SUCCESS [ 57.128 s]
[INFO] org.eclipse.equinox.p2.tests.verifier .............. SUCCESS [  0.753 s]
[INFO] org.eclipse.equinox.p2.tests ....................... SUCCESS [ 57.869 s]
[INFO] org.eclipse.equinox.p2.tests.discovery ............. SUCCESS [  3.297 s]
[INFO] org.eclipse.equinox.p2.tests.ui .................... SUCCESS [  1.143 s]
[INFO] rt.equinox.p2 ...................................... SUCCESS [  0.000 s]
[INFO] org.eclipse.ant.optional.junit ..................... SUCCESS [  0.409 s]
[INFO] org.eclipse.pde.tools.versioning ................... SUCCESS [  0.864 s]
[INFO] org.eclipse.rcp .................................... SUCCESS [  0.049 s]
[INFO] org.eclipse.releng.tools ........................... SUCCESS [  1.163 s]
[INFO] org.eclipse.releng.tests ........................... SUCCESS [  0.736 s]
[INFO] org.eclipse.sdk.examples ........................... SUCCESS [  0.051 s]
[INFO] org.eclipse.sdk.tests .............................. SUCCESS [  0.049 s]
[INFO] org.eclipse.test ................................... SUCCESS [  1.894 s]
[INFO] org.eclipse.test.performance.win32 ................. SUCCESS [  0.615 s]
[INFO] org.eclipse.help ................................... SUCCESS [ 11.333 s]
[INFO] org.eclipse.rcp .................................... SUCCESS [  0.093 s]
[INFO] org.eclipse.platform ............................... SUCCESS [  4.223 s]
[INFO] org.eclipse.releng.tools ........................... SUCCESS [  0.517 s]
[INFO] org.eclipse.sdk .................................... SUCCESS [  0.111 s]
[INFO] org.eclipse.sdk.examples ........................... SUCCESS [  0.121 s]
[INFO] org.eclipse.test ................................... SUCCESS [  0.537 s]
[INFO] org.eclipse.sdk.tests .............................. SUCCESS [  0.214 s]
[INFO] eclipse.platform.releng.tychoeclipsebuilder ........ SUCCESS [  0.003 s]
[INFO] org.eclipse.rcp.configuration ...................... SUCCESS [  0.210 s]
[INFO] org.eclipse.rt.osgistarterkit.product .............. SUCCESS [  8.175 s]
[INFO] equinox-sdk ........................................ SUCCESS [ 22.992 s]
[INFO] org.eclipse.rcp.id ................................. SUCCESS [ 14.033 s]
[INFO] org.eclipse.rcp.sdk.id ............................. SUCCESS [ 20.919 s]
[INFO] org.eclipse.platform.ide ........................... SUCCESS [01:49 min]
[INFO] org.eclipse.platform.sdk ........................... SUCCESS [01:02 min]
[INFO] org.eclipse.sdk.ide ................................ SUCCESS [05:35 min]
[INFO] eclipse-junit-tests ................................ SUCCESS [ 24.916 s]
[INFO] eclipse.platform.repository ........................ SUCCESS [05:25 min]
[INFO] platform-aggregator ................................ SUCCESS [  0.000 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43:26 min
[INFO] Finished at: 2017-01-05T21:31:33+01:00
[INFO] Final Memory: 875M/1717M
[INFO] ------------------------------------------------------------------------
{% endhighlight %}



<H1>Motivations for Building Eclipse Platform ??</H1>

<BR/>

Initial motivations, from purely technical perspectives:
<ul>
<li>Eclipse is 100% Open-Source, 100% java (+native SWT), with high quality code standard .. A model of perfection to read for every professional developper</li>
<li>As a developper you can have access to ALL parts, there is no proprietary parts or priviledges for core / non-core plugins : "all players plays with the same rules"  ... unlike so many proprietary softwares</li>
<li>Eclipse is a wonderfull User Interface (SWT respect your OS, and does not re-implement a langage specific widget toolkit, it just use native widgets, unlike so many other wrong libraries)</li> 
<li>Eclipse is a wonderfull platform for installing plugins (OSGI principles + ClassLoader runtime loading and isolation)</li>
<li>Eclipse is a wonderfull platform for developper, to create new plugins (PDE, tycho)</li>
<li>Eclipse has a very beautifull internal core design, in particular with the IAdapter / AdapterFactory / PlatformRegistry for defining extensions points</li>
<li>Eclipse JDT is a fantastic java development env to use, and even to extends as a Java-AST toolkit to define your own refactoring actionss and code analysis tools</li>
<li>Eclipse plugins have clear separations between public/internal parts, and between *.core and *.ui ... and most eclipse contributors also respect this pretty well, so it is easy to find code in plugins and dive into plugin code</li>
</ul>

 
Unfortunatly.. I would like few annoying things to be solved in Eclipse! 
<ul>
<li>Lockings, freeze, slowness, ...</li>
<li>typing a letter on a keyboard should react immediatly as a letter appearing on screen ... not as a wait cursor for doing/locking internal things</li>
<li>Killing "Progress Tasks" does not kill tasks</li>
<li>stupid internal things like "Validating JPA", "indexing", should not interfer with user</li>
<li>Since version neon (?), I have "Internal StackOverflowError" from time to time, which freeze my CPU to 100%</li>
<li>Files should be automatically refreshed by default (not by click or with an insulting error message)</li>
<li>Eclipse really miss core features like decent JSON / Yaml / JavaScript / TypeScript / Kotlin editors, What a pity !!</li>
<li>Eclipse should delegate as much as possible to a maven background compiler, to really do what maven compiles... example: change src/main/resources file ... Eclipse should (ask maven to) simply re-copy the file to target/classes !!</li>
</ul>

So, this is a challenge to compile Eclipse, take a bug from Eclipse BugZilla, reproduce it, debug it, solve it, pull-request it, make it be accepted by community!



