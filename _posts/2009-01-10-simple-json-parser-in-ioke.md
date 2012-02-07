--- 
wordpress_id: "39"
layout: post
title: Simple JSON Parser in Ioke
wordpress_url: http://martin.elwin.com/blog/?p=39
---
After having enjoyed most of the days off over Christmas my fingers started itching - time to do some programming. Luckily around the same time I stumbled across <a href="http://olabini.com/blog/category/ioke/">Ola Bini's posts on the Ioke language</a>. As the <a href="http://ioke.org/guide.html">Ioke guide</a> begins:

<blockquote>
Ioke is a general purpose language. It is a strongly typed, extremely dynamic, prototype object oriented language. It is homoiconic and it's closest ancestors is Io, Smalltalk, Ruby and Lisp - but it's quite a distance from all of them.
</blockquote>

So what does this all mean? It means it's fun! The language syntax is very regular (although not quite as regular as lisp) - code is data, data is code and everything is a message. It's also one of relatively few languages that provide macros similar to those in lisp. In Ioke this means that a message stream can be operated on (modified, transformed, etc) before it's evaluated, using normal Ioke methods. Anyone who groks lisp should be familiar with the power this gives.

In Ioke, a chain of messages is separated by space, with the first message being sent to the current receiver and subsequent messages are sent to the result of the previous one. For instance, in the code

{% highlight ioke %}
"ioke rocks" upper println
{% endhighlight %}

the text literal "ioke rocks" results in an internal message which creates a Text object. To this object the `upper` messages is sent, which results in an upper case copy of the Text object is created. To this object the `println` message is sent, which prints the upper case text to the console.

As the Ioke reader handles space separated tokens, it's quite easy to create macros to process non Ioke code. As a learning exercise I decided to try to implement a <a href="http://www.json.org/">JSON</a> parser. Nothing too fancy - and doesn't even have to be secure - just enough to parse simple JSON into Ioke objects.

JSON is usually parsed in one of two ways - into custom objects/classes or into collections like dictionaries and arrays. For this exercise I decided to create Ioke dictionaries and arrays and I want the result to work like this:

{% highlight ioke %}
json({
  "string" : "string1",
  "int" : 1234,
  "arr" : ["item1", "item2"],
  "dict" : {"key1":"value1"}
}) println  ;; => {dict={key1=value1}, arr=[item1, item2], int=1234, string=string1}

json(["string",
  1234,
  ["item1", "item2"],
  {"key1": "val1"}
]) println ;; => [string, 1234, [item1, item2], {key1=val1}]
{% endhighlight %}

Now, looking at JSON, it's suspiciously similar to Ioke - [] is used for arrays, {} is used for "dictionaries", comma separates entries, " quotes strings, etc. The main problem is the colon ":" used for separating keys and values in a dictionary. As Ioke is a dynamic language we can redefine how messages are handled. Instead of using colon to define symbols, we can redefine it to instead create pairs suitable for a dict:

{% highlight ioke %}
Text cell(":") = macro(call resendToMethod("=>"))
{% endhighlight %}

":" is originally defined on DefaultBehavior Literals, but in the JSON the ":" message is always sent to Text, so it's enough to redefine it there.

This single redefinition allows us to parse JSON directly with the Ioke reader - very nice.

But... Redefining ":" for all Text objects is not really good (especially not when Ola adds concurrency constructs!). Another alternative would be to redefine how the text objects are created and create a new ":" cell just for our JSON text objects - but I never got this to work, as the internal:createText message works with a raw Java string, which I couldn't work with in a macro/method...

<strong>Update:</strong> With <a href="http://github.com/olabini/ioke/commit/8af3954df7961b3f594c73db5059310469e45df5">Ola's recent commit</a>, the previously described workaround can now be simplified to the following code.

Now the above redefinition of the ":" message can be used, but is visible only in the scope of a `let`:

{% highlight ioke %}
json = macro(let(Text cell(":"), DefaultBehavior cell("=>"),
    call argAt(0)
))

;; Used as:

json({
  "string" : "string1",
  "int" : 1234,
  "arr" : ["item1", "item2"],
  "dict" : {"key1":"value1"}
  }) println

json(["string",
  1234,
  ["item1", "item2"],
  {"key1": "val1"}
  ]) println
{% endhighlight %}

As can be seen, the Ioke macro functionality gives us nice control over how messages are handled, and the environment can be changed before the arguments are evaluated. I'm looking forward to the day when we have full Java interop and can use these powerful constructs to control Java code!

Source code to the above example is committed to my Ioke fork at: <a href="git://github.com/melwin/ioke.git">git://github.com/melwin/ioke.git</a>

/M

PS: The thing that confuses me the most: how the heck are you supposed to pronounce Ioke?

<strong>Update:</strong> Sam Aaron shared the fact that Ola pronounces it eye-oh-key in the following interview: <a href="http://www.akitaonrails.com/2008/11/22/rails-podcast-brasil-qcon-special-ola-bini-jruby-ioke">http://www.akitaonrails.com/2008/11/22/rails-podcast-brasil-qcon-special-ola-bini-jruby-ioke</a>
