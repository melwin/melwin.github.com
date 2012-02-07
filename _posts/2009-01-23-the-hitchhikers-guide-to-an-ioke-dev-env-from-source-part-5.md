--- 
wordpress_id: "148"
layout: post
title: "The Hitchhiker's Guide to an Ioke Dev Env From Source (part 5: Ioke and the REPL)"
wordpress_url: http://martin.elwin.com/blog/?p=148
---
This is the fifth part in a series of posts for non-experts about setting up an Ioke development environment on Linux. Please see the previous posts to start at the beginning:

<ul>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-1/">Part 1: Git</a></li>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-2/">Part 2: Emacs</a></li>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-3/">Part 3: emacs-starter-kit</a></li>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-4/">Part 4: Java and Ant</a></li>
</ul>

This time we will finally get the Ioke source code, build it, and test the Ioke <a href="http://en.wikipedia.org/wiki/REPL">REPL</a>!

### Ioke

Ioke uses <a href="http://git-scm.com">Git</a> as the source code versioning tool, with a <a href="http://github.com/olabini/ioke">public repository</a> set up on <a href="http://github.com/">GitHub</a> (soon I'll be linking like Jeff Atwood).

So - let's get Ioke using Git by cloning Ola's repository:

{% highlight sh %}
~/bin$ cd ~/work
~/work$ git clone git://github.com/olabini/ioke
Initialized empty Git repository in /home/melwin/work/ioke/.git/
remote: Counting objects: 9221, done.
remote: Compressing objects: 100% (2941/2941), done.
remote: Total 9221 (delta 5782), reused 8656 (delta 5351)
Receiving objects: 100% (9221/9221), 45.95 MiB | 399 KiB/s, done.
Resolving deltas: 100% (5782/5782), done.
{% endhighlight %}

Great - let's quickly move on to the build step:

{% highlight sh %}
~/work$ cd ioke
~/work/ioke$ ant
...
     [java] 2606 examples, 0 failures
jar:
      [jar] Building jar: /home/melwin/work/ioke/lib/ioke.jar

BUILD SUCCESSFUL
Total time: 22 seconds
{% endhighlight %}

Running just `ant` will execute the default target in the `build.xml` file. This target will make sure Ioke is compiled and the test suite is run.

At the end is a summary of the test results - if these are not all successful you might have a problem with the environment or you might have checked out a broken build from the repository.

### The Ioke REPL

Now all is installed and Ioke is compiled, so let's fire up the REPL. This is done by just running the main Ioke binary:

{% highlight sh %}
~/work/ioke/$ bin/ioke
iik>
{% endhighlight %}

When doing this we get the `iik>` REPL prompt. From here we can execute most Ioke code. Let's try to print a string:

{% highlight ioke %}
iik> "Hello World!" println
Hello World!
+> nil

iik>
{% endhighlight %}

Here we create a Text (Ioke's string type) literal and send it the message `println`, which will cause the text to be printed to the console. The return type of `println` is `nil` (similar to null), which is printed by the REPL itself. After that we get the prompt back and can execute another line.

Let's try a little more complex example - the factorial:
{% highlight ioke %}
iik> fact = method(n, if(n > 1, n * fact(n pred), 1))
+> fact:method(n, if(n >(1), n *(fact(n pred)), 1))
{% endhighlight %}

Note how the REPL prints the method definition in a canonical form - it makes it obvious how Ioke parses the code, what are messages and what are arguments.

Let's run our new factorial function:

{% highlight ioke %}
iik> fact(3)
+> 6

iik> fact(10)
+> 3628800

iik> fact(50)
+> 30414093201713378043612608166064768844377641568960512000000000000
{% endhighlight %}

Ioke handles arbitrarily big numbers, so we don't need to think about what fits in a certain number of bits.

The Iik REPL also comes with a simple debugger, which helps with handling conditions. For instance, let's try to run the fact method, passing a message which doesn't mean anything (yet):

{% highlight ioke %}
iik> fact(f)
*** - couldn't find cell 'f' on 'Ground_0xC360E7' (Condition Error NoSuchCell)
 f                                                [<init>:1:5]
The following restarts are available:
 0: storeValue           (Store value for: f)
 1: useValue             (Use value for: f)
 2: abort                (restart: abort)
 3: quit                 (restart: quit)

 dbg:1> 1
  dbg:1:newValue> 5
  +> 5

+> 120
{% endhighlight %}

The above output shows that when we try to reference something that doesn't exist we get a chance to provide that value. Here I decided to provide a value using the useValue restart and entered the value 5. This let the method continue executing and the result was printed.

To exit the REPL, just type: `exit`

Next up are the Emacs modes for Ioke - stay tuned!

/M
