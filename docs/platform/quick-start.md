LEAP Platform Quick Start
=====================================================

This is a very minimal tutorial that will walk you through the process of creating and deploying a service provider running the LEAP platform.

All the commands in this tutorial are run on your home desktop or laptop computer, or whatever machine you want to do your system administration from.

What you will need:

* A machine running Debian (or derivatives)
* At least 4gb of RAM
* A fast internet connection

Installation
--------------------------------

Install prerequisites:

    sudo apt-get install git ruby ruby-dev rsync openssh-client openssl rake

Install Vagrant in order to be able to test with local virtual machines (typically optional, but required for this tutorial):

    sudo apt-get install vagrant virtualbox

This tutorial should work with ruby1.8, but has only been tested using ruby1.9.

Install the `leap` command:

    sudo gem install leap_cli

Alternately, you can install `leap` from source:

    git clone git://leap.se/leap_cli.git
    cd leap_cli
    rake build

Then, install as root user (recommended):

    sudo rake install

Or, install as unprivileged user:

    rake install
    # watch out for the directory leap is installed to, then i.e.
    sudo ln -s ~/.gem/ruby/1.9.1/bin/leap /usr/local/bin/leap

With both methods, you can use now /usr/local/bin/leap, which in most cases will be in your $PATH.

Create a provider instance
---------------------------------------

A provider instance is a directory tree, usually stored in git, that contains everything you need to manage an infrastructure for a service provider. In this case, we create one for bitmask.net and call the instance directory 'bitmask'.

    mkdir -p ~/leap/bitmask

Now, we will initialize this directory to make it a provider instance. Your provider instance will need to know where it can find local copy of the git repository leap_platform, which holds the puppet recipes you will need to manage your servers. Typically, you will not need to modify leap_platform.

    cd ~/leap/bitmask
    leap new .

The `leap new` command will ask you for several required values:

* domain: The primary domain name of your service provider. In this tutorial, we will be using "bitmask.net".
* name: The name of your service provider.
* contact emails: A comma separated list of email addresses that should be used for important service provider contacts (for things like postmaster aliases, Tor contact emails, etc).
* platform: The directory where you have a copy of the `leap_platform` git repository checked out. If it doesn't exist, it will be downloaded for you.

You may want to poke around and see what is in the files we just created. For example:

    cat provider.json

Optionally, commit your provider directory using the version control software you fancy. For example:

    git init
    git add .
    git commit -m "initial commit"

Now add yourself as a privileged sysadmin who will have access to deploy to servers:

    leap add-user --self

NOTE: in most cases, `leap` must be run from within a provider instance directory tree (e.g. ~/leap/bitmask).

Now generate required X509 certificates and keys:

    leap cert ca
    leap cert csr

To see details about the keys and certs that the prior two commands created, you can use `leap inspect` like so:

    leap inspect files/ca/ca.crt


Edit provider.json configuration
--------------------------------------

There are a few required settings in provider.json. At a minimum, you must have:

    {
      "domain": "bitmask.net",
      "name": "Bitmask",
      "contacts": {
        "default": "email1@domain.org, email2@domain.org"
      }
    }

For a full list of possible settings, you can use `leap inspect` to see how provider.json is evaluated after including the inherited defaults:

    leap inspect provider.json

Create nodes
---------------------

A "node" is a server that is part of your infrastructure. Every node can have one or more services associated with it. Some nodes are "local" and used only for testing. These local nodes exist only as virtual machines on your computer and cannot be accessed from outside (see `leap help local` for more information).

Create a local node, with the service "webapp":

    leap node add --local web1 services:webapp

This created a node configuration file, but it did not create the virtual machine. In order to test our node "web1", we need to first spin up a virtual machine. The next command will probably take a very long time, because it will need to download a VM image (about 700mb).

    leap local start

Now that the virtual machine for web1 is running, you need to initialize it and then deploy the recipes to it. You only need to initialize a node once, but there is no harm in doing it multiple times. These commands will take a while to run the first time, as it needs to update the package cache on the new virtual machine.

    leap node init web1
    leap deploy web1

That is it, you should now have your first running node. However, the LEAP web application requires a database to run, so let's add a "couchdb" node:

    leap node add --local db1 services:couchdb
    leap local start
    leap node init db1
    leap deploy db1

What is going on here?
--------------------------------------------

First, some background terminology:

* **puppet**: Puppet is a system for automating deployment and management of servers (called nodes).
* **hiera files**: In puppet, you can use something called a 'hiera file' to seed a node with a few configuration values. In LEAP, we go all out and put *every* configuration value needed for a node in the hiera file, and automatically compile a custom hiera file for each node.

When you run `leap deploy`, a bunch of things happen, in this order:

1. **Compile hiera files**: The hiera configuration file for each node is compiled in YAML format and saved in the directory `hiera`. The source material for this hiera file consists of all the JSON configuration files imported or inherited by the node's JSON config file.
* **Copy required files to node**: All the files needed for puppet to run are rsync'ed to each node. This includes the entire leap_platform directory, as well as the node's hiera file and other files needed by puppet to set up the node (keys, binary files, etc).
* **Puppet is run**: Once the node is ready, leap connects to the node via ssh and runs `puppet apply`. Puppet is applied locally on the node, without a daemon or puppetmaster.

You can run `leap -v2 deploy` to see exactly what commands are being executed.

<!-- See [under the hood](under-the-hood) for more details. -->

Additional commands
-------------------------------------------

Here are a few useful commands you can run on your new local nodes:

* `leap ssh web1` -- SSH into node web1 (requires `leap node init web1` first).
* `leap list` -- list all nodes.
* `leap list --print ip_address` -- list a particular attribute of all nodes.
* `leap local reset web1` -- return web1 to a pristine state.
* `leap local stop` -- stop all local virtual machines.
* `leap local status` -- get the running state of all the local virtual machines.
* `leap cert update` -- generate new certificates if needed.

See the full command reference for more information.

Node filters
-------------------------------------------

Many of the `leap` commands take a "node filter". You can use a node filter to target a command at one or more nodes.

A node filter consists of one or more keywords, with an optional "+" before each keyword.

* keywords can be a node name, a service type, or a tag.
* the "+" before the keyword constructs an AND condition
* otherwise, multiple keywords together construct an OR condition

Examples:

* `leap list openvpn` -- list all nodes with service openvpn.
* `leap list openvpn +production` -- only nodes of service type openvpn AND tag production.
* `leap deploy webapp openvpn` -- deploy to all webapp OR openvpn nodes.
* `leap node init vpn1` -- just init the node named vpn1.

Running on real hardware
-----------------------------------

The steps required to initialize and deploy to nodes on the public internet are basically the same as we have seen so far for local testing nodes. There are a few key differences:

* Obviously, you will need to acquire a real or virtual machine that you can SSH into remotely.
* When creating the node configuration, you should give it the tag "production" if the node is to be used in your production infrastructure.
* When creating the node configuration, you need to specify the IP address of the node.

For example:

    leap node add vpn1 tags:production services:openvpn ip_address:4.4.4.4

Also, running `leap node init NODE_NAME` on a real server will prompt you to verify the fingerprint of the SSH host key and to provide the root password of the server NODE_NAME. You should only need to do this once.

What services do I need
-----------------------------------

Run `leap list` to see a list of available services.

TODO: link to another document that details the different service types and which are required.


