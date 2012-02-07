--- 
wordpress_id: "125"
layout: blog_post
title: "The Hitchhiker's Guide to an Ioke Dev Env From Source (part 4: Java+Ant)"
wordpress_url: http://martin.elwin.com/blog/?p=125
---
This is the fourth part in a series of posts for non-experts about setting up an Ioke development environment on Linux. Please see the previous posts to start at the beginning:

<ul>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-1/">Part 1: Git</a></li>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-2/">Part 2: Emacs</a></li>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-3/">Part 3: emacs-starter-kit</a></li>
</ul>

It took a bit longer than anticipated to get to this point, as I got side tracked with the <a href="http://martin.elwin.com/blog/2009/01/type-checking-in-ioke-java-methods/">Ioke type checking activities</a>. But now when <a href="http://ioke.org">Ioke S</a> is out we can continue with the series. So! We're getting closer... Only a few more things to go.

### Installing Java

As the current version of Ioke is implemented to run on a Java Virtual Machine, we need to get hold of one of those to be able to continue. The JVM brand we will install here is the latest stable one  produced by Sun.

What we need is Java SE Development Kit 6u11 for Linux, Multi-language, and the the filename for this is `jdk-6u11-linux-i586.bin`. Note that this is <em>not</em> the `.rpm.bin`, which is also available.

Download the file from <a href="http://java.sun.com/javase/downloads/index.jsp">http://java.sun.com/javase/downloads/index.jsp</a> (I'm sure you can find it) and put it in the `~/work` directory.

To unpack the file, let's make it executable and then run it:

{% highlight sh %}
~$ cd ~/work
~/work$ cd ~/work                   
~/work$ chmod a+x jdk-6u11-linux-i586.bin 
~/work$ ./jdk-6u11-linux-i586.bin         
... lots of legalese here...
{% endhighlight %}

Once the license agreement is accepted, the JVM will unpack itself into the directory `~/work/jdk1.6.0_11`. Let's move it to where we want it and add `java` and `javac` to the `~/bin` directory as before:

{% highlight sh %}
~/work$ mv jdk1.6.0_11 ../opt
~/work$ cd ~/bin
~/bin$ ln -s ../opt/jdk1.6.0_11/bin/java
~/bin$ ln -s ../opt/jdk1.6.0_11/bin/javac
{% endhighlight %}

And test it:

{% highlight sh %}
~/bin$ java -version
java version "1.6.0_11"
Java(TM) SE Runtime Environment (build 1.6.0_11-b03)
Java HotSpot(TM) Client VM (build 11.0-b16, mixed mode, sharing)
{% endhighlight %}

That done - let's move on to Ant.

### Ant

Ioke uses <a href="http://ant.apache.org">Apache Ant</a> to handle the build process, so to build Ioke we need to get Ant installed.

Let's download the Ant distributable package and unpack it in one go - as we did with Git:

{% highlight sh %}
~/bin$ cd ~/opt
~/opt$ wget -O - http://www.apache.org/dist/ant/binaries/apache-ant-1.7.1-bin.tar.bz2 | tar xjv
</lang>

And add it to the ~/bin directory again so it's on the path:

<pre lang="sh">
~/opt$ cd ~/bin
~/bin$ ln -s ../opt/apache-ant-1.7.1/bin/ant
{% endhighlight %}

Let's try it out:

{% highlight sh %}
~/bin$ ant -version
Apache Ant version 1.7.1 compiled on June 27 2008
{% endhighlight %}

Looking good!

Soon to come: How to get the Ioke source and build it.

/M

<strong>Update:</strong> <a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-5/">Part 5: Ioke and the REPL</a>
