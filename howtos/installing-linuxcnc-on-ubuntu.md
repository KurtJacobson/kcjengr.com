---
layout: page
title: Installing LinuxCNC on Ubuntu
date: 2016-01-09 12:00:00 +0500
---

**Page Content**
* TOC
{:toc}


## Introduction
I have LinuxCNC running on Ubuntu. I am using Ubuntu mainly because it plays
better than Debian with my elo touch screen and seems more touch friendly in
general. If you have no good reason to use Ubuntu it is much easier to install
Debian and LinuxCNC at the same time using the Live/Install Image. 

The steps on this page are mostly for my own reference, use with caution.

## Installing Ubuntu
I am assuming the following steps will be performed on a windoze computer.

### Download Ubuntu
LinuxCNC will run on any flavor of Ubuntu Precise 12.04 x86 (32-bit). You can 
find the images here: <http://releases.ubuntu.com/precise/>  
If you are impatient here is a direct download link:
<http://releases.ubuntu.com/precise/ubuntu-12.04.5-desktop-i386.iso>

### Create a Bootable USB drive
Download Rufus, an excellent utility for creating bootable USB drives.
It does not require installation: <https://rufus.akeo.ie/>

Select a 2Gb or larger USB thumb drive. All the information on it will be
overwritten so use a new one or one you don't care about. (After you are done
with the USB drive you can reformat it and use it for other purposes. However I
would suggest leaving Ubuntu on it and putting it on your key ring. It may come
in handy at some point to recover files if your Windoze PC dies on you!)

Follow the instructions to create the bootable USB here: 
<https://www.ubuntu.com/download/desktop/create-a-usb-stick-on-windows>

Plug the USB into the target computer and boot from the USB drive. If you want
to have both Windoze and Linux on the same PC it is best to install Windoze first.

You can run Ubuntu from the flash drive and get a feel for it before installing,
but none of the changes you make or programs you install will be saved.

Install Ubuntu. If this installation is only for machine control enter a short,
easy to type password when prompted.

Now, time to remove some of the Ubuntu bloatware to make it faster and more
suitable for machine control.

### Remove Bloatware
Ubuntu 12.04 comes with a lot of bloatware that is not necessary and can make it
very slow on older hardware. I ran the following commands which made Ubuntu run
much faster on my Intel D525MW and also made the appearance and work flow more
to my taste.

**1) Open a Terminal**

Hitting ctr + alt + t is the quickest method, but you can also find the terminal
under Accessories in the program list.  


**2) Replace Unity with Gnome Shell**

First, get rid of the Unity desktop shell which is very resource intensive and
IMHO ugly and unintuitive.

Install Gnome shell.  
```sudo apt-get install gnome-shell```  

Log out and choose **"gnome classical no effects"** and log back in.  

Now uninstall Unity and Compiz since we don't need them anymore.  
```sudo apt-get remove unity```  
```sudo apt-get remove compiz```  


**3)  Replace Software Center with Synaptic**

Uninstall the add ridden and confusing Ubuntu Software Center  
```sudo apt-get remove software-center```  
```sudo apt-get autoremove software-center```

Install synaptic  
```sudo apt-get install synaptic```


**4) Remove Ubuntu One**

Ubuntu one was a cloud storage app similar Dropbox or Google Drive. I believe
it is now defunct, but for some reason it was still using resources. Away with it.  
```sudo apt-get autoremove ubuntuone-client python-ubuntuone-control-panel```


**5) Remove Indicator-messages**

This is something that appears in the upper bar of the desktop and lets you
manage messaging and other social media stuff. Don't foresee using that anywhere,
much less on a machine control and it it takes up a good chunk of memory. Away with it too.  
```sudo apt-get autoremove indicator-messages```  
```sudo apt-get autoremove telepathy-indicator```


**6)  Remove Deja-dup**

This is a backup tool. It might be useful but I don't plan on using it so I removed it.  
```sudo apt-get autoremove deja-dup```

**7) Remove Zeitgeist**

According to Wikipedia:  
_Zeitgeist is a software service which logs the users's activities and events,
anywhere from files opened to websites visited and conversations. It makes this
information readily available for other applications to use in the form of
timelines and statistics._

Nuff said. Lets have no more of that.  
```sudo apt-get autoremove zeitgeist-core```


**8) Remove Totem and Rhythmbox**

Totem and Rhythmbox are multimedia players. It is up to you whether you want
to keep them, they should not have much impact on performance but neither are
they necessary.  
```sudo apt-get remove totem```  
```sudo apt-get remove rhythmbox```


**9) Restart**

It is probably a good idea to restart. To restart form terminal say  
```sudo shutdown -r now```

