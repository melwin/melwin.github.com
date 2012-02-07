--- 
wordpress_id: "20"
layout: blog_post
title: Compressed and Encrypted Backup with SquashFS and LUKS
wordpress_url: http://martin.elwin.com/blog/?p=20
---
<a href="http://kubuntu.org">Hardy Heron (KDE flavor)</a> has been out for a while now and this weekend I finally decided to upgrade my (sweet sweet Dell M90) laptop from Gutsy. I used this as justification to get a new Hitachi SATA 200GB 7200RPM disk to replace the old 250GB 5200RPM in an effort to boost performance a little. Getting a new disk also makes the upgrade a lot less risky - I can keep the old disk as it is while setting up the new system.

This time around I decided to get rid of the Windows partition previously used for dual booting (haven't booted into Windows in 6 months). I also wanted to switch from the extremely useful and amazing <a href="http://www.truecrypt.org/">Truecrypt</a> to using Linux native <a href="http://luks.endorphin.org/">LUKS</a> encryption for my work and private data. I originally used Truecrypt on Linux with an NTFS file system to make the encrypted drive compatible with both Windows and Linux, but since then I've switched to an ext3 file system and don't need the capability to mount it both under Linux and Windows. If you do, make sure to try - nay, use! - Truecrypt. It's very very nice.

So, the 200GB disk ended up being partitioned like so:

<pre>
$sudo fdisk -l /dev/sda

Disk /dev/sda: 200.0 GB, 200049647616 bytes
255 heads, 63 sectors/track, 24321 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Disk identifier: 0x000cbcfd

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1        3647    29294496   83  Linux
/dev/sda2            3648        7294    29294527+  83  Linux
/dev/sda3            7295       23707   131837422+  83  Linux
/dev/sda4           23708       24321     4931955   82  Linux swap
</pre>

That is, 30GB for root, 30GB to be encrypted, 135GB for data and the rest for swap.

Of course, with Hardy you can <a href="http://learninginlinux.wordpress.com/2008/04/23/installing-ubuntu-804-with-full-disk-encryption/">encrypt the root file system</a> just by selecting to do so in the alternate installer, but I don't feel a need to go that way right now - maybe later.

### Backups

Now, slowly approaching the main topic of this post, while playing around with the new disk and the old data I was thinking about putting in place a new backup strategy. At the same time (I can do anything while simultaneously going through my subscriptions in Google Reader:) I happened upon the <a href="http://jwz.livejournal.com/801607.html">JWZ post about backups</a>. JWZ, who obviously is a smart guy, is onto something. I'm already doing backups (of part of my data) using `rsync` to my server at home - not full disk image, admittedly, which was JWZ's prescription. However, I wanted to try something different and achieve a few other things:

<ul>
	<li>Retain multiple snapshots of data at a time</li>
	<li>Compress the data</li>
	<li>Encrypt the data</li>
</ul>

For a while I was considering ZFS (which I in a moment of temporary megalomania started porting to a native Linux kernel module, strictly for private use! - maybe something for another post) with its nice snapshot and compression support, but this idea was quickly discarded for being a bit too unnatural for most Linux setups. I added the additional requirement that the solution should work without having too many non-standard dependencies.

<a href="http://bisqwit.iki.fi/source/cromfs.html">CromFS</a> has a nice feature set, but it's a FUSE file system and it's not available in Hardy. The comparison chart on the CromFS page however led me to <a href="http://squashfs.sourceforge.net/">SquashFS</a>. SquashFS is available out-of-the-box in Hardy and by `apt-get`ting the` squashfs-tools` package, everything needed is installed. The nice <a href="http://tldp.org/HOWTO/SquashFS-HOWTO/">HowTo</a> gives a good introduction to the commands required to use it. Note - SquashFS, as most compressed file systems, is read-only. This suites me perfectly as I want to make a snapshot for backup purposes, but it might not be what you want.

The idea I got was to use SquashFS as the compressed snapshot file system and wrap it in LUKS for encryption. By doing this I get a single compressed file which I can mount on a modern Linux distribution to access the data. It's securely and (somewhat) efficiently stored.

Luckily, what I wanted to do is very similar to what the <a href="http://gentoo-wiki.com/HOWTO_Burn_Encrypted_Optical_Media_With_Luks">Gentoo guide for how to burn and encrypted CD image</a> describes - namely wrapping an already existing file system image in a LUKS encrypted container. Most of the following is based on the Gentoo guide.

There are a few steps to the process:

<ol>
	<li>Create SquashFS image from source data</li>
	<li>Create LUKS container</li>
	<li>Put SquashFS image inside LUKS container</li>
</ol>

The trickiest (which isn't really that tricky) part is figuring out how large to make the LUKS container. As we don't know beforehand how large the SquashFS image is, we need to create it and then calculate how large the LUKS container should be.

### Step 1: SquashFS

Let's start with creating the SquashFS image:

{% highlight sh %}
$sudo mksquashfs /crypt /tmp/cryptbackup.sqsh -e /crypt/stuff
Parallel mksquashfs: Using 2 processors
Creating little endian 3.1 filesystem on /tmp/cryptbackup.sqsh, block size 131072.
[==========================================================================] 2297/2297 100%
Exportable Little endian filesystem, data block size 131072, compressed data, compressed metadata, compressed fragments, duplicates are removed
Filesystem size 13606.69 Kbytes (13.29 Mbytes)
        21.57% of uncompressed filesystem size (63077.89 Kbytes)
Inode table size 38419 bytes (37.52 Kbytes)
        32.62% of uncompressed inode table size (117780 bytes)
Directory table size 33629 bytes (32.84 Kbytes)
        58.10% of uncompressed directory table size (57880 bytes)
Number of duplicate files found 115
Number of inodes 3176
Number of files 1877
Number of fragments 45
Number of symbolic links  945
Number of device nodes 0
Number of fifo nodes 0
Number of socket nodes 0
Number of directories 354
Number of uids 5
        ....
Number of gids 12
        ....
{% endhighlight %}

This creates a new compressed SquashFS image from the data in the `/crypt` directory (the contents of this directory becomes the contents of the image root). The `-e` flag excludes all files in the given directories - here, everything in `/crypt/stuff`. Note that it might be more efficient to store this image on a separate disk, if available. Even an external USB 2.0 mounted disk might be faster to write to while reading the data from the main disk.

The SquashFS image was compressed by about 50% in my case - 10GB of data stored in a 5GB image. Of course, the compression ratio achieved depends on the type data stored.

### Step 2: LUKS Container

Step 1 done, we now need to create a LUKS container to store the SquashFS image in. How big should it be? Well... The Gentoo guide linked to above calculates the size of the LUKS overhead by checking the difference between a LUKS container and the mapped block device. It turns out that this overhead is 1032 blocks (each block being 512 bytes), no matter what the block size is. Googling this seems to confirm it, so for now I'm assuming that LUKS always adds 1032 blocks of overhead.

The size in 512 byte blocks of the SquashFS image can be found by doing:

{% highlight sh %}
$ls -l --block-size=512 /tmp/cryptbackup.sqsh
-rwx------ 1 root root 27216 2008-05-11 20:55 /data/cryptbackup.sqsh
{% endhighlight %}

Which in the above case indicates that the file is 27216 512 byte blocks large (this is a test file...).

Adding 1032 blocks gives us the size needed for the LUKS container - 28248 blocks - let's create it (while letting the shell handle the calculation for us):

{% highlight sh %}
$sudo dd if=/dev/zero of=/tmp/cryptbackupluks.img bs=512 count=1 seek=$((27216+1032))
{% endhighlight %}

Note that this creates a sparse file on most modern file systems, so it's quite quick. We don't need to fill it with random numbers or anything as the whole container will be updated when we write the SquashFS image to it.

Now, let's map it. First locate an available loop device:

{% highlight sh %}
$sudo losetup -f
/dev/loop0
{% endhighlight %}

`loop0` is available - no loops on this system.

Set up the container file as a loop device:

{% highlight sh %}
$sudo losetup /dev/loop0 /tmp/cryptbackupluks.img
{% endhighlight %}

Then make it a LUKS volume:

{% highlight sh %}
$sudo cryptsetup luksFormat /dev/loop0

WARNING!
========
This will overwrite data on /dev/loop0 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter LUKS passphrase:
Verify passphrase:
Command successful.
{% endhighlight %}

Make sure you remember the password... ;P

And open the device:

{% highlight sh %}
$sudo cryptsetup luksOpen /dev/loop0 cryptbackup
Enter LUKS passphrase:
key slot 0 unlocked.
Command successful.
{% endhighlight %}

Now the device is available as `/dev/mapper/cryptbackup`, ready to accept our SquashFS image.

### Step 3: Put SquashFS Image into LUKS Container

Let's validate the overhead of LUKS:

{% highlight sh %}
$echo $((`sudo blockdev --getsize /dev/loop0`-`sudo blockdev --getsize /dev/mapper/cryptbackup`))
1032
{% endhighlight %}

Sweet! So, the size of the mapped device should be the same as our SquashFS image:

{% highlight sh %}
$sudo blockdev --getsize /dev/mapper/cryptbackup
27217
{% endhighlight %}

Hmmm... Close enough... ;P Perhaps there is some rounding to a full KB or something like that going on. Anywho, at least it is big enough for our 27216 block image. Let's transfer it:

{% highlight sh %}
$sudo dd if=/tmp/cryptbackup.sqsh of=/dev/mapper/cryptbackup bs=512
27216+0 records in
27216+0 records out
13934592 bytes (14 MB) copied, 0.0544451 s, 256 MB/s
{% endhighlight %}

Done and done. To verify that it works we can mount the file system:

{% highlight sh %}
$sudo mkdir /mnt/cryptbackup
$sudo mount /dev/mapper/cryptbackup /mnt/cryptbackup
{% endhighlight %}

If all went well, `ls /mnt/cryptbackup` should now give the contents of the original directory.

To unmount, do:

{% highlight sh %}
$sudo umount /mnt/cryptbackup
$sudo cryptsetup luksClose cryptbackup
$sudo losetup -d /dev/loop0
{% endhighlight %}

Now remove the old SquashFS image `/tmp/cryptbackup.sqsh` and store the LUKS container `/tmp/cryptbackupluks.img` in a safe location. I use a portable external hard disk and the server at home to save the backup images. To mount later, just run a few of the, slightly modified, above commands again:

{% highlight sh %}
$LOOP=`sudo losetup -s -f /tmp/cryptbackupluks.img`
$sudo cryptsetup luksOpen $LOOP cryptbackup
Enter LUKS passphrase:
key slot 0 unlocked.
Command successful.
$sudo mount /dev/mapper/cryptbackup /mnt/cryptbackup
{% endhighlight %}

Now, all that remains is to create some helper scripts to avoid having to write all this every time I want to make a backup...

A drawback of this method is that it takes quite a while to perform the backups. It's not incremental either, so the backups will presumably take longer and longer to make each time (as more data is accumulated). Next thing might be to try to create a base image with SquashFS and then do incremental backups with UnionFS or something... Hmmm......

/M
