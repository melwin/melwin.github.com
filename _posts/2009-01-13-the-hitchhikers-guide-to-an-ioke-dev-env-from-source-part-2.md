--- 
wordpress_id: "88"
layout: blog_post
title: "The Hitchhiker's Guide to an Ioke Dev Env From Source (part 2: Emacs)"
wordpress_url: http://martin.elwin.com/blog/?p=88
---
This is the second part in a series of posts for non-experts about setting up an Ioke development environment on Linux. Please see the previous post to start at the beginning:

<ul>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-1/">Part 1: Git</a></li>
</ul>

In this post we will install GNU Emacs from source and also get a basic configuration set up using the <a href="http://github.com/technomancy/emacs-starter-kit/tree/master">emacs-starter-kit</a>.

If you already have Emacs installed and have an old configuration laying around you probably want to make a backup of this before following the below instructions. This post assumes that there is no Emacs installed and that the user doesn't have any Emacs configuration in the home directory.

### Installing Emacs

Git is an efficient Distributed Version Control System. Although the primary VCS for Emacs is still CVS - and it looks like they are moving towards Bazaar - we'll get the Emacs sources from the <a href="Emacs Git mirror">Emacs Git mirror</a>. We want to be <a href="http://www.unethicalblogger.com/posts/2009/01/im_using_git_because_it_makes_me_feel_cool">cool</a>, right?

{% highlight sh %}
~$ cd work
~/work$ git clone --depth 1 git://git.sv.gnu.org/emacs.git
Initialized empty Git repository in /home/melwin/work/emacs/.git/
remote: Counting objects: 46400, done.
remote: Compressing objects: 100% (24410/24410), done.
remote: Total 46400 (delta 41204), reused 25501 (delta 21836)
Receiving objects: 100% (46400/46400), 74.38 MiB | 213 KiB/s, done.
Resolving deltas: 100% (41204/41204), done.
Checking out files: 100% (2837/2837), done.
{% endhighlight %}

With the `--depth 1` parameter we limit the history so that we only get the latest version of the files - in this case we're not interested in the full history.

<strong>Note:</strong> As we're getting the bleeding edge source code of Emacs, it could happen that the build is broken. I've never had this happen on me, but in case you get strange errors when building Emacs, this might be the reason. Usually such problems are fixed quickly, so try to do a `git pull` a bit later to update the downloaded source.

Previously, to get fun things like <a href="http://www.emacswiki.org/emacs/MultiTTYSupport">multi-tty support</a> and <a href="http://www.emacswiki.org/emacs/XftGnuEmacs">smooth fonts</a>, we had to get specific feature branches of Emacs. Nowadays, however, all the things we want are merged into the Emacs master branch. Before building, the only thing we need to do is to make sure the necessary development libraries are installed. In case you wonder how I came up with this list: the good ol' method of trial and error. I simply ran the `configure` command and checked the error messages. This helped me identify the needed libraries. Now you can reap the benefits by just doing the following:

{% highlight sh %}
~/work$ sudo apt-get install libgtk2.0-dev libxpm-dev libjpeg-dev libgif-dev libtiff-dev
...
{% endhighlight %}

... and then run the classical `configure`, `make` and `make install`:

{% highlight sh %}
~/work$ cd emacs
~/work/emacs$ ./configure --prefix=$HOME/opt/emacs-23.0.60
...
~/work/emacs$ make
...
~/work/emacs$ make install
...
{% endhighlight %}

Phew. That burned som CPU cycles...! Once done - let's add it to our `bin` directory, just as we did with Git.

{% highlight sh %}
~/work/emacs$ cd ~/bin
~/bin$ ln -s ../opt/emacs-23.0.60/bin/emacs
~/bin$ ln -s ../opt/emacs-23.0.60/bin/emacsclient
~/bin$ ls
emacs  emacsclient  git
{% endhighlight %}

Time to test. Just run emacs:

{% highlight sh %}
~/bin% emacs
{% endhighlight %}

This should show you an Emacs window with a pretty(?) GNU.

<img src="http://martin.elwin.com/blog/wp-content/uploads/2009/01/emacs-window.png" alt="Emacs Window" title="Emacs Window" width="694" height="671" class="alignnone size-full wp-image-89" />

If you're completely new to Emacs, this might be a good opportunity to run the Emacs tutorial. Emacs is a self-documenting editor, which means that most things that you might want to learn about Emacs can be found inside the editor itself - including information about internal functions and libraries.

To run the tutorial, just press `Ctrl+h t` (that is, `control` and `h`, release control, then press `t`), or, in Emacs lingo, `C-h t` (which is the convention I'll use from now on).

Once you've learned enough (you did complete the whole thing, right?), just quit using `C-x c`.

<strong>Note:</strong> To update the source code and rebuild, do a `git pull` in the `emacs` directory, then `make distclean` (in case some build files changed) and then perform the compile and installation as per the above again.

Next up is `emacs-starter-kit` to get more functionality in Emacs and a decent default configuration to help us get going - stay tuned!

/M

<strong>Update:</strong> <a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-3/">Part 3: emacs-starter-kit</a>
