Known working combinations
--------------------------

Please consider that using other combinations might work for you as well, these are just the combinations we tried and worked for us:


Debian Wheezy
-------------

* `virtualbox-4.2 4.2.16-86992~Debian~wheezy` from Oracle and `vagrant 1.2.2` from vagrantup.com


Ubuntu Raring 13.04
-------------------

* `virtualbox 4.2.10-dfsg-0ubuntu2.1` from Ubuntu raring and `vagrant 1.2.2` from vagrantup.com

Troubleshooting Vagrant
=======================

To troubleshoot vagrant issues, try going through these steps:

* Try plain vagrant using the [Getting started guide](http://docs.vagrantup.com/v2/getting-started/index.html).
* If that fails, make sure that you can run virtual machines (VMs) in plain virtualbox (Virtualbox GUI or VBoxHeadless). 
  We don't suggest a sepecial howto for that, [this one](http://www.thegeekstuff.com/2012/02/virtualbox-install-create-vm/) seems pretty decent, or you follow the [Oracale Virtualbox User Manual](http://www.virtualbox.org/manual/UserManual.html). There's also specific documentation for [Debian](https://wiki.debian.org/VirtualBox) and for [Ubuntu](https://help.ubuntu.com/community/VirtualBox). If you succeeded, try again if you now can start vagrant nodes using plain vagrant (see first step).
* If plain vagrant works for you, you're very close to using vagrant with leap ! If you encounter any problems now, please [contact us](https://leap.se/en/about-us/contact) or use our [issue tracker](https://leap.se/code)
