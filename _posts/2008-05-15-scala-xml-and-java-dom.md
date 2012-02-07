--- 
wordpress_id: "21"
layout: post
title: Scala XML and Java DOM
wordpress_url: http://martin.elwin.com/blog/?p=21
---
How do we go from and DOM to Scala XML the most efficiently? Well... That's what I asked myself the other day. The Scala API <a href="http://www.scala-lang.org/docu/files/api/scala/xml/XML$object.html">`scala.xml.XML`</a> object has helper functions to create Scala XML structures from the usual suspects: Strings, InputStream, Reader, File, etc. But DOM Document or Element is missing.

Going via a Byte array, Char array or String is simple. For instance, the following outputs the DOM to a char array, which is then used as the source of the Scala XML creation:

{% highlight java %}Using Char array
//dom is the DOM Element
val charWriter = new CharArrayWriter()
TransformerFactory.newInstance.newTransformer.transform(new DOMSource(dom), new StreamResult(charWriter))
val xml = XML.load(new CharArrayReader(charWriter.toCharArray))
{% endhighlight %}

This works fine, and is reasonably fast. However, it does allocate some unnecessary memory (the char array) and performs some unnecessary parsing (of the char array) - both of which we'd really like to avoid.

How do we do this? Well, here's one option that I came up with:

{% highlight java %}Using SAX
val saxHandler = new NoBindingFactoryAdapter()
saxHandler.scopeStack.push(TopScope)
TransformerFactory.newInstance.newTransformer.transform(new DOMSource(dom), new SAXResult(saxHandler))
saxHandler.scopeStack.pop
val xml = saxHandler.rootElem
{% endhighlight %}

What's going on here? Well, the Scala XML library uses SAX to parse XML and create the XML structure. One way of generating SAX events is to walk a DOM tree, which is handled by the `javax.xml.transform.Transformer` with a `DOMSource` as input and a `SAXResult` as output. The extension of the `DefaultHandler` needed for handling the SAX events is implemented by the `scala.xml.parsing.FactoryAdapter`, which is extended by the `NoBindingFactoryAdapter` used to construct the XML structure. Because of this, we can do violence on the API and use the `NoBindingFactoryAdapter` directly as a SAX `DefaultHandler` - nice! The `scopeStack` calls are done to maintain the scope information, which I stole from the `loadXML` method in the `AdapterFactory` class.

However, let's take a moment to reflect on this. Using the Scala XML library in this way is not really good. Even if it's possible to do it this way, I've not seen it described as a supported way of using it and therefore it should be done only after considering that the next release of Scala might remove this possibility.

[<strong>Update 2008-07-02:</strong> <a href="http://burak.emir.googlepages.com/">Burak Emir</a> kindly added a comment; "Don't worry, the SAX factory adapter is not going to go away." - good to know!]

That said - let's consider this an exploration of possibilities which could potentially lead to an update of the Scala XML API to allow a DOM to be used as a source instead...!

A quick test gives the following result.

Test data is a 6897 bytes XML file containing 118 elements with some 4 different namespaces.

I ran each test in 1000 iterations, with a full garbage collection before the first iteration. For every 100 iterations I printed the delta of the free memory and then timed the time for the complete 1000 iterations.

Char array: 100 iterations use around 28 MB, full test: 1414 ms
SAX: 100 iterations use around 18 MB, full test: 970 ms

So, in conclusion, not overwhelming difference, but around 1/3 faster and 1/3 less memory consumed. Can we do better? I'm not sure. :)

The next step is to do Scala XML to DOM... This could be more interesting. I see two options:

<ol>
	<li>Implement the DOM API wrapping Scala XML</li>
	<li>Generate SAX events based on the Scala XML and use that to build a DOM</li>
</ol>

Option 1 would be more efficient - but the DOM API isn't fun implementing. Option 2 would be much simpler, but probably would be less efficient and require more allocations. Gotta think about this one...

/M
