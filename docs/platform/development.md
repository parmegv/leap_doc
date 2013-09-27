@title = "Development Environment"

If you are wanting to make local changes to your provider, or want to contribute some fixes back to LEAP, we recommend that you follow this guide to build up a development environment to test your changes first. Using this method, you can quickly test your changes without deploying them to your production environment, while benefitting from the convenience of reverting to known good states in order to retry things from scratch. 

This page will walk you through setting up nodes using [Vagrant](http://www.vagrantup.com/) for convenient deployment testing, snapshotting known good states, and reverting to previous snapshots. 

Requirements
============

* Be a real machine with virtualization support in the CPU (VT-x or AMD-V). In other words, not a virtual machine.
* Have at least 4gb of RAM.
* Have a fast internet connection (because you will be downloading a lot of big files, like virtual machine images).

Install prerequisites
--------------------------------

For development purposes, you will need everything that you need for deploying the LEAP platform:

* LEAP cli
* A provider instance

You will also need to setup a virtualized Vagrant environment, to do so please make sure you have the following
pre-requisites installed:

*Debian & Ubuntu*

Install core prerequisites:

    sudo apt-get install git ruby ruby-dev rsync openssh-client openssl rake make

Install Vagrant in order to be able to test with local virtual machines (typically optional, but required for this tutorial):

    sudo apt-get install vagrant virtualbox

<!--
*Mac OS*

1. Install rubygems from https://rubygems.org/pages/download (unless the `gem` command is already installed).
2. Install Vagrant.dmg from http://downloads.vagrantup.com/
-->


Adding development nodes to your provider
=========================================

Now you will add local-only Vagrant development nodes to your provider. 

You do not need to setup a different provider instance for development, in fact it is more convenient if you do not, but you can if you wish.  If you do not have a provider already, you will need to create one and configure it before continuing (it is recommended you go through the [Quick Start](quick-start) before continuing down this path). 


Create local development nodes
------------------------------

We will add "local" nodes, which are special nodes that are used only for testing. These nodes exist only as virtual machines on your computer, and cannot be accessed from the outside. Each "node" is a server that can have one or more services attached to it. We recommend that you create different nodes for different services to better isolate issues.

While in your provider directory, create a local node, with the service "webapp":

    $ leap node add --local web1 services:webapp
     = created nodes/web1.json
     = created files/nodes/web1/
     = created files/nodes/web1/web1.key
     = created files/nodes/web1/web1.crt

This command creates a node configuration file in `nodes/web1.json` with the webapp service. 

Starting local development nodes
--------------------------------

In order to test the node "web1" we need to start it. Starting a node for the first time will spin up a virtual machine. The first time you do this will take some time because it will need to download a VM image (about 700mb). After you've downloaded the base image, you will not need to download it again, and instead you will re-use the downloaded image (until you need to update the image).

NOTE: Many people have difficulties getting Vagrant working. If the following commands do not work, please visit the [Vagrant page](vagrant) to troubleshoot your Vagrant install before proceeding.

    $ leap local start web
     = created test/
     = created test/Vagrantfile
     = installing vagrant plugin 'sahara'
    Bringing machine 'web1' up with 'virtualbox' provider...
    [web1] Box 'leap-wheezy' was not found. Fetching box from specified URL for
    the provider 'virtualbox'. Note that if the URL does not have
    a box for this provider, you should interrupt Vagrant now and add
    the box yourself. Otherwise Vagrant will attempt to download the
    full box prior to discovering this error.
    Downloading or copying the box...
    Progress: 3% (Rate: 560k/s, Estimated time remaining: 0:13:36)
    ...
    Bringing machine 'web1' up with 'virtualbox' provider...
    [web1] Importing base box 'leap-wheezy'...
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%

Now the virtual machine 'web1' is running. You can add another local node using the same process. For example, the webapp node needs a databasse to run, so let's add a "couchdb" node:

    $ leap node add --local db1 services:couchdb
    $ leap local start
     = updated test/Vagrantfile
    Bringing machine 'db1' up with 'virtualbox' provider...
    [db1] Importing base box 'leap-wheezy'...
    [db1] Matching MAC address for NAT networking...
    [db1] Setting the name of the VM...
    [db1] Clearing any previously set forwarded ports...
    [db1] Fixed port collision for 22 => 2222. Now on port 2202.
    [db1] Creating shared folders metadata...
    [db1] Clearing any previously set network interfaces...
    [db1] Preparing network interfaces based on configuration...
    [db1] Forwarding ports...
    [db1] -- 22 => 2202 (adapter 1)
    [db1] Running any VM customizations...
    [db1] Booting VM...
    [db1] Waiting for VM to boot. This can take a few minutes.
    [db1] VM booted and ready for use!
    [db1] Configuring and enabling network interfaces...
    [db1] Mounting shared folders...
    [db1] -- /vagrant

You now can follow the normal LEAP process and initialize it and then deploy your recipes to it:

    $ leap node init web1
    $ leap deploy web1
    $ leap node init db1
    $ leap deploy db1


Useful local development commands
=================================

There are many useful things you can do with a virtualized development environment. 

Listing what machines are running
---------------------------------

Now you have the two virtual machines "web1" and "db1" running, you can see the running machines as follows:

    $ leap local status
    Current machine states:

    db1                      running (virtualbox)
    web1                     running (virtualbox)

    This environment represents multiple VMs. The VMs are all listed
    above with their current state. For more information about a specific
    VM, run `vagrant status NAME`.

Stopping machines
-----------------

It is not recommended that you leave your virtual machines running when you are not using them. They consume memory and other resources! To stop your machines, simply do the following:

    $ leap local stop web1 db1

Connecting to machines
----------------------

You can connect to your local nodes just like you do with normal LEAP nodes, by running 'leap ssh node'. 

However, if you cannot connect to your local node, because the networking is not setup properly, or you have deployed a firewall that locks you out, you may need to access the graphical console.

In order to do that, you will need to configure Vagrant to launch a graphical console and then you can login as root there to diagnose the networking problem. To do this, add the following to you
$HOME/.leaprc:

    @custom_vagrant_vm_line = 'config.vm.boot_mode = :gui'

and then start, or restart, your local Vagrant node. You should get a VirtualBox graphical interface presented to you showing you the bootup and eventually the login.

Snapshotting machines
---------------------

A very useful feature of local Vagrant development nodes is the ability to snapshot the current state and then revert to that when you need. 

For example, perhaps the base image is a little bit out of date and you want to get the packages updated to the latest before continuing. You can do that simply by starting the node, connecting to it and updating the packages and then snapshotting the node:

    $ leap local start web1
    $ leap ssh web1
    web1# apt-get -u dist-upgrade
    web1# exit
    $ leap local save web1

Now you can deploy to web1 and if you decide you want to revert to the state before deployment, you simply have to reset the node to your previous save:

    $ leap local reset web1

More information
----------------

See `leap help local` for a complete list of local-only commands and how they can be used.
