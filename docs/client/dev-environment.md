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

    sudo apt-get install git python-dev python-setuptools python-virtualenv python-pip libssl-dev python-openssl libsqlite3-dev g++ openvpn pyside-tools python-pyside libffi-dev


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

    $ virtualenv </path/to/new/environment>
    $ source </path/to/new/environment>/bin/activate

### Avoid compiling PySide inside a virtualenv

If you attempt to install PySide inside a virtualenv as part of the rest
of the dependencies using pip, basically it will take ages to compile.

As a workaround, you can run the following script after creating your
virtualenv. It will symlink to your global PySide installation (*this is
the recommended way if you are running a debian-based system*):

    $ pkg/postmkvenv.sh

A second option if that does not work for you would be to install PySide
globally and pass the `--site-packages` option when you are creating
your virtualenv:

    $ apt-get install python-pyside
    $ virtualenv --site-packages .

After that, you must export `LEAP_VENV_SKIP_PYSIDE` to skip the
installation:

    $ export LEAP_VENV_SKIP_PYSIDE=1

And now you are ready to proceed with the next section.

### Install python dependencies

You can install python dependencies with `pip`. If you do it inside your
working environment, they will be installed avoiding the need for
administrative permissions:

    $ pip install -r pkg/requirements.pip


Using automagic helper script
-----------------------------

You can use a helper script that will get you started with bitmask and all the related repos.

1. install system dependencies
2. download automagic script
3. run it :)

Commands so you can copy/paste:

    mkdir bitmask && cd bitmask
    wget https://raw.githubusercontent.com/leapcode/bitmask_client/develop/pkg/scripts/bootstrap_develop.sh
    chmod +x bootstrap_develop.sh
    ./bootstrap_develop.sh init  # use help parameter for more information

This script allows you to get started, update and run the bitmask app with all its repositories.
