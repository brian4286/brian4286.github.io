---
title: How To - Work with LVM (Logical Volume Management)
description: How to work with LVM
---

{% highlight nginx %}
[root@server ~]# blkid
{% endhighlight %}


{% highlight text %}
/dev/xvdc1: UUID="db67089c-1455-4ebc-9b25-e872c013452e" TYPE="swap"
/dev/xvda1: UUID="765e3d12-202a-4c30-9d94-e03d9e402fd0" TYPE="ext4"
/dev/xvdb1: PARTLABEL="primary" PARTUUID="cdfcfae9-29e1-4b8c-b38f-8669377d17ee"
/dev/xvdd1: PARTLABEL="primary" PARTUUID="ae1b9a6a-8319-4743-a96e-c23349d3cbe8"
/dev/xvde1: PARTLABEL="primary" PARTUUID="2f497396-0c59-4b3b-bc9c-3c29a19b06fe"
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# pvcreate /dev/xvdb1 /dev/xvdd1 /dev/xvde1
{% endhighlight %}

{% highlight text %}
Physical volume "/dev/xvdb1" successfully created
Physical volume "/dev/xvdd1" successfully created
Physical volume "/dev/xvde1" successfully created
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# blkid
{% endhighlight %}

{% highlight text %}
/dev/xvdc1: UUID="db67089c-1455-4ebc-9b25-e872c013452e" TYPE="swap"
/dev/xvda1: UUID="765e3d12-202a-4c30-9d94-e03d9e402fd0" TYPE="ext4"
/dev/xvdb1: UUID="2yeI89-vSum-nWEV-V2y1-eTtp-7Tay-uo4rvZ" TYPE="LVM2_member" PARTLABEL="primary" PARTUUID="298f6c66-d5ec-4c79-aa93-6e906e62e64d"
/dev/xvdd1: UUID="9Gfbme-k4dn-WlOT-qoGj-nfwB-4DBO-43eqVD" TYPE="LVM2_member" PARTLABEL="primary" PARTUUID="ae1b9a6a-8319-4743-a96e-c23349d3cbe8"
/dev/xvde1: UUID="37dsrq-dU7T-ZqVD-SKhz-1tii-rqwp-dZU2qV" TYPE="LVM2_member" PARTLABEL="primary" PARTUUID="2f497396-0c59-4b3b-bc9c-3c29a19b06fe"
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# pvremove /dev/xvdb1 /dev/xvdd1 /dev/xvde1
{% endhighlight %}

{% highlight text %}
Labels on physical volume "/dev/xvdb1" successfully wiped
Labels on physical volume "/dev/xvdd1" successfully wiped
Labels on physical volume "/dev/xvde1" successfully wiped
{% endhighlight %}

{% highlight nginx %}
  [root@server ~]# pvdisplay
{% endhighlight %}

{% highlight text %}
"/dev/xvdb1" is a new physical volume of "75.00 GiB"
--- NEW Physical volume ---
PV Name               /dev/xvdb1
VG Name               
PV Size               75.00 GiB
Allocatable           NO
PE Size               0   
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               L4t8Dd-dJ0X-Fxg7-6qoX-B0VZ-mm0m-R3f3GO

"/dev/xvde1" is a new physical volume of "75.00 GiB"
--- NEW Physical volume ---
PV Name               /dev/xvde1
VG Name               
PV Size               75.00 GiB
Allocatable           NO
PE Size               0   
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               CFYpzt-PBBA-3DDM-PxOL-kROV-Jne1-Pnsu5u

"/dev/xvdd1" is a new physical volume of "75.00 GiB"
--- NEW Physical volume ---
PV Name               /dev/xvdd1
VG Name               
PV Size               75.00 GiB
Allocatable           NO
PE Size               0   
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               nelzUy-4BCN-5HLp-h0oz-Dne6-mwE7-bqL01N
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# vgcreate jbod /dev/xvdb1 /dev/xvde1 /dev/xvdd1
{% endhighlight %}

{% highlight text %}
Volume group "jbod" successfully created
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# vgcreate jbod /dev/xvdb1 /dev/xvde1 /dev/xvdd1
{% endhighlight %}

{% highlight text %}
Volume group "jbod" successfully created
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# vgdisplay
--- Volume group ---
VG Name               jbod
System ID             
Format                lvm2
Metadata Areas        3
Metadata Sequence No  1
VG Access             read/write
VG Status             resizable
MAX LV                0
Cur LV                0
Open LV               0
Max PV                0
Cur PV                3
Act PV                3
VG Size               224.99 GiB
PE Size               4.00 MiB
Total PE              57597
Alloc PE / Size       0 / 0   
Free  PE / Size       57597 / 224.99 GiB
VG UUID               wKxBdF-H1Kq-fDh3-Xwg1-u5ZU-T8As-r5Zo66
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# vgremove jbod
{% endhighlight %}

