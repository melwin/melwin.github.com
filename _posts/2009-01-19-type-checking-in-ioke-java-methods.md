--- 
wordpress_id: "132"
layout: blog_post
title: Type Checking in Ioke Java Methods
wordpress_url: http://martin.elwin.com/blog/?p=132
---
Ola Bini recently issued <a href="http://kenai.com/projects/ioke/lists/dev/archive/2009-01/message/21">a call to arms</a> to help with the receiver and argument validation for Java methods in Ioke. These are the Ioke methods that are implemented in Java, instead of in Ioke itself.

The Java methods usually operate on the specific data contained within Ioke objects. This data corresponds to different Java classes, depending on what the Ioke object should hold. For instance, an Ioke List contains inside the data area a Java List. To operate on this List the Java code needs to get hold of the List reference - and it does this in a lot of cases by assuming the data object is of a certain type and casts to this type. If the data object is of a different type then a Java `ClassCastException` is thrown, which makes the interpreter quit.

Since Ola mailed the call to arms a number of things have happened. A few people (including myself) have volunteered to help out with adding the validation code. Additionally, a few supporting methods and additions have been added to the Ioke code base to simplify adding type checks to the existing code. And it's these additions that I will talk about here.

### Current Code

As an example what the new type validations can look like I will use the `List +` method that Ola also mentioned in the example.

This is the previous code from <a href="http://github.com/olabini/ioke/blob/e3018142943253f0fd13a967ffb68d39087d9600/src/main/ioke/lang/IokeList.java">IokeList.java</a>:

{% highlight java %}
obj.registerMethod(runtime.newJavaMethod("returns a new list that contains the receivers elements and the elements of the list sent in as the argument.", new JavaMethod("+") {
        private final DefaultArgumentsDefinition ARGUMENTS = DefaultArgumentsDefinition
            .builder()
            .withRequiredPositional("otherList")
            .getArguments();
        
        @Override
        public DefaultArgumentsDefinition getArguments() {
            return ARGUMENTS;
        }
        
        @Override
        public Object activate(IokeObject method, IokeObject context, IokeObject message, Object on) throws ControlFlow {
            List<Object> args = new ArrayList<Object>();
            getArguments().getEvaluatedArguments(context, message, on, args, new HashMap<String, Object>());
            List<Object> newList = new ArrayList<Object>();
            newList.addAll(((IokeList)IokeObject.data(on)).getList());
            newList.addAll(((IokeList)IokeObject.data(args.get(0))).getList());
            return context.runtime.newList(newList, IokeObject.as(on));
        }
    }));
{% endhighlight %}



### New Type Checks

The key to the new validation are two new classes, which extend the functionality of the normal `JavaMethod` class with type checking functionality. These are:

<ul>
	<li><a href="http://github.com/olabini/ioke/blob/0e20b492b8e057e9d2c006698deca02ebf8f45f7/src/main/ioke/lang/TypeCheckingArgumentsDefinition.java">TypeCheckingArgumentsDefinition</a></li>
	<li><a href="http://github.com/olabini/ioke/blob/0e20b492b8e057e9d2c006698deca02ebf8f45f7/src/main/ioke/lang/TypeCheckingJavaMethod.java">TypeCheckingJavaMethod</a></li>
</ul>

When using these to define the Java method we can "annotate" the arguments definition with types for both the arguments and the receiver. By then implementing the appropriate `activate(...)` the super class takes care of evaluating and validating (converting as appropriate) the receiver and arguments.

For instance, for the `List +` example, the above code gets changed to the following:

{% highlight java %}
obj.registerMethod(runtime.newJavaMethod("returns a new list that contains the receivers elements and the elements of the list sent in as the argument.", new TypeCheckingJavaMethod("+") {
        private final TypeCheckingArgumentsDefinition ARGUMENTS = TypeCheckingArgumentsDefinition
            .builder()
            .receiverMustMimic(runtime.list)
            .withRequiredPositional("otherList").whichMustMimic(runtime.list)
            .getArguments();
        
        @Override
        public TypeCheckingArgumentsDefinition getArguments() {
            return ARGUMENTS;
        }
        
        @Override
        public Object activate(IokeObject self, Object on, List<Object> args, Map<String, Object> keywords, IokeObject context, IokeObject message) throws ControlFlow {
            List<Object> newList = new ArrayList<Object>();
            newList.addAll(((IokeList)IokeObject.data(on)).getList());
            newList.addAll(((IokeList)IokeObject.data(args.get(0))).getList());
            return context.runtime.newList(newList, IokeObject.as(on));
        }
    }));
{% endhighlight %}

Note that most of the boiler plate code for arguments handling (which most JavaMethods do) is removed, leaving a clean implementation of the necessary logic - to concatenate two lists in this case.

A few things are done:

<ol>
	<li>The JavaMethod is changed to TypeCheckingJavaMethod</li>
	<li>The arguments definition is changed to a TypeCheckingArgumentsDefinition</li>
	<li>The appropriate types are added to the arguments definition</li>
	<li>Return type of getArguments is changed to TypeCheckingArgumentsDefinition</li>
	<li>The active method is changed to the one which gets the arguments list and keywords</li>
	<li>The manual call to getArguments().getEvaluatedArguments() is removed as we now get the arguments passed</li>
</ol>

The relevant tests for this, which should be added to the `list_spec.ik` test file, could look like:

{% highlight ioke %}
describe(List,
  describe("+", 
    it("should validate type of receiver",
      x = Origin mimic
      x cell("+") = List cell("+")
      fn(x + [3]) should signal(Condition Error Type IncorrectType)
    )

    it("should validate type of argument",
      fn([1,2,3] + 3) should signal(Condition Error Type IncorrectType)
    )
  )
)
{% endhighlight %}

So for you to do:

<ol>
	<li>Fork Ioke</li>
	<li>Pick a `IokeData` subclass which no one has started on yet (check with Ola/naeu/me in the IRC channel #ioke on FreeNode)</li>
        <li>Identify a method which makes faulty assumptions on receiver or arguments</li>
	<li>Add a test as per the above to validate the arguments and receiver</li>
	<li>Change `JavaMethod` implementation as per the above to fix the test</li>
	<li>Commit test and fix in togther to your fork</li>
	<li>Rinse and repeat until the whole file is done</li>
	<li>Convince Ola to pull your fork</li>
</ol>

Let's get crackin'!

(but first - back to normal work... :)

/M
