--- 
wordpress_id: "74"
layout: blog_post
title: "The Hitchhiker's Guide to an Ioke Dev Env From Source (part 1: Git)"
wordpress_url: http://martin.elwin.com/blog/?p=74
---
In this series of posts I will guide you, the humble reader, through the install procedure to get a development environment for <a href="http://ioke.org/">Ioke</a> up and running. All the cool kids want to develop in Ioke nowadays, so let's make you can as well!

The components we will be installing are:

<ul>
	<li><a href="http://git-scm.com/">Git 1.6.1</a></li>
	<li><a href="http://www.gnu.org/software/emacs/">Emacs (latest snapshot)</a> with <a href="http://github.com/technomancy/emacs-starter-kit/tree/master">emacs-starter-kit</a></li>
	<li><a href="http://java.sun.com/">Java JDK 1.6</a></li>
	<li><a href="http://ioke.org/">Ioke (latest snapshot)</a></li>
</ul>

These instructions are written for a non-expert who might not have too much experience with compiling things, using Git or Emacs. If you're an expert you will find nothing new or exciting here! The environment I use is a freshly installed <a href="http://www.kubuntu.org/">Kubuntu Intrepid Ibex</a> box, although most of it should be similar, if not identical, to other Ubuntu/Debian based distributions.

The components listed above will all be installed manually - not using the package system. This is to allow us to get the freshest versions of what we need without depending on the packages in the distribution to be updated (or hunted down in alternative repositories). The only thing we will install from the distribution are the compilers, tools and development libraries used to compile the software.

Why do we not use the package system? Well, that's a valid question. Most of the above can be installed from packages in custom repositories. It's simple and convenient. However here we will not use if for two reasons:
<ol>
	<li>If packages are used we will depend on the repositories to be updated to use the latest version of some software, which can be annoying if we want to develop on the bleeding edge.</li>
	<li>It's good to know how to compile things yourself - so by not using prepared packages we might learn something!</li>
</ol>

To keep control over our custom built software we'll install it in dedicated directories under `$HOME/opt` instead of in the normal locations like `/usr/bin`. This allows us to get everything installed without needing root access (except to install the compilers etc from the distribution).

### First Step

# Don't Panic!

Always sound advice to start with - especially if you are hitchhiking. I'm going to try to cover each step in detail, but if you think something is unclear - just leave a comment and I'll try to clarify. So - grab your towel and let's get going!

### Install Git

For Emacs, emacs-starter-kit and Ioke we will use Git to retrieve the sources. Therefore the first thing we need to install is Git.

In the command line instructions below, all lines prefixed by the prompt `&lt;dir&gt;$` are commands to be typed in. The rest are output or my comments.

Open a terminal window and create a new work directory under your home using the following instructions. In this directory we will work with downloaded source files and build the software.

{% highlight sh %}
~$ mkdir ~/work
~/work$ cd ~/work
~/work$ wget -O - http://kernel.org/pub/software/scm/git/git-1.6.1.tar.bz2 | tar xjv
... many lines...
{% endhighlight %}

The last command downloads the git source package and expands it in one go by piping it directly from `wget` to `tar`, which is told to expand it using bz2 with the `j` parameter. This is a convenient way to get packages off the net and unpacked, especially when there is no need to keep the package itself around.

The result is that we now have a `git-1.6.1` directory in the work directory. Let's see how we compile it:

{% highlight sh %}
~/work$ cd git-1.6.1/
~/work/git-1.6.1$ head INSTALL

                Git installation

Normally you can just do "make" followed by "make install", and that
will install the git programs in your own ~/bin/ directory.  If you want
to do a global install, you can do

        $ make prefix=/usr all doc info ;# as yourself
        # make prefix=/usr install install-doc install-html install-info ;# as root

~/work/git-1.6.1$
{% endhighlight %}

So - two commands. Simple enough! However, before we compile - we need to make sure all the relevant packages are installed:

