---
title: How To - Use parted for disk partitioning and resizing
description: How to use parted.
keywords: brian4286, Brian Haun, how to, parted, parted, partitions, lvm, fdisk, volume management
---

So you just added that new disk or RAID that is grater then 2TB+. You quickly realize that fdisk will no longer allow you to add a partition. The reason is fdisk does not support GPT partition tables or it is limited to 2TB. GNU parted is going to be your new friend as it supports EFI/GPT partition tables.

You may be thinking, Brian can't I just throw a filesystem or LVM on the raw device? Technically you are correct, and with recent versions of boot loaders you can book LVM directly. If you manage you own environment and no other [SA's](https://en.wikipedia.org/wiki/System_administrator){:target="_blank"} will login, then go ahead and skip it. So someone does not see the device as unused and format it later it may be best to just format the disk.

We are going to assume that the new raw disk is /dev/xvda. parted has a shell much like fdisk but you can also invoke the -s or script flag which we will get into later. First assuming parted is installed, type parted in your shell:

### Parted Shell

{% highlight nginx %}
box:/# parted /dev/xvda
{% endhighlight %}

{% highlight text %}
GNU Parted 3.1
Using /dev/xvda
Welcome to GNU Parted! Type 'help' to view a list of commands.
{% endhighlight %}

### Print partitions

Let's double check to see if any existing partitions exist on this disk. In the parted shell type 'p' to get a the latest printout. You can see we have 4 existing partitions that will need to be removed.

{% highlight nginx %}
(parted) p    
{% endhighlight %}

{% highlight text %}                                                            
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  20.1GB  20.1GB  primary               boot
 2      20.1GB  32.2GB  12.1GB  primary
 3      32.2GB  40.3GB  8053MB  primary
 4      40.3GB  80.5GB  40.3GB  primary

(parted)                                      
{% endhighlight %}

### Label block device

While this is not a large block device, I am going to set this as a GPT partition table by invoking the "mklabel" command. Then I will remove the existing partitions "rm x" and finally we will print out the partition table. Keep in mind that while I have done this, the partition table is not written yet - just like fdisk:

{% highlight nginx %}
(parted) mklabel gpt
(parted) rm 1
(parted) rm 2
(parted) rm 3
(parted) rm 4
(parted) p
{% endhighlight %}

{% highlight text %}                                                             
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags

(parted)                                       
{% endhighlight %}

### Create first partition

Now that we have the old partitions removed, lets start by making our first partition. The command "mkpart" is invoked and we will add a "primary" partition starting from the first usable block "1" and consume "50%" of the disk and finally the "p" will print the output. Since this is a external drive and will be managed my LVM, you have no real need to create multiple partitions other then to show you it can be done. You can interchangeably use percentages (%), MB or GB, for ease of this how to I am going to show you percentages.

{% highlight nginx %}
(parted) mkpart primary 1 50% p
{% endhighlight %}

{% highlight text %}
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  40.3GB  40.3GB               primary

(parted)
{% endhighlight %}

### Create second partition

Here we will invoke the same command but start at "50%" of the disk and end at "100%". At this point 100% of the disk is split evenly between two partitions.

{% highlight nginx %}
(parted) mkpart primary 50% 100% p
{% endhighlight %}

{% highlight text %}
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  40.3GB  40.3GB               primary
 2      40.3GB  80.5GB  40.3GB               primary

(parted)
{% endhighlight %}

### Enable the flags

Since I want to use lvm, I am going to set a [flag (GNU parted manual)](https://www.gnu.org/software/parted/manual/html_chapter/parted_2.html#SEC28){:target="_blank"}. We "set" the flag on partition "2" which will be "lvm" flag and turn it "on".

{% highlight nginx %}
(parted) set 2 lvm on p
{% endhighlight %}

{% highlight text %}
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  40.3GB  40.3GB               primary
 2      40.3GB  80.5GB  40.3GB               primary  lvm

(parted)
{% endhighlight %}

### Flag the second partition

To show how we can set a separate flag, I will enable the "boot" flag on the "1"st partition, as if I were going to use this as boot device. Finally to write all the partition tables I will just "quit". You can run the parted shell again and invoke "p" to verify the changes have been written.

{% highlight nginx %}
GNU Parted 3.1
Using /dev/xvda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) set 1 boot on p                                                  
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  40.3GB  40.3GB               primary  boot
 2      40.3GB  80.5GB  40.3GB               primary  lvm

(parted) quit
{% endhighlight %}

### Scripted, verify any existing partitions

The parted command as I noted previously has the -s or script flag. This makes it far easier in most cases to do the same commands but directly in the shell, which is cleaner and easier for me. You may want to use this if you are automating.

What we will do here is after invoking the script flag (-s) is couple all the individual commands together into one long command.

{% highlight nginx %}
box:/# parted /dev/xvda -s p
{% endhighlight %}

{% highlight text %}
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start  End  Size  Type  File system  Flags
{% endhighlight %}

### Fully script partitioning

Now that we are sure we are working on a raw disk, lets do this in a one-liner...

{% highlight nginx %}
box:/# parted /dev/xvde -s mklabel gpt mkpart primary 1 50% mkpart primary 50% 100% set 1 boot on set 2 lvm on p
{% endhighlight %}

{% highlight text %}
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 80.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  40.3GB  40.3GB               primary  boot
 2      40.3GB  80.5GB  40.3GB               primary  lvm
{% endhighlight %}

Now that you are partitioned the disk, read my how to on LVM.
