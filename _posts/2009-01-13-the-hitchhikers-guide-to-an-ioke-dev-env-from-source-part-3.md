--- 
wordpress_id: "103"
layout: post
title: "The Hitchhiker's Guide to an Ioke Dev Env From Source (part 3: emacs-starter-kit)"
wordpress_url: http://martin.elwin.com/blog/?p=103
---
This is the third part in a series of posts for non-experts about setting up an Ioke development environment on Linux. Please see the previous posts to start at the beginning:

<ul>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-1/">Part 1: Git</a></li>
	<li><a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-2/">Part 2: Emacs</a></li>
</ul>

The <a href="http://github.com/technomancy/emacs-starter-kit/tree/master">emacs-starter-kit</a> is a set of base configuration for Emacs. It contains a number of useful elisp libraries, with a slight focus on dynamic languages.

To install it, perform the following steps (note that we move any existing Emacs configuration out of the way first to avoid stomping what you currently have):

{% highlight sh %}
~/work/emacs$ cd
~$ mv .emacs.d .emacs.d.old
~$ mv .emacsrc .emacsrc.old
~$ git clone git://github.com/technomancy/emacs-starter-kit.git .emacs.d
{% endhighlight %}

If you now start Emacs again you'll see that the menu bar and the toolbar is gone. This is the default in emacs-starter-kit as most Emacs users don't find them useful. For new users the menu bar can sometimes come in handy, to get it back temporarily, just press `F1`.

### Configuring Emacs

If you want to add your own customizations to Emacs when using emacs-starter-kit, just add an Emacs LISP file called <em>`username.el`</em>, or <em>`hostname.el`</em>, in the `~/.emacs.d` directory. For instance, to make the menu bar always visible:


1. Open Emacs, if it's not open already.
1. Press `C-x C-f` and type in: `~/.emacs.d/username.el`<br>Where <em>username</em> is the name you log in with (for instance, in my case the complete filename is `melwin.el`).
1. Type in the following in the file:
        (menu-bar-mode 1)
And save the file with `C-x s`.
1. Now quit (`C-x c`) and restart and you'll see that the menu bar is shown.

### Working with Magit

emacs-starter-kit includes, among many other things, the very nice <a href="http://zagadka.vm.bytemark.co.uk/magit/magit.html">Magit Git mode for Emacs</a>, which gives you a nice interface for working with a Git repository.

Let's use this mode to commit our recent changes to the configuration file to our local clone of the `emacs-starter-kit` repository. This helps us track changes we make and also makes a backup of the file in case we screw (sorry, mess) something up.

<strong>Note 1:</strong> To move easily between Emacs windows using the keyboard, just press `Shift` and the arrow key pointing in the direction you want to move.
<strong>Note 2:</strong> To only show the current Emacs window - press: `C-x 1`

<ol>
	<li>Inside Emacs, press `C-x g` to run `magit-status` and enter the directory (note that `Tab` auto-completes): `~/.emacs.d`<br>
        <img src="http://martin.elwin.com/blog/wp-content/uploads/2009/01/emacs-magit-status.png" alt="Emacs magit-status" title="Emacs magit-status" width="694" height="353" class="alignnone size-full wp-image-114" />
        </li>
	<li>Put the cursor over the `<em>username</em>.el` in the list of `Untracked files`.</li>
	<li>Press `s` to Stage the new file - this adds the file to the Git staging area, from which all files are committed.<br>
<img src="http://martin.elwin.com/blog/wp-content/uploads/2009/01/emacs-magit-status-staged.png" alt="Emacs magit-status staged" title="Emacs magit-status staged" width="694" height="353" class="alignnone size-full wp-image-115" />
</li>
	<li>Press `d` and then accept to diff against HEAD - this will show you a diff view of the changes we have staged - just the add of a single file.<br>
<img src="http://martin.elwin.com/blog/wp-content/uploads/2009/01/emacs-magit-status-diff.png" alt="Emacs magit-status diff" title="Emacs magit-status diff" width="757" height="429" class="alignnone size-full wp-image-116" />
</li>
	<li>Press `c` to perform the commit. This opens a new buffer into which a commit message can be added.</li>
	<li>Write something like: `Add personal configuration file.`</li>
        <li>Now press `C-c C-c` to commit the file.</li>
</ol>

That's it - now the change has been committed to the local clone of the emacs-starter-kit Git repository.

To see the log of all commits, press `l` (lowercase L) in the `magit-status` buffer:<br>
<img src="http://martin.elwin.com/blog/wp-content/uploads/2009/01/emacs-magit-status-log.png" alt="Emacs magit-status log" title="Emacs magit-status log" width="757" height="429" class="alignnone size-full wp-image-121" />

To look at a certain commit - just press `Enter` on it and a view of the diff will be shown. This makes it quite easy to browse through commits in a repository. At the top of the log is the most recent commit, which in this case is the file we just added.

To update emacs-starter-kit with the latest changes in the GitHub repository, just press `F` in the `magit-status` buffer or run `git pull` in the `~/.emacs.d` directory.

### Summary

Now that we have Git and Emacs set up we can finally move to Ioke. In the next post we'll go through installing the latest Java JDK and getting and compiling the Ioke source code.

Join me then!

/M

<strong>Update:</strong> <a href="http://martin.elwin.com/blog/2009/01/the-hitchhikers-guide-to-an-ioke-dev-env-from-source-part-4/">Part 4: Java and Ant</a>
