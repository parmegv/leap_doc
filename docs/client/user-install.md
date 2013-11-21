@nav_title = 'Installing'
@title = 'Installing Bitmask'

Installation
============

This part of the documentation covers the installation of Bitmask. We
assume that you want to get it properly installed before being able to
use it. But we can we wrong.

Standalone bundle
-----------------

Maybe the quickest way of running Bitmask in your machine is using the
standalone bundle. That is the recommended way to use Bitmask for the
time being.

You can get the latest bundles, and their matching signatures at [the
downloads page](https://downloads.leap.se/client/).

### Linux

-   [Linux 32 bits
    bundle](https://downloads.leap.se/client/linux/Bitmask-linux32-latest.tar.bz2)
    ([signature](https://downloads.leap.se/client/linux/Bitmask-linux32-latest.tar.bz2.asc))
-   [Linux 64 bits
    bundle](https://downloads.leap.se/client/linux/Bitmask-linux64-latest.tar.bz2)
    ([signature](https://downloads.leap.se/client/linux/Bitmask-linux64-latest.tar.bz2.asc))

### OSX

-   [OSX
    bundle](https://downloads.leap.se/client/osx/Bitmask-OSX-latest.dmg)
    ([signature](https://downloads.leap.se/client/osx/Bitmask-OSX-latest.dmg.asc))

### Windows

-   [Windows 32 bits
    bundle](https://downloads.leap.se/client/windows/Bitmask-win32-latest.zip)
    ([signature](https://downloads.leap.se/client/windows/Bitmask-win32-latest.zip.asc))

### Signature verification

For the signature verification you can use :

    $ gpg --verify Bitmask-linux64-latest.tar.bz2.asc

Asuming that you downloaded the linux 64 bits bundle.

Debian package
--------------

> **warning**
>
> The debian package that you can currently find in the leap
> repositories is from the stable, 0.2.0 release, which is now outdated.
> You are encouraged to install the development version or the
> standalone bundles while we upload the newest packages.

First, you need to bootstrap your apt-key:

    # gpg --recv-key 0x1E34A1828E207901 0x485B12FA218E81EB
    # gpg --list-sigs 0x1E34A1828E207901
    # gpg --list-sigs 0x485B12FA218E81EB
    # gpg -a --export 0x1E34A1828E207901  | sudo apt-key add - 

Add the archive to your sources.list:

    # echo "deb http://deb.leap.se/debian unstable main" >> /etc/apt/sources.list
    # apt-get update
    # apt-get install leap-keyring

And then you can happily install bitmask:

    apt-get install bitmask

Distribute & Pip
----------------

> **note**
>
> The rest of the methods described below in this page assume you are
> familiar with python code, and you can find your way through the
> process of dependencies install. For more insight, you can also refer
> to the sections setting up a working environment or
> fetching latest code for testing.

![image](https://pypip.in/v/leap.bitmask/badge.png%0A%20%20%20%20%20:target:%20https://crate.io/packages/leap.bitmask)

Installing Bitmask is as simple as using
[pip](http://www.pip-installer.org/) for the already released versions :

    $ pip install leap.bitmask

Show me the code!
-----------------

For the brave souls that can find their way through python packages, you can
get the code from LEAP public git repository :

    $ git clone https://leap.se/git/bitmask_client

Or from the github mirror :

    $ git clone https://github.com/leapcode/bitmask_client.git

Once you have grabbed a copy of the sources, and installed all the base
dependencies, the recommended way to proceed is to install things in a virtualenv.

    $ virtualenv bitmask && source bitmask/bin/activate
    $ make  # compile the resources
    $ python2 setup.py install

Or you can install it into your global site-packages easily :

    $ make  # compile the resources
    $ sudo python2 setup.py install

WARNING: installing a package in the global site-packages can be harmful because the dependency installation can overwrite some of the existing packages.
