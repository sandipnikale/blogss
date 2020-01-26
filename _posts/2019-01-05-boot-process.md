---
layout:     post
title:      Linux Boot Process
date:       2019-01-05
author:     Sandip
summary:    Linux Boot Process
categories: linux
thumbnail:  linux 
tags:
  linux
  boot
  
---

### 1 ] Power On 

  - The system/server hardware firmware, either UEFI (Unified Extensible Firmware Interface) 
    Or BIOS (Basic Input Output System) runs Power On Self Test (POST) when system gets power-on.
                
### 2 ] BIOS/UEFI gets executed

- BIOS/UEFI get executed and checks for proper connectivity of devices and then it would check for memory availability, and finally     locates the boot device.


BIOS/UEFI locates the boot device & loads MBR/GPT (GUID Partition Table) from an active partition


### MBR/GPT loads into memory

 - The 'Master Boot Record' (MBR) would be 512 bytes in size and it would be located on the first sector of the Primary/Active boot device. The MBR loads into the memory at this stage. Partition table is stored within MBR. This loads the boot loader.

- If boot device is larger than 2TB then GPT partitioning scheme would be used instead of MBR. At this stage, GPT details would be read into memory. For more details on GPT
                
### 3]  GRUB (Grand Unified Boot loader). This loads in 4 stages (GRUB2 in case of RHEL7)

{% highlight ruby %}
 First Stage Boot loader 
{% endhighlight %}
- The First Stage Boot loader is read into memory which is of size 446 bytes inside MBR. This is also called “Primary Boot Loader" OR “Stage 1 Loader”. This is a small binary code within 512 bytes of MBR capable of loading Stage 2/1.5 loader.

     *In case of RHEL7, 'boot.img' file gets loaded. This file is of size 446 bytes.

{% highlight ruby %}
Stage 1.5 boot loader 
{% endhighlight %}
- On some hardware platforms this intermediate loader is required. This is sometimes true when the '/boot' partition is above the 1024 cylinder head of the hard drive or when using LBA (Logical Block Addressing) mode. The Stage 1.5 boot loader is found either on the '/boot' partition or in a small part of the MBR.

    *In case of RHEL7, 'core.img' file gets loaded. 

{% highlight ruby %}
Stage 2 boot loader
{% endhighlight %}
- This is also called “Secondary Bootloader” or "Stage 2 Bootloader". The secondary boot loader displays the GRUB menu and command environment. At this stage, user can interrupt the booting process and select specific kernel to boot into and pass additional parameters to the kernel. The files which belongs to stage2 are stored in '/boot/grub' or '/boot/grub2' (in case of RHEL7).


  - The “initramfs” (Initial RAMDISK) gets loaded into memory.



Stage 2 boot loader transfers the control to the kernel.

- The secondary boot loader reads the operating system or kernel as well as the contents of '/boot/sysroot/' into memory. Once GRUB/GRUB2 determines which operating system or kernel to start, it loads it into memory and transfers control of the machine to that operating system.


### Kernel loads into memory

When the kernel is loaded, it immediately initializes and configures the computer's memory and configures the various hardware attached to the system, including processors, I/O subsystems, and storage devices. It then looks for the compressed 'initrd' image in a predetermined location in memory, decompresses it, mounts it, and loads all necessary drivers. Next, it initializes virtual devices related to the file system, such as LVM (Logical Volume Manager) or software RAID (Redundant Array of Independent/Inexpensive Drives) before un-mounting the initrd disk image and freeing up all the memory the disk image once occupied.

Root partition gets mounted read-only

kernel then creates a root device, mounts the root partition read-only, and frees used memory.

Kernel calls init or systemd program

### 4]  init gets loaded into memory Or systemd (in case of RHEL7)

   The '/sbin/init' program (also called init) coordinates the rest of the boot process and configures the environment for user. The same file '/sbin/init' is a symbolic link to '/usr/lib/systemd/systemd' in case of RHEL7. The 'init' or 'systemd' is the first process which starts in.
   
{% highlight ruby %}
 In RHEL6 & earlier versions...
{% endhighlight %}

{% highlight ruby %}
Init calls '/etc/rc.d/rc.sysinit' script.
{% endhighlight %}

