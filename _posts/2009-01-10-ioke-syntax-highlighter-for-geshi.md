--- 
wordpress_id: "56"
layout: post
title: Ioke Syntax Highlighter for GeSHi
wordpress_url: http://martin.elwin.com/blog/?p=56
---
Speaking of Ioke - just added a syntax highlighter definition for GeSHi in my Ioke fork repository. The commit can be found here:

<a href="http://github.com/melwin/ioke/commit/54cfd7e54de0be910385c6ec805693fd3ed4e294">http://github.com/melwin/ioke/commit/54cfd7e54de0be910385c6ec805693fd3ed4e294</a>

The keyword definitions I borrowed (read stole) from <a href="http://sam.aaron.name/">Sam Aaron</a>'s TextMate bundle (also in the Ioke repository). As Sam just said in the #ioke on freenode: "what goes around comes around". :)

Highlighting test:

{% highlight ioke %}
    m = #/({areaCode}\d{3})-({localNumber}\d{5})/ =~ number

    describe("start",
      it("should return the start index of group zero, which is the whole group",
        (#/foo/ =~ "foobar") start should == 0
        (#/foo/ =~ "abcfoobar") start should == 3

        (#/foo/ =~ "foobar") start(0) should == 0
        (#/foo/ =~ "abcfoobar") start(0) should == 3
      )

      it("should return the start index of another group",
        (#/(..) (..) (..)/ =~ "fooab cd efbar") start(2) should == 6
      )

      it("should return the start index from the name of a named group",
        (#/({one}..) ({two}..) ({three}..)/ =~ "fooab cd efbar") start(:two) should == 6
      )

      it("should return -1 for a group that wasn't matched",
        (#/(..)((..))?/ =~ "ab") start(2) should == -1
        (#/({no}..)(({way}..))?/ =~ "ab") start(:way) should == -1

        (#/(..)((..))?/ =~ "ab") start(10) should == -1
        (#/({no}..)(({way}..))?/ =~ "ab") start(:blarg) should == -1
      )

      it("should validate type of receiver",
        Regexp Match should checkReceiverTypeOn(:start)
      )
    )
    
    x = #/bla #{"foo"} bar/ 
{% endhighlight %}

/M