{% highlight sh %}
~/work/git-1.6.1$ sudo apt-get install libcurl4-openssl-dev zlib1g-dev libexpat-dev tk8.5 asciidoc docbook2x
{% endhighlight %}

Depending on your internet connection, this could take quite a while, as we need to download the `texlive` distribution, among other things, to build all of the Git documentation. This is not strictly necessary, but here we'll just do it for completeness' sake.

Let's build git and make sure it's installed under our `$HOME/opt` directory as we said in the beginning:

{% highlight sh %}
~/work/git-1.6.1$ make prefix=~/opt/git-1.6.1 all doc info
... lots of lines...
{% endhighlight %}

Compiling Git will take some time as well. Go get a coffee (or perhaps a <a href="http://en.wikibooks.org/wiki/Bartending/Cocktails/Pan_Galactic_Gargle_Blaster">Pan Galactic Gargle Blaster</a> - sweet like nectar).

Once the compile is done - install it. Note that we don't need to do this as `root` as we're installing under the user home directory:

{% highlight sh %}
~/work/git-1.6.1$ make prefix=~/opt/git-1.6.1 install install-doc install-html install-info
{% endhighlight %}

Let's add it to the user's path by linking it into the private `bin` directory. This depends on the standard Ubuntu `bash` shell profile script which adds `~/bin` to the `PATH` variable. If a different shell is used you need to perform the appropriate steps yourself.

{% highlight sh %}
~/work/git-1.6.1$ mkdir ~/bin
~/work/git-1.6.1$ cd !$
~/bin$ ln -s ../opt/git-1.6.1/bin/git
{% endhighlight %}

Now close the shell/terminal, open a new one - and try the `git` command:

{% highlight sh %}
~$ git --version
git version 1.6.1
{% endhighlight %}

If you get the above output - great! You're don! Sit back and relax for a bit before moving on to the next section.

However, if you don't get the version output, but instead see the following message, review the previous instructions and make sure it works ok before continuing.

{% highlight sh %}
#INCORRECT OUTPUT - SOMETHING WAS MISSED!
#GO BACK AND REVIEW
~$ git                                                                                                                                                                                        
The program 'git' is currently not installed.  You can install it by typing:                                                                                                                  
sudo apt-get install git-core                                                                                                                                                                 
-bash: git: command not found
{% endhighlight %}

If you still can't get it to work - drop me a comment!

### Git in 30 Seconds

There are loads of good Git tutorials and information. You can find several on the <a href="http://git-scm.com/documentation">official Git page</a> and at <a href="http://github.com/guides/home">GitHub</a> (go sign up if you haven't already, and <a href="http://github.com/melwin">fork me</a>!).

Here is a quick run through of a few common Git commands you could try out with your freshly brewed cup of Git:

{% highlight sh %}
~$ cd
~$ mkdir gittest
~$ cd gittest/
~/gittest$ git init
Initialized empty Git repository in /home/melwin/gittest/.git/
~/gittest$ echo Test file! > test.txt
~/gittest$ git add test.txt
~/gittest$ git commit -m "Initial import."
[master (root-commit)]: created 79c091d: "Initial import."
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 test.txt
~/gittest$ echo Add line. >> test.txt
~/gittest$ git diff test.txt
diff --git a/test.txt b/test.txt
index 1cbaf90..3746f9e 100644
--- a/test.txt
+++ b/test.txt
@@ -1 +1,2 @@
 Test file!
+Add line.
~/gittest$ git commit -a -m "Add new line."
[master]: created b20f9a4: "Add new line."
 1 files changed, 1 insertions(+), 0 deletions(-)
{% endhighlight %}

If you can follow the above - great! First part finished. Next up is to install GNU Emacs from source and the emacs-starter-kit, which provides a decent default a set of configuration for Emacs.

Stay tuned!

/M

<strong>Update:</strong> <a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-2/">Part 2: Emacs</a>