{% highlight text %}
Volume group "jbod" successfully removed
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# lvcreate -n Entire_Disk jbod -l 100%VG
{% endhighlight %}

{% highlight text %}
Logical volume "Entire_Disk" created.
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# lvdisplay
--- Logical volume ---
LV Path                /dev/jbod/Entire_Disk
LV Name                Entire_Disk
VG Name                jbod
LV UUID                cSkFsK-rJFN-wpJT-Nm21-JYBB-1GY2-35JUmG
LV Write Access        read/write
LV Creation host, time server, 2016-07-15 17:06:42 -0400
LV Status              available
# open                 0
LV Size                224.99 GiB
Current LE             57597
Segments               3
Allocation             inherit
Read ahead sectors     auto
- currently set to     8192
Block device           253:0
{% endhighlight %}

{% highlight nginx %}
root@server ~]# lvreduce -L25G /dev/jbod/Entire_Disk
{% endhighlight %}
{% highlight text %}
WARNING: Reducing active logical volume to 25.00 GiB
THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce Entire_Disk? [y/n]: y
Size of logical volume jbod/Entire_Disk changed from 224.99 GiB (57597 extents) to 25.00 GiB (6400 extents).
Logical volume Entire_Disk successfully resized.
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# lvcreate -n Half_Disk jbod -l 50%VG
{% endhighlight %}

{% highlight text %}
Logical volume "Half_Disk" created.
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# lvdisplay
{% endhighlight %}

{% highlight text %}
--- Logical volume ---
LV Path                /dev/jbod/Entire_Disk
LV Name                Entire_Disk
VG Name                jbod
LV UUID                cSkFsK-rJFN-wpJT-Nm21-JYBB-1GY2-35JUmG
LV Write Access        read/write
LV Creation host, time server, 2016-07-15 17:06:42 -0400
LV Status              available
# open                 0
LV Size                25.00 GiB
Current LE             6400
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     8192
Block device           253:0

--- Logical volume ---
LV Path                /dev/jbod/Half_Disk
LV Name                Half_Disk
VG Name                jbod
LV UUID                YVt4KA-3F1w-9yG9-H9I3-ffVS-OlBh-tfJKYk
LV Write Access        read/write
LV Creation host, time server, 2016-07-15 17:12:11 -0400
LV Status              available
# open                 0
LV Size                112.49 GiB
Current LE             28798
Segments               2
Allocation             inherit
Read ahead sectors     auto
- currently set to     8192
Block device           253:1
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# lvextend -L50G /dev/jbod/Entire_Disk
{% endhighlight %}

{% highlight text %}
Size of logical volume jbod/Entire_Disk changed from 25.00 GiB (6400 extents) to 50.00 GiB (12800 extents).
Logical volume Entire_Disk successfully resized.
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# lvdisplay
{% endhighlight %}

{% highlight text %}
--- Logical volume ---
LV Path                /dev/jbod/Entire_Disk
LV Name                Entire_Disk
VG Name                jbod
LV UUID                cSkFsK-rJFN-wpJT-Nm21-JYBB-1GY2-35JUmG
LV Write Access        read/write
LV Creation host, time bria9733-eino1, 2016-07-18 17:06:42 -0400
LV Status              available
# open                 0
LV Size                50.00 GiB
Current LE             12800
Segments               3
Allocation             inherit
Read ahead sectors     auto
- currently set to     8192
Block device           253:0

--- Logical volume ---
LV Path                /dev/jbod/Half_Disk
LV Name                Half_Disk
VG Name                jbod
LV UUID                YVt4KA-3F1w-9yG9-H9I3-ffVS-OlBh-tfJKYk
LV Write Access        read/write
LV Creation host, time bria9733-eino1, 2016-07-18 17:12:11 -0400
LV Status              available
# open                 0
LV Size                112.49 GiB
Current LE             28798
Segments               2
Allocation             inherit
Read ahead sectors     auto
- currently set to     8192
Block device           253:1
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# mkfs.ext4 /dev/jbod/Entire_Disk
{% endhighlight %}

{% highlight text %}
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
3276800 inodes, 13107200 blocks
655360 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2162163712
400 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done  
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# mount /dev/jbod/Entire_Disk /mnt/
{% endhighlight %}

{% highlight nginx %}
[root@server ~]# df -h
{% endhighlight %}

{% highlight text %}
Filesystem                    Size  Used Avail Use% Mounted on
/dev/xvda1                     20G  1.7G   17G  10% /
devtmpfs                      233M     0  233M   0% /dev
tmpfs                         242M     0  242M   0% /dev/shm
tmpfs                         242M   21M  222M   9% /run
tmpfs                         242M     0  242M   0% /sys/fs/cgroup
tmpfs                          49M     0   49M   0% /run/user/1001
/dev/mapper/jbod-Entire_Disk   50G   53M   47G   1% /mnt
{% endhighlight %}
