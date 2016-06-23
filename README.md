 

 

### Netbooting into iPXE from a MacOS X Netboot Server

I spent the last few days trying to figure out how NetBooting works on a Mac
which is different from traditional PXE netbooting. The goal was, to be able to
boot from a number of recovery boot images like acronis, clonezilla etc. I
started by using a bootable iso created with
[rom-o-matic](https://rom-o-matic.eu) on a virtual machine for testing. The menu
settings are coming from a http-server, and are built from php logic.

I tested the setup to be working on any type of computer, by putting a boot
image on a normal PXE server as well. After a NetBoot on my macbook, I
discovered the keyboard does not work when you’re in the menu. That s\#ck\$!

For now I just put in a workaround. The boot logic in PHP detects the vendor,
and boots directly to a specific boot image when booting a Mac. Too bad this
isn’t working correctly on a mac. Maybe in the future…I hope.



[​](​)Special thanx go out to [Andreas Fink](http://www.fink.org/), as he provided
an almost perfect script for building the nbi image, which I adapted and edited
to a working state, and [skunkie](https://github.com/skunkie/ipxe-phpmenu), for
a wonderful PHP-to-ipxe menu system for ease of management.

 

#### Step 1: you need a NetBoot Server.

I use a MacOS X Server which is next to it. You can also
use [BSDPy](https://bitbucket.org/bruienne/bsdpy) under Linux.

Also very useful tended to be [Deploy Studio](http://www.deploystudio.com/) as
it allowed me to boot into a mini OS X and then remote control the Mac via
RemoteDesktop to set up a startup volume or do things like partitioning etc.

 

#### Step 2: download IPXE sources

You should do this on a Linux machine as the current version of IPXE doesnt
build under OS X due to some GCC compiler flags which make the OS X compiler
spill out errors.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
git clone git://git.ipxe.org/ipxe.git
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

#### Step 3: Edit the build configuration

To use ipxe-phpmenu, uncomment these options in the header files:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
config/general.h
#define IMAGE_PNG       /* PNG image support */
#define PARAM_CMD       /* Form parameter commands */
#define CONSOLE_CMD     /* Console command */
config/console.h:
#define CONSOLE_FRAMEBUFFER /* Graphical framebuffer console */
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

#### Step 4:Install dependencies

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Debian/Ubuntu:
    apt-get install binutils-dev curl libiberty-dev zip

under Fedora/Redhat/CentOS it's probably something like 
    yum install binutils-devel curl libiberty-devel zip
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

#### Step 6:Install PHP menu

Save the contents of this repository to a subfolder **iPXE** on your webserver,
so  
the boot url will be something like **http://192.168.2.100/iPXE/boot.php**

I changed the PHP logic to be working from a subfolder.

 

#### Step 5: Download and run my build script and run it

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd ipxe/src
wget https://raw.githubusercontent.com/mikesmth/macos-netboot-ipxe/master/build/build_ipxe_nbi.sh
#bash ./build_ipxe_nbi.sh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This would build the 64bit ipxe efi binary and bundle it with the needed files
to make up a netbootable nbi folder as OS X Server requires it, however, we want
to build a IPXE booter which automatically loads an URL and executes the IPXE
script there, so we pass it on the command like like this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bash ./build_ipxe_nbi.sh http://bootserver/boot.php
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The output you get is now a ipxe.nbi.zip which you can copy on to your OS X
server, unpack it there and put it into /Library/NetBoot/NetBootSP0/

 

ipxe-phpmenu
============

This is a PHP based iPXE menu with submenus and with authentication and access
restriction.

Set the *067 Bootfile Name* option in the DHCP *Server Options* to the file
*boot.php*, e.g. `http://ipxe.domain.corp/boot.php`. More information about
configuring the DHCP Server Options can be found at
http://ipxe.org/howto/chainloading.

### Features of ipxe-phpmenu

-   Authentication menu

-   iPXE main menu and its submenus are easily generated by PHP

-   Users see only those menu items that they have access to (set by a character
    marker-sequence)

-   Kickstart file installation of CentOS 6 with configurable network settings
    and hostname

 

### Users and access groups

Users and access groups are set in the file *config.php*.

 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<?php

$USERS = array('user:pass' => 'm', # user1
               'admin:admin' => 'a', # user2
              );
#
# to allow a user to see a menu item, it would be specified as:
#
# title("Disk Utilities", "ma");
# item("GNOME Partition Editor (GParted)", "gparted", "gparted.php", "ma");
# item("Clonezilla", "clonezilla", "clonezilla.php", "a");
# item("Acronis ...", "acronis", "submenu_acronis.php", "a");
# in above example the Disk Utilities title will be visible to user and admin
# user will see just the first menu item
 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 
