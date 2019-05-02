title: "Understanding Fstab"
date: 2017-06-07 22:35:56
tags: [Linux]
categories: [Engineering, Dev Ops]
---
![](https://wenzhong-1259152588.cos.ap-beijing.myqcloud.com/img/blog/fstab.jpg)

Today I encounter a very strange problem that all recent deployed applications in a specific host fail to start, with a simple error message **"Permission Denied. xxxx.sh could not be executed"**. 

Nonsense! They've run for a very long time with a Jenkins driven deployment, started by a deploy script. And a binary "could not be executed" might be controlled by user & file access flag. Both checked, using `root` user & `755` access flag.

Suddenly I remember that the hard-disk on this host have been re-mounted by Ops a few days ago, this operation might corrupt some filesystem runtime context. So I wrote a simple test script `a.sh`, place it into different mounting point `/tmp` and `/data` and try to execute them via `./a.sh` (not `bash ./a.sh` because in this case we are using `r` way instead of `x` to access the script). And script could be executed under `/tmp` but not `/data`.

Seems we are close to the root cause, but what stop us from executing a binary in different mounting point? The answer is the `fstab`.

<!-- more -->

> fstab is a configuration file that contains information of all the partitions and storage devices in your computer. The file is located under /etc, so the full path to this file is /etc/fstab. /etc/fstab contains information of where your partitions and storage devices should be mounted and how.   

After viewing the content of `/etc/fstab`, we then know why things happen. Here are the content:

{% codeblock lang:conf %}
#
# /etc/fstab
# Created by anaconda on Fri Aug 26 16:02:35 2016
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=7e452929-a3a3-4f1f-80a3-91eced90b453	/	xfs	defaults	0 0
UUID=b797b27d-0499-451e-a1cc-8d7fc014777c	/boot	xfs	defaults	0 0
UUID=a47c3334-ae2c-486f-9756-7172b5570035	swap	swap	defaults	0 0

/dev/mapper/centos-data /data xfs defaults,noexec 0 0
{% endcodeblock %}

There are 6 columns for a mount option, each represent:

* block special device or remote filesystem to be mounted
* mount point of file system
* file system type
* mount option associate with the mount
* dump(8) flag
* boot check sequence  

With help of `man 5 fstab` and `man 8 mount` we could see the `/data` mounting point is bound with a ridiculous `noexec` option. According to man page of `fstab(5)` about fourth field (fs_mntops):
* This field describes the mount options associated with the filesystem.   
* It  is  formatted  as a comma separated list of options.  
* It contains at least the type of mount plus any additional options appropriate to the filesystem type.  
* For documentation  on the available mount options, see mount(8).
* For documentation on the available swap options, see swapon(8).
            
then check `mount(8)`  
* **noexec**  Do not allow direct execution of any binaries on the mounted  filesystem. (Until recently it was possible to run binaries anyway using a command like /lib/ld*.so /mnt/binary. This trick fails since Linux 2.4.25 / 2.6.0.)

**So problem resolved after we remove the ",noexec" option from the `/data` mount point.** The previous statement "(Binary) could not be executed might be controlled by user & file access flag." are not accurate enough. Binary execution could also be controlled by **File system mount options** under "/etc/fstab".

Now it's time to ask why Ops assigned such a flag on this mount...


