--- 
wordpress_id: "22"
layout: blog_post
title: DCOP and xmobar
wordpress_url: http://martin.elwin.com/blog/?p=22
---
For me, the computer is a work tool that should get out of the way as much as possible to allow me to perform the task at hand. I spend many hours every day working with, primarily, my laptop, and seemingly insignificant improvements to the desktop environment and development tools add up over time to improve my overall efficiency. Because of this I like to spend time looking into how the tools I use can be tuned to better help in what I need to do.

One of the fairly recent chances I've made is to switch to a <a href="http://en.wikipedia.org/wiki/Tiling_window_manager">tiling window manager</a>. For those too lazy to read the Wikipedia entry: A tiling window manager automatically arranges the open windows to fill the available space of the screen - saving you the job of arranging the windows yourself. That sounds simple, but can be quite complex in practice.

I started out using <a href="http://modeemi.fi/~tuomov/ion/">Ion3</a> in mid 2007. Ion3 is nice and smooth and worked great and really got me hooked on tiled window managers. However, as a big fan of open source, I was a bit concerned by the author's approach to controlling the development of Ion (see <a href="http://womble.decadent.org.uk/blog/renaming-of-ion3">this</a> or <a href="http://forums.gentoo.org/viewtopic-t-559010-start-0-postdays-0-postorder-asc-highlight-.html">this</a> for instance) so I switched to <a href="http://www.suckless.org/wiki/">wmii</a>. wmii was also quite nice, but I never got into it as well as Ion, for some reason.

That's when I found <a href="http://xmonad.org/">xmonad</a> - a very nice window manager written in <a href="http://www.haskell.org/">Haskell</a>. Haskell is also used as the configuration language, which makes the configuration possibilities close to endless (actually, they probably are endless, as Haskell is Turing equivalent (although there are <a href="http://en.wikipedia.org/wiki/Talk:Type_polymorphism#Why_Haskell.3F">people who doubt it</a>!) :).

Today, some 6 months later, I use xmonad 0.7 together with KDE 3.5.9 (similar to the <a href="http://www.haskell.org/haskellwiki/Xmonad/Using_xmonad_in_KDE">setup described on the Haskell wiki</a>) on Kubuntu 8.04 and am very happy with it.

### Status Bar
However, xmonad is just a tiling window manager, but I also want a status bar to allow me to see chosen pieces of information at a glance. I use KDE3 and have the KDE3 panel `kicker` running, but it's hidden unless I put the mouse in the bottom left corner. I don't use it normally and only keep it around to access the system tray (which I almost never need). Instead of the ugly and bulky `kicker` I want a smaller, slimmer alternative...

Enter <a href="http://code.haskell.org/~arossato/xmobar/">xmobar</a> - a text based status bar, also written in Haskell. xmobar comes with a number of plugins to show different aspects of the system on the status bar - date, cpu, memory, network activity, etc. A few things it doesn't do out of the box, which I want to see, are number of new mails, current keyboard layout (I use US mainly, but switch to Swedish for the national chars) and speaker mute state. Luckily, xmobar provides a plugin to execute shell commands, and using this mechanism we can put anything on the status bar, as long as we can figure out a command to run to produce the wanted text.

### New Mails
I use a <a href="http://en.wikipedia.org/wiki/Maildir">Maildir</a> mail storage (together with <a href="http://www.dovecot.org/">dovecot</a> to provide an IMAP interface to my mail), and since I don't automatically split mails into groups/folders it's enough to just check the number of mails in my Maildir's `new` folder. The xmobar command for this ends up as:

{% highlight haskell %}
Run Com "sh" ["-c", "\"ls ~/Maildir/new | wc -l\""] "mailcount" 600
{% endhighlight %}

This command invokes the `sh` shell and passes a <em>command string</em> using the `-c` switch. The command string contains the actual shell command we want to execute: `ls ~/Maildir/new | wc -l`

Executing `sh` and passing a command string allows us to use the pipe to pass the output of the `ls` to the `wc` to get the actual file count back.

The command is mapped to the `mailcount` alias, and the interval is set to 600 tenths of a second = 60 seconds.

### Keyboard Layout
Since I use KDE3 the keyboard layout switching is handled by the `kxkb` application (shows up as a flag in the tray if multiple layouts are configured). KDE3 uses <a href="http://en.wikipedia.org/wiki/DCOP">DCOP</a> (unlike KDE4 which uses <a href="http://en.wikipedia.org/wiki/D-Bus">D-Bus</a>) for IPC. Available DCOP services can easily be interogated from the command line using the... you guessed it... `dcop` command.