If you want to shutdown (power off) replace the -r which stands for restart
with an -h for halt.  
```sudo shutdown -h now```

You could also shutdown graphically by going to the "gear" in the upper right
of the screen and selecting Shutdown or physically by hitting the power button
and selecting restart from the menu. Many choices . . .

## Installing LinuxCNC
Run the following to bring the machine up to date with the latest packages in
Ubuntu Precise.  
```sudo apt-get update```  
```sudo apt-get dist-upgrade```

Add the LinuxCNC Archive Signing Key to your apt keyring by running  
```sudo apt-key adv --keyserver hkp://keys.gnupg.net --recv-key 3cb9fd148f374fef```

_**NOTE:** Several people have reported that they could not successfully add the
signing key using the above command. If you have this problem see "Alternate
ways to add Signing Key" at the bottom of this page._

Add a new apt source  
```sudo add-apt-repository "deb http://linuxcnc.org/ precise base 2.7-rtai"```


Fetch the package list from linuxcnc.org.  
```sudo apt-get update```

Install the RTAI kernel and modules by running  
```sudo apt-get install linux-image-3.4-9-rtai-686-pae rtai-modules-3.4-9-rtai-686-pae```

If you want to be able to build LinuxCNC from source using the git repo, also run  
```sudo apt-get install linux-headers-3.4-9-rtai-686-pae```

### Booting into the RTAI kernel
At this point the RTAI kernel should be installed, but it will not be used by
default so every time you boot you will have to specifically select it. We will
fix that in a minute, but first lets make sure that Linux will boot successfully
with the new kernel.

Restart the machine  
```sudo shutdown -r now```

Right after the bios screen disappears press and hold the Shift Key to enter
the GRUB bootloader, if the Ubuntu logo pops up you were too late.

Grub will present a list of boot options, navigate to  
```Previous Linux versions > Ubuntu, with Linux 3.4-9-rtai-686-pae```

Hit enter and the machine should boot with the 3.4-9-rtai-686-pae RTAI kernel.
To verify this run  
```uname -r```

### Making RTAI the default kernel
If all went well we should make Linux boot with the 3.4-9-rtai-686-pae kernel
by default. To to this we edit /etc/default/grub

We need root permissions to edit this file, so if you are comfortable with the
terminal say  
```sudo gedit /etc/default/grub```

If you prefer a graphical approach open a root file browser by saying  
```sudo nautilus```
   
and then browse to and open /etc/default/grub

Edit the GRUB_DEFAULT line to read  
```GRUB_DEFAULT="Previous Linux versions>Ubuntu, with Linux 3.4-9-rtai-686-pae"```

**Important** The changes will not be registered unless you say  
```sudo update-grub```

Restart and run ```uname -r``` again to make sure the machine booted with 3.4-9-rtai-686-pae

### Install LinuxCNC
To install Run  
```sudo apt-get install linuxcnc```

## Alternate Methods

### Installing as Run in Place
Running ```sudo apt-get install linuxcnc``` will install the latest LinuxCNC
release, currently 2.7.9. However, there are times when it is necessary or
desirable to have the very latest version of LinuxCNC. For example, Hazzy
requires LinuxCNC 2.8, which has not yet been released. Although it is possible
to install 2.8 from the build bot I find it much more convenient to build from
source and use LinuxCNC as Run in Place. The advantage of Run in Place is that
it is possible to have multiple versions installed at the same time and be able
to switch between them as needed. Here is an excellent tutorial for and RIP
installation: <http://www.wiki.eusurplus.com/index.php?title=LinuxCNC_Run_In_Place>


### Adding a Signing Key
I don't know much about these things, but a I have read that a firewall might
cause adding the Signing Key to fail. If so maybe this line which forces it to
use port 80 will work.  
```sudo apt-key adv --keyserver hkp://keys.gnupg.net:80 --recv-key 3cb9fd148f374fef```

If still no joy you could get the signing key manually here: <http://pgp.mit.edu>  
Enter [```0x3cb9fd148f374fef```](http://pgp.mit.edu/pks/lookup?op=get&search=0x3CB9FD148F374FEF)
in the search field and it should return a link to the key. Open the link (there
might be more than one, any should do) and copy everything but the heading at
the top to a file and name it something, key.txt, I don't think it maters.

Assuming the file is on the Desktop open a terminal and cd to the Desktop
(cd Desktop). Then say  
```sudo apt-key add key.txt```

That should work, in theory, but I have not tested it. Let me know if there is
a better method, or better yet, edit this wiki to include it!

## References
<http://linuxcnc.org/docs/2.7/html/getting-started/getting-linuxcnc.html>
