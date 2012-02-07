--- 
wordpress_id: "26"
layout: post
title: Clustering Scala Actors with Oracle Coherence for Fun and Profit
wordpress_url: http://martin.elwin.com/blog/?p=26
---
[Disclaimer: I work for Oracle.]

Although I haven't used it too much yet, Scala is definitely one of the languages I find most interesting right now. Many customers I work with are heavily Java focused, and getting a more flexible and powerful language with superb Java interoperability to use on the JVM feels very liberating. Now I just need to convince the customers that Scala is the future... :] But if <a href="http://www.adam-bien.com/roller/abien/entry/java_net_javaone_which_programming">Gosling likes it</a> it must be good, right?

A few months ago Jonas Bonér wrote about how <a href="http://jonasboner.com/2008/01/25/clustering-scala-actors-with-terracotta/">Scala actors can be clustered with Terracotta</a>. I really enjoyed the article and I think the idea of distributed, redundant actors is very appealing. The actor paradigm is a nice way of developing concurrent applications (see the intro in Jonas' blog entry for more info) and if we can liberate the actors from the confines of a single JVM and easily distribute them over multiple hosts - all the better.

### Clustering with Coherence

I'm not going to compare Coherence and Terracotta here. In short, <a href="http://www.oracle.com/technology/products/coherence/index.html">Coherence</a> provides, among other things, a distributed caching and code execution mechanism without single points of failure. Coherence can be downloaded for evaluation purposes from the Oracle web site.

The idea I wanted to try was to have Scala actors that store their state in the distributed Coherence cache and run as normal Scala actors on a single node in the cluster at a time. The node the actor runs on should be allocated by Coherence and if the node fails, the actor should be automatically started on another node with maintained state and without any lost messages.

Also, I wanted this to work as similarly to normal Scala actors as possible with compatibility between the two.

### The Result

Before investigating the proof of concept solution, let's look at the result and what it gives us.

Here's a simple test application with a normal Scala actor. It uses the recommended way of creating actors with `actor { ... }`:

{% highlight scala %}
package coherencetest1

import java.util.Date

import scala.actors.Actor._

object ActorTest {
  def main(args : Array[String]) : Unit = {

    actor {
      var pings : Int = 0
      
      println("Actor started.")
      
      self ! ('ping, 1)
      
      loop {
        react {
        case ('ping, i : Int) =>
          pings = pings + 1
          println(new Date + " - Got ping: " + i + " Total pings: " + pings)
          
          Thread.sleep(1000)

          self ! ('ping, i+1)
        }
      }
    }

  }
}
{% endhighlight %}

When this code is run, a simple actor that sends a message to itself is created and started. It sleeps for 1 second to pace the execution and to simulate a task that takes time to perform (of course, normally you shouldn't sleep in real actors as you tie up the thread).

When the code is run, the following is displayed:

{% highlight text %}
Actor started.
Sun Jun 29 15:57:16 CEST 2008 - Got ping: 1 Total pings: 1
Sun Jun 29 15:57:17 CEST 2008 - Got ping: 2 Total pings: 2
Sun Jun 29 15:57:18 CEST 2008 - Got ping: 3 Total pings: 3
Sun Jun 29 15:57:19 CEST 2008 - Got ping: 4 Total pings: 4
Sun Jun 29 15:57:20 CEST 2008 - Got ping: 5 Total pings: 5
Sun Jun 29 15:57:21 CEST 2008 - Got ping: 6 Total pings: 6
...
{% endhighlight %}

Nothing too fancy, but a decent test case for our actor distribution. An important aspect of this actor is that it defines a local variable `pings` and prints a message in the initialization part, before the `loop` and `react`. The value of the local var must be maintained and the initialization code must only be run once, and not when an actor is started on a new node after a failure.

Let's make it a distributed Actor:

{% highlight scala %}
package coherencetest1

import java.util.Date

import scala.actors.coherence.CoActor._

@serializable
object DactorTest {
  def main(args : Array[String]) : Unit = {

    dactor {
      var pings : Int = 0
      
      println("Actor started.")
      
      self ! ('ping, 1)
      
      loop {
        react {
        case ('ping, i : Int) =>
          pings = pings + 1
          println(new Date + " - Got ping: " + i + " Total pings: " + pings)
          
          Thread.sleep(1000)

          self ! ('ping, i+1)
        }
      }
    }

  }
}
{% endhighlight %}

What have we done here? Three things:

<ol>
	<li>Import `scala.actors.coherence.CoActor._` instead of `scala.actors.Actor._`</li>
	<li>Made the <em>application object</em> serializable</li>
	<li>Create the actor using `dactor { ... }` instead of `actor { ... }`</li>
</ol>

The first point is simple - we need access to the new functionality, so we import the new CoActor object instead of the standard Actor object.

For number two - this is slightly nasty. If I interpret things correctly; as the code block created as a parameter to `react` needs to be serializable (so that the actor can be distributed over the network), all enclosing types needs to be serializable. I struggled with this for a while and the only option seems to be creating a proper named serializable type... But since I want to be able to create an actor in-line, we need to do it this way.

For the last point - `dactor { ... }` is simply the function used to create a distributed actor instead of a normal actor.

Let's run it:

{% highlight text %}
2008-06-29 16:11:18.779 Oracle Coherence 3.3.1/389 <Info> (thread=main, member=n/a): Loaded operational configuration from resource "jar:file:/opt/coherence-3.3.1/lib/coherence.jar!/tangosol-coherence.xml"
2008-06-29 16:11:18.785 Oracle Coherence 3.3.1/389 <Info> (thread=main, member=n/a): Loaded operational overrides from resource "jar:file:/opt/coherence-3.3.1/lib/coherence.jar!/tangosol-coherence-override-dev.xml"
2008-06-29 16:11:18.786 Oracle Coherence 3.3.1/389 <D5> (thread=main, member=n/a): Optional configuration override "/tangosol-coherence-override.xml" is not specified

Oracle Coherence Version 3.3.1/389
 Grid Edition: Development mode
Copyright (c) 2000-2007 Oracle. All rights reserved.

2008-06-29 16:11:19.042 Oracle Coherence GE 3.3.1/389 <Info> (thread=main, member=n/a): Loaded cache configuration from resource "file:/crypt/dev/scala/CoherenceTest1/config/scalacoherence.xml"
2008-06-29 16:11:19.331 Oracle Coherence GE 3.3.1/389 <Warning> (thread=main, member=n/a): UnicastUdpSocket failed to set receive buffer size to 1428 packets (2096304 bytes); actual size is 714 packets (1048576 bytes). Consult your OS documentation regarding increasing the maximum socket buffer size. Proceeding with the actual value may cause sub-optimal performance.
2008-06-29 16:11:19.459 Oracle Coherence GE 3.3.1/389 <D5> (thread=Cluster, member=n/a): Service Cluster joined the cluster with senior service member n/a
2008-06-29 16:11:22.662 Oracle Coherence GE 3.3.1/389 <Info> (thread=Cluster, member=n/a): Created a new cluster with Member(Id=1, Timestamp=2008-06-29 16:11:19.343, Address=192.168.54.1:8088, MachineId=24065, Location=process:31397@dellicious, Edition=Grid Edition, Mode=Development, CpuCount=2, SocketCount=1) UID=0xC0A836010000011AD4A9DCAF5E011F98
2008-06-29 16:11:22.834 Oracle Coherence GE 3.3.1/389 <D5> (thread=DistributedCache, member=1): Service DistributedCache joined the cluster with senior service member 1
Actor started.
Sun Jun 29 16:11:23 CEST 2008 - Got ping: 1 Total pings: 1
Sun Jun 29 16:11:24 CEST 2008 - Got ping: 2 Total pings: 2
Sun Jun 29 16:11:25 CEST 2008 - Got ping: 3 Total pings: 3
Sun Jun 29 16:11:26 CEST 2008 - Got ping: 4 Total pings: 4
Sun Jun 29 16:11:27 CEST 2008 - Got ping: 5 Total pings: 5
Sun Jun 29 16:11:28 CEST 2008 - Got ping: 6 Total pings: 6
...
{% endhighlight %}

After the Coherence initialization (which happens automatically and which I've disabled in the outputs below) the actor starts up as expected. However, if we start this on two nodes - there will be two actors created, and no way for a new JVM to get hold of a reference to a specific existing actor. To handle this, let's specify a name for the actor that we create using the `dactor(name : Symbol) { ... }` function:

{% highlight scala %}
...
object DactorTest {
  def main(args : Array[String]) : Unit = {

    dactor('pingActor) {
      var pings : Int = 0
...
{% endhighlight %}

This simply means: Give me a reference to `pingActor`, but if it doesn't exist - create it with the following body. This mechanism makes it easy to have a single instance of an actor even if the same application is running on multiple nodes, without having to explicitly check if an actor has already been created or not.

Now we can run the program on two different nodes. After the actor has started and is running on one node, I'll kill that node:




<table>
<tr valign="top">
<td>Node 1</td><td>Node 2</td></tr>
<tr valign="top">
<td>{% highlight text %}
Actor started.
Sun Jun 29 16:30:41 CEST 2008 - Got ping: 1 Total pings: 1
Sun Jun 29 16:30:42 CEST 2008 - Got ping: 2 Total pings: 2
Sun Jun 29 16:30:43 CEST 2008 - Got ping: 3 Total pings: 3
Sun Jun 29 16:30:44 CEST 2008 - Got ping: 4 Total pings: 4
Sun Jun 29 16:30:45 CEST 2008 - Got ping: 5 Total pings: 5
Sun Jun 29 16:30:46 CEST 2008 - Got ping: 6 Total pings: 6
Sun Jun 29 16:31:02 CEST 2008 - Got ping: 19 Total pings: 19
Sun Jun 29 16:31:03 CEST 2008 - Got ping: 20 Total pings: 20
Sun Jun 29 16:31:04 CEST 2008 - Got ping: 21 Total pings: 21
Sun Jun 29 16:31:05 CEST 2008 - Got ping: 22 Total pings: 22
Sun Jun 29 16:31:06 CEST 2008 - Got ping: 23 Total pings: 23
Sun Jun 29 16:31:07 CEST 2008 - Got ping: 24 Total pings: 24
Sun Jun 29 16:31:08 CEST 2008 - Got ping: 25 Total pings: 25
Sun Jun 29 16:31:09 CEST 2008 - Got ping: 26 Total pings: 26
Sun Jun 29 16:31:10 CEST 2008 - Got ping: 27 Total pings: 27
Sun Jun 29 16:31:11 CEST 2008 - Got ping: 28 Total pings: 28
...
{% endhighlight %}</td>
<td>{% highlight text %}
Sun Jun 29 16:30:47 CEST 2008 - Got ping: 6 Total pings: 6
Sun Jun 29 16:30:48 CEST 2008 - Got ping: 7 Total pings: 7
Sun Jun 29 16:30:49 CEST 2008 - Got ping: 8 Total pings: 8
Sun Jun 29 16:30:50 CEST 2008 - Got ping: 9 Total pings: 9
Sun Jun 29 16:30:51 CEST 2008 - Got ping: 10 Total pings: 10
Sun Jun 29 16:30:53 CEST 2008 - Got ping: 11 Total pings: 11
Sun Jun 29 16:30:54 CEST 2008 - Got ping: 12 Total pings: 12
Sun Jun 29 16:30:55 CEST 2008 - Got ping: 13 Total pings: 13
Sun Jun 29 16:30:56 CEST 2008 - Got ping: 14 Total pings: 14
Sun Jun 29 16:30:57 CEST 2008 - Got ping: 15 Total pings: 15
Sun Jun 29 16:30:58 CEST 2008 - Got ping: 16 Total pings: 16
Sun Jun 29 16:30:59 CEST 2008 - Got ping: 17 Total pings: 17
Sun Jun 29 16:31:00 CEST 2008 - Got ping: 18 Total pings: 18
Sun Jun 29 16:31:01 CEST 2008 - Got ping: 19 Total pings: 19^C
{% endhighlight %}</td>
</tr></table>

First Node 1 started up and ran the actor until Node 2 started. At this point the actor was distributed to Node 2 (determined by the automatic cache partitioning done by Coherence) and started there. As can be seen, the local state (the total pings) was persisted and transferred over. When Node 2 was killed the actor was migrated back and started on Node 1. Note that the state of the actor is persisted for each message, so a sudden shutdown of a JVM is not a problem.

One might wonder why the message for ping number 6 and 19 can be seen in both outputs - this happens as the actor was migrated while the actor thread was sleeping - before the react body was complete. This causes the new node to rerun the message (since the processing of the message didn't complete on the old node) and the support code in the old node makes sure all messages sent by the old actor are discarded as it's been terminated. It's a bit tricky coding actors to be fully idempotent as not everything is handled in a transaction, but limiting side effects to sending messages at the end of the processing makes it fairly reliable.

Here's a slightly more complex example:

{% highlight scala %}
package coherencetest1

import java.util.Date

import scala.actors.coherence.CoActor._

@serializable
object DactorTest2 {
  def main(args : Array[String]) : Unit = {
    init
    
    readLine
    
    val numActors = 80

    val actors = for(id <- 1 to numActors)
      yield dactor {
        loop {
          react {
          case 'ping =>
            println(new Date + " - Actor " + id + " got ping.")
            reply(('pong, id))
          }
        }
      }
    
    actors.map(_ ! 'ping).force
    
    var pongs = 0
    
    while(pongs < numActors) {
      receive {
      case ('pong, x : Int) =>
        pongs = pongs + 1
      }
    }
    
    println("Got " + pongs + " pongs.")
    
    readLine
  }
}
{% endhighlight %}

In this example 80 distributed actors are created and sent a ping message. After that the main thread receives all the pong replies. The output of this, when run on 4 nodes look like so:

<table>
<tr valign="top">
<td>Node 1</td><td>Node 2</td></tr>
<tr valign="top">
<td>{% highlight text %}
Sun Jun 29 09:19:34 CEST 2008 - Actor 1 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 12 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 15 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 18 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 22 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 25 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 27 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 29 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 41 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 42 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 47 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 50 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 54 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 56 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 61 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 75 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 79 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 80 got ping.
Got 80 pongs.
{% endhighlight %}</td>
<td>{% highlight text %}
Sun Jun 29 15:19:34 CEST 2008 - Actor 9 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 13 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 19 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 20 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 21 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 23 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 24 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 28 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 33 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 36 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 46 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 52 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 57 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 58 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 62 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 67 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 68 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 71 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 77 got ping.
{% endhighlight %}</td>
</tr>
<tr valign="top">
<td>Node 3</td><td>Node 4</td></tr>
<tr valign="top">
<td>{% highlight text %}
Sun Jun 29 15:19:34 CEST 2008 - Actor 2 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 4 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 5 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 6 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 7 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 8 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 10 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 11 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 30 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 31 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 32 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 34 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 35 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 37 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 39 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 40 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 43 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 44 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 45 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 53 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 59 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 60 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 63 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 64 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 66 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 69 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 70 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 72 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 74 got ping.
{% endhighlight %}</td>
<td>{% highlight text %}
Sun Jun 29 15:19:34 CEST 2008 - Actor 3 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 14 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 16 got ping.
Sun Jun 29 15:19:34 CEST 2008 - Actor 17 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 26 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 38 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 48 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 49 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 51 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 55 got ping.
Sun Jun 29 15:19:35 CEST 2008 - Actor 65 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 73 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 76 got ping.
Sun Jun 29 15:19:36 CEST 2008 - Actor 78 got ping.
{% endhighlight %}</td>
</tr>
</table>

The program was started on all 4 nodes, but only the first node passes the first readLine. The other nodes just init the distributed actor framework and wait. As can be seen, the actors were distributed over the running nodes as expected - however, with small numbers like this the distribution can be a bit uneven (compare Node 3 and Node 4).

### The Solution

The solution to allow the distribution of serializable actors is based on the Coherence backing map listeners. These can be used to get notifications of which specific node is the master for a certain object. As there only is one master for an object at any point, and a new master is allocated automatically if a master fails, we can use this to determine where the actor should run.

The object returned from `dactor { ... }` is a Proxy object - very similar to the Proxy used in the standard Scala remote actors. In fact, this whole thing is built in a way similar to the standard Scala remote actors, with Proxy objects acting on behalf of the sender on the receiver side and receiver on the sender side.

Additionally, the Coherence grid invocation mechanism allows us to deliver messages to running actors directly to the node where it is running.

When a new dactor is created, the following happens:

<ol>
	<li>`dactor { ... }` creates a new anonymous `CoActor` class with the dactor body set as the `init` method.</li>
	<li>The CoActor.distribute method is called which in turn saves the anonymous class instance to the Coherence cache. The object gets serialized when this happens. The key to the object in the cache is either the passed in symbol name, or a name created from a `java.util.UUID`</li>
	<li>The `dactor` function returns a Proxy object with the name of the newly created distributed actor.</li>
	<li>Meanwhile, in the node which Coherence designates the master of the object, the backing map listener gets an insert event and starts the actor.</li>
	<li>The default `act()` method is called, which in a CoActor calls the `init` method first.</li>
	<li>The `init` method contains the `dactor { ... }` body, which gets executed.</li>
	<li>The body executes as normally up until the `loop` block. `loop` in the CoActor first saves the body of the loop into a variable in CoActor so that it can continue executing just this body after it's started in a new node, without running the initialization code again. After this the loop body is run as normal.</li>
	<li>When `react` is hit, the thread breaks and the actor waits for incoming message using the normal actor scheduling mechanism.</li>
</ol>

When a message is sent to a distributed actor using a Proxy object the following happens:

<ol>
	<li>The message is sent using the Coherence invocation mechanism which transports the message to the master node where the cached actor runs.</li>
	<li>In the master node, a Proxy actor representing the sender (which does not need to be a distributed actor) is created - this is because the actor framework always needs a sending actor which the receiver can reply to.</li>
	<li>The sender Proxy is asked to send the actual message to the real distributed actor receiver.</li>
	<li>The normal Scala actor framework handles the passing of the message and scheduling of the actors.</li>
	<li>An overriden `send` in CoActor locks the actor and stores the message and a Save message in the mailbox.</li>
	<li>The Save message gets the actor to first persist itself to the Coherence cache with the real message still in the mailbox. This is to ensure the message isn't lost in case of a node failure.</li>
	<li>After this the real message is processed by the actor and an overridden `react` saves the actor again after the message has been processed. This to update the cache with the new state of the actor.</li>
</ol>

If the distributed actor does a `reply` or sends a message to the `sender`, the `Proxy` which represents the non-distributed actor gets called as follows:

<ol>
	<li>As the recevier (the original sender) isn't a distributed actor handled by Coherence we cannot use the invocation mechanism. Instead the message is just put into a message cache.</li>
	<li>A `MapListener` on every node checks to see if the newly inserted message in the message cache is intended for an actor running on that specific node.</li>
	<li>If so, the message is deleted from the cache and delivered to the local actor through a Proxy representing the sender - just as in the previous case.</li>
</ol>

### The Limitations

The distributed actors are a bit limited in what they can do, as they always needs to be serializable and I didn't want to change any of the standard Scala code. For instance - when a synchronous invocation is made the return channel is stored in the Scala actor. The return channel implementation used in Scala isn't serializable, so I decided to not implement this feature for now.

Basically, only message sending (!), reply, sender, loop and react are allowed in the distributed actors. However, they can interoperate with normal actors as can be seen in this example:

{% highlight scala %}
    val distActor = dactor('distActor) {
      loop {
        react {
        case ('ping) =>
          reply('pong)
        }
      }
    }
    
    actor {
      distActor ! 'ping
      loop {
        react {
        case x =>
          println("Actor got message: " + x)
        }
      }
    }
{% endhighlight %}

The Proxy objects can be used by actors as they serialize correctly.

### The Conclusion

Distributing (or GridEnabling(tm) or whatever the word <em>du jour</em> is) actors to easily use the processing power and resilience multiple computers give but at the same time hiding the complexity from the developer is a nice way to fairly easily scale up an actors based application. To add more processing power or redundancy - just fire up new nodes.

The proof of concept I made here just scratches the surface, but it's interesting to see that it can be done with Coherence while maintaining the syntax and behavior expected by Scala developers.

### The Source

The highly experimental and hackish source code for the proof of concept is available in the git repository at:

<pre>
http://martin.elwin.com/git/scala-coherence.git
</pre>

Dependencies are Scala (2.7.1 recommended) and Oracle Coherence. There are some scripts for Linux which are trivial to adapt to other operating environments.

/M