Running `dcop` without parameters shows a list of available applications. On my system it shows:

{% highlight bash %}
% dcop
konsole-9099
kdebluetooth
kicker
kxkb
guidance-6223
kded
adept_notifier
kmix
knotify
kio_uiserver
klauncher
khotkeys
kwalletmanager
digikam-6484
klipper
ksmserver
knetworkmanager
{% endhighlight %}

Woohoo! kxkb is there, so let's dig deeper. By a bit of trial and error we can probe the internals of the dcop-exposed application:
{% highlight bash %}
% dcop kxkb
qt
MainApplication-Interface
kxkb
% dcop kxkb kxkb
QCStringList interfaces()
QCStringList functions()
bool setLayout(QString layoutPair)
QString getCurrentLayout()
QStringList getLayoutsList()
void forceSetXKBMap(bool set)
% dcop kxkb kxkb getCurrentLayout
us
{% endhighlight %}

Sweet! Now we have the dcop "path" to find the current keyboard layout. Let's add it to the xmobar configuration:

{% highlight haskell %}
Run Com "dcop" ["kxkb", "kxkb", "getCurrentLayout"] "kbd" 20
{% endhighlight %}

### Speaker Mute State
Using `dcop` again we can find the mute state:

{% highlight bash %}
% dcop kmix
qt
MainApplication-Interface
Mixer0
kmix
kmix-mainwindow#1
% dcop kmix Mixer0
QCStringList interfaces()
QCStringList functions()
void setVolume(int deviceidx,int percentage)
void setMasterVolume(int percentage)
void increaseVolume(int deviceidx)
void decreaseVolume(int deviceidx)
int volume(int deviceidx)
int masterVolume()
void setAbsoluteVolume(int deviceidx,long int absoluteVolume)
long int absoluteVolume(int deviceidx)
long int absoluteVolumeMin(int deviceidx)
long int absoluteVolumeMax(int deviceidx)
void setMute(int deviceidx,bool on)
void setMasterMute(bool on)
void toggleMute(int deviceidx)
void toggleMasterMute()
bool mute(int deviceidx)
bool masterMute()
int masterDeviceIndex()
void setRecordSource(int deviceidx,bool on)
bool isRecordSource(int deviceidx)
void setBalance(int balance)
bool isAvailableDevice(int deviceidx)
QString mixerName()
int open()
int close()
% dcop kmix Mixer0 masterMute
true
{% endhighlight %}

Perfect - now we can create the last xmobar run command:

{% highlight haskell %}
Run Com "dcop" ["kmix", "Mixer0", "masterMute"] "mute" 20
{% endhighlight %}

### Final Configuration

So, here's the final `.xmobarrc`:

{% highlight haskell %}
Config { font = "xft:Consolas-8"
       , bgColor = "black"
       , fgColor = "grey"
       , position = Bottom
       , commands = [ Run Network "eth0" ["-L","0","-H","32","--normal","green","--high","red"] 50
                    , Run Network "wlan0" ["-L","0","-H","32","--normal","green","--high","red"] 51
                    , Run Cpu ["-L","3","-H","50","--normal","green","--high","red"] 52
                    , Run Memory ["-t","Mem: <usedratio>%"] 54
                    , Run Date "%a %b %_d %H:%M:%S" "date" 10
                    , Run StdinReader
                    , Run Com "dcop" ["kxkb", "kxkb", "getCurrentLayout"] "kbd" 20
                    , Run Com "sh" ["-c", "\"ls ~/Maildir/new | wc -l\""] "mailcount" 600
                    , Run Com "dcop" ["kmix", "Mixer0", "masterMute"] "mute" 20
                    ]
       , sepChar = "%"
       , alignSep = "}{"
       , template = "<fc=#ee9a00>%date%</fc> | %cpu% | %memory% | %eth0% - %wlan0% } %StdinReader% { Mail: %mailcount% | Kbd: %kbd% | Mute: %mute%"
       }
{% endhighlight %}

And a small screenshot for your viewing pleasure:

<a href='http://martin.elwin.com/blog/wp-content/uploads/2008/05/xmonad.png'><img src="http://martin.elwin.com/blog/wp-content/uploads/2008/05/xmonad.png" alt="" title="xmonad" width="300" height="187" class="alignnone size-medium wp-image-23" /></a>
(Yes, my laptop's hostname is <em>dellicious</em>... ;)
