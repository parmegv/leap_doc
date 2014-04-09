@nav_title = 'Hacking'
@title = 'Setting up a development environment'

Setting up a development environment
====================================

This document covers how to get an environment ready to contribute code
to Bitmask.

Cloning the repo
----------------

> **note**
>
> Stable releases are in *master* branch. Development code lives in
> *develop* branch.

    git clone https://leap.se/git/bitmask_client
    git checkout develop

Dependencies
------------

Bitmask depends on these libraries:

- python 2.6 or 2.7
- qt4 libraries
- openssl
- [openvpn](http://openvpn.net/index.php/open-source/345-openvpn-project.html)

### Install dependencies in a Debian based distro

In debian-based systems:

    $ sudo apt-get install git python-dev python-setuptools python-virtualenv python-pip libssl-dev python-openssl libsqlite3-dev g++ openvpn pyside-tools python-pyside libffi-dev


Working with virtualenv
-----------------------

### Intro

*Virtualenv* is the *Virtual Python Environment builder*.

It is a tool to create isolated Python environments.

> The basic problem being addressed is one of dependencies and versions,
and indirectly permissions. Imagine you have an application that needs
version 1 of LibFoo, but another application requires version 2. How can
you use both these applications? If you install everything into
`/usr/lib/python2.7/site-packages` (or whatever your platform's standard
location is), it's easy to end up in a situation where you
unintentionally upgrade an application that shouldn't be upgraded.

Read more about it in the [project documentation
page](http://www.virtualenv.org/en/latest/virtualenv.html).

### Create and activate your dev environment

You first create a virtualenv in any directory that you like:

    $ mkdir ~/Virtualenvs
    $ virtualenv ~/Virtualenvs/bitmask
    $ source ~/Virtualenvs/bitmask/bin/activate
    (bitmask)$

Note the change in the prompt.

### Avoid compiling PySide inside a virtualenv

If you attempt to install PySide inside a virtualenv as part of the rest
of the dependencies using pip, basically it will take ages to compile.

As a workaround, you can run the following script after creating your
virtualenv. It will symlink to your global PySide installation (*this is
the recommended way if you are running a debian-based system*):

    $ pkg/postmkvenv.sh

A second option if that does not work for you would be to install PySide
globally and pass the `--system-site-packages` option when you are creating
your virtualenv:

    $ apt-get install python-pyside
    $ virtualenv --system-site-packages .

After that, you must export `LEAP_VENV_SKIP_PYSIDE` to skip the
installation:

    $ export LEAP_VENV_SKIP_PYSIDE=1

And now you are ready to proceed with the next section.

### Install python dependencies

You can install python dependencies with `pip`. If you do it inside your
working environment, they will be installed avoiding the need for
administrative permissions:

    $ pip install -r pkg/requirements.pip

This step is not strictly needed, since the `setup.py develop` in the next
paragraph with also fetch the needed dependencies. But you need to know abou it:
when you or any person in the development team will be adding a new dependency,
you will have to repeat this command so that the new dependencies are installed
inside your virtualenv.

Install Bitmask
---------------

Normally we would install the `leap.bitmask` package as any other package
inside the virtualenv.
But, instead, we will be using setuptools **development mode**. The difference
is that, instead of installing the package in a permanent location in your
regular installed packages path, it will create a link from the local
site-packages to your working directory. In this way, your changes will always
be in the installation path without need to install the package you are working
on.::

    (bitmask)$ python2 setup.py develop

After this step, your Bitmask launcher will be located at
`~/Virtualenvs/bitmask/bin/bitmask`, and it will be in the path as long as you
have sourced your virtualenv.

Make resources
--------------

We also need to compile the resource files::

    (bitmask)$ make resources

You need to repeat this step each time you change a `.ui` file.

Copy script files
-----------------

The openvpn invocation expects some files to be in place. If you have not
installed `bitmask` from a debian package, you must copy these files manually
by now:

    $ sudo mkdir -p /etc/leap
    $ sudo cp pkg/linux/resolv-update /etc/leap


Running openvpn without root privileges
---------------------------------------

In linux, we are using `policykit` to be able to run openvpn without root
privileges, and a policy file is needed to be installed for that to be
possible.
The setup script tries to install the policy file when installing bitmask
system-wide, so if you have installed bitmask in your global site-packages at
least once it should have copied this file for you.

If you *only* are running bitmask from inside a virtualenv, you will need to
copy this file by hand:

    $ sudo cp pkg/linux/polkit/net.openvpn.gui.leap.policy /usr/share/polkit-1/actions/


Running!
--------

If everything went well, you should be able to run your client by invoking
`bitmask`. If it does not get launched, or you just want to see more verbose
output, try the debug mode:

   (bitmask)$ bitmask --debug


Using automagic helper script
-----------------------------

You can use a helper script that will get you started with bitmask and all the related repos.

1. install system dependencies
2. download automagic script
3. run it :)

Commands so you can copy/paste:

    $ mkdir bitmask && cd bitmask
    $ wget https://raw.githubusercontent.com/leapcode/bitmask_client/develop/pkg/scripts/bootstrap_develop.sh
    $ chmod +x bootstrap_develop.sh
    $ ./bootstrap_develop.sh init  # use help parameter for more information

This script allows you to get started, update and run the bitmask app with all its repositories.