- This sets the environment path, starts swap, checks the file systems, and executes all other steps required for system initialization.

{% highlight ruby %}
Init checks the default runlevel in '/etc/inittab' file.
{% endhighlight %}

{% highlight ruby %}
/etc/rc.d/init.d/functions gets executed.
{% endhighlight %}
   - This defines how to start, kill, and determine the PID (Process ID) of a program.


Init program processes all Start (s) and Kill (k) scripts depending on the run level determined.

The init program starts all of the background processes by looking in the appropriate 'rc' directory for the runlevel specified as default in '/etc/inittab' file. The 'rc' directories are numbered which corresponds to the runlevel they represent. When booting to runlevel 5, the init program looks in the '/etc/rc.d/rc5.d' directory to determine which processes to start and stop.


All of the files in '/etc/rc.d/rc5.d' are symbolic links pointing to scripts located in the '/etc/rc.d/init.d' directory.  The symbolic links are used in each of the 'rc' directories so that the runlevels can be reconfigured by creating, modifying, and deleting the symbolic links without affecting the actual scripts they reference. First all “k” scripts gets executed and then all “s” scripts. 


 /etc/inittab script forks an /sbin/mingetty process (Upstart would be used in RHEL 6 for forking mingetty)

- Virtual Consoles gets initiated at this stage depending on run level defined. The '/sbin/mingetty' process opens communication pathways to tty devices, sets their modes, prints the login prompt, accepts the user's username and password, and initiates the login process. 

In run level 5 “/etc/X11/prefdm” script gets executed. Preferred display manager would gets loaded at this stage.   

{% highlight ruby %}
Finally init calls /etc/rc.d/rc.local script.
{% endhighlight %}

### In RHEL7 .....
      
  - The systemd process checks and starts the 'default.target' which would be either 'graphical.target' or 'multi-user.target'.

  - So, basically the 'default.target' is a symbolic link to either 'graphical.target' or 'multi-user target'. 

  - Depending on default target set which is either '/lib/systemd/system/graphical.target' or '/lib/systemd/system/multi-user.target' those respective 'Wants' & 'Requires' directives would be started. So, the target defined in 'Requires' would be stared first. Take a look at the 'graphical.target' file:


```[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

 - So, looking at the above snap of 'graphical.target' file, the 'multi-user.target' should be started first and start graphical service, there is 'Wants' directive which says that 'display-manager.service' should be started along. Like-wise, the 'multi-user.target' file also holds 'Requires' & 'Wants' directive which are dependencies to be started first and later. 

 - The 'multi-user.target' requires 'basic.target' other target as shown below:
 
### systemd ---->   default.target ----> graphical.target ----> multi-user.target -------> sysinit.target .

- The 'sysinit.target' doesn't contain 'Requires' directive, however, got 'Wants' directive:

{% highlight ruby %}
 Wants=local-fs.target swap.target
{% endhighlight %}

- This 'sysinit.target' starts system initialization services, such as mounting file system, enabling swaps, enable logging, start udevd to detect hardware etc. Like-wise there is 'After' directive defined as well in each targets, so as in 'sysinit.target' which is 'local-fs-pre.target' which runs and is responsible for importing network configuration from initramfs and runs file system check on root when required, remounts root file system. 

- So, this process as started, starts from end and at final end it would run either 'graphical.target' or 'multi-user.target' which would present the final login prompt to user.

### 5 ] User Login Screen.


## Skeleton View of Boot Process on x86 BIOS Based System
---------------------------------------------------------------------------------------------------------

BIOS -> MBR -> GRUB (Stage 1 Boot Loader -> Stage 2 Boot Loader) -> Kernel -> Init -> Login

BIOS -> MBR -> GRUB2 (Stage 1 Boot Loader -> Stage 2 Boot Loader) -> Kernel -> Systemd -> Login (in case of RHEL7.x)


![My helpful screenshot](/myblog/assets/boot.png)




*UEFI (Unified Extensible Firmware Interface) based systems would normally implement GPT (Guid Partition Table) partition scheme instead of MBR which supports disks of larger size (more than 2 TB).
