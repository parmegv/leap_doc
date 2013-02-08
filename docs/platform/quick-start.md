Installation
--------------------------------

Install prerequisites:

    sudo apt-get install git ruby ruby-dev rsync openssh-client openssl

This tutorial should work with ruby1.8, but has only been tested using ruby1.9.

Install the `leap` command:

    sudo gem install leap_cli

Alternately, you can install `leap` from source:

    sudo apt-get install rake
    git clone git://leap.se/leap_cli.git
    cd leap_cli
    rake build

Install as unprivileged user:

    rake install
    # watch out for the directory leap is installed to, then i.e.
    ln -s ~/.gem/ruby/1.9.1/bin/leap /usr/local/bin/leap

Install as root user (recommended):

    sudo rake install

With both methods, you can use now /usr/local/bin/leap, 
which in most cases will be in your $PATH.


Create a provider instance
---------------------------------------

A provider instance is a directory tree, usually stored in git, that contains everything you need to manage an infrastructure for a service provider. In this case, we create one for rewire.net and call the instance directory 'rewire'.

    mkdir -p ~/leap/rewire

Now, we will initialize this directory to make it a provider instance. Your provider instance will need to know where it can find local copy of the git repository leap_platform, which holds the puppet recipes you will need to manage your servers. Typically, you will not need to modify leap_platform.

    cd ~/leap/rewire
    leap init --domain rewire.net --name rewire --platform ../leap_platform .

In this case, `../leap_platform` will be created if it does not exist.

You may want to poke around and see what is in the files we just created. For example:

    cat provider.json

If you are familiar with git, you might want to have the your provider directory under 
version control:

    git init
    git add .
    git commit -m "initial commit"

Now add yourself as a privileged sysadmin who will have access to deploy to servers:

    leap add-user --self

NOTE: in most cases, `leap` must be run from within a provider instance directory tree (e.g. ~/leap/rewire).

Now generate required X509 certificates and keys:

    leap cert ca
    leap cert csr

To see details about the keys and certs that the prior two commands created, you can use `leap inspect` like so:

    leap inspect files/ca/ca.crt


Edit configuration files and provide some information for new provider, 
these are the files you need to touch:

* common.json


Create nodes

A "node" is a server that is part of your infrastructure. Every node can have one or more services associated with it. Some nodes are "local" and used only for testing. These local nodes exist only as virtual machines on your computer and cannot be accessed from outside (see `leap help local` for more information).

Create a local node, with the service "webapp":

    leap node add --local web1 services:webapp

This created a node configuration file, but it did not create the virtual machine. In order to test our node "web1", we need to first spin up a virtual machine. The next command will probably take a very long time, because it will need to download an image to create the virtual machine (about 700mb).

    leap local start

Now that the virtual machine for web1 is running, you need to initialize it and then deploy the recipes to it. You only need to initialize a node once, but there is no harm in doing it multiple times. These commands will take a while to run the first time, as it needs to update the package cache on the new virtual machine.

    leap node init web1
    leap cert update
    leap deploy web1

That is it, you should now have your first running node. However, the LEAP web application requires a database to run, so let's add a "couchdb" node:

    leap node add --local db1 services:couchdb
    leap local start
    leap node init db1
    leap deploy db1

What is going on here?
--------------------------------------------

(explain about hiera files, general deploy process)

Additional commands
-------------------------------------------

Here are a few useful commands you can run on your new local nodes:

* `leap ssh web1` -- SSH into node web1 (requires `leap node init web1` first).
* `leap list` -- list all nodes.
* `leap list --print ip_address` -- list a particular attribute of all nodes.
* `leap local reset web1` -- return web1 to a pristine state.
* `leap local stop` -- stop all local virtual machines.
* `leap local status` -- get the running state of all the local virtual machines.

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

todo: link to another document that details the different service types and which are required.

