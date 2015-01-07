@title = 'LEAP Platform quick start tutorial'
@nav_title = 'Quick Start'
@summary = 'Getting leap platform up, the quick way'


Testing Leap platform with Vagrant
==================================

There are two ways how you can setup leap platform 
using vagrant.

Use the leap_cli vagrant integration
------------------------------------

Install leap_cli and leap_platform on your host, 
configure a provider from scratch and use the 
`leap local` commands to manage your vagrant node(s).

See https://leap.se/en/docs/platform/development how to use 
the leap_cli vagrant integration and
https://leap.se/en/docs/platform/tutorials/single-node how 
to setup a single node mail server.


Using the Vagrantfile provided by Leap Platform
-----------------------------------------------

This is by far the easiest way. 
It will install a single node mail Server in the default
configuration with one single command.

Clone the 0.6.1 platform branch with

    git clone -b 0.6.1 https://github.com/leapcode/leap_platform.git

Start the vagrant box with 

    cd leap_platform
    vagrant up

Follow the instructions how to configure your `/etc/hosts`
in order to use the provider!

You can login via ssh with the systemuser `vagrant` and the same password.

There are 2 users preconfigured: 

. `testuser`  with pw `hallo123`
. `testadmin` with pw `hallo123`

Testing your provider
=====================

Using the bitmask client
------------------------

Download the provider ca:

    wget --no-check-certificate https://example.org/ca.crt -O /tmp/ca.crt

Start bitmask:

    bitmask --ca-cert-file /tmp/ca.crt



Recieving Mail
--------------

Use i.e. swaks to send a testmail

    swaks -f noone@example.org -t testuser@example.org -s example.org

and use your favorite mail client to examine your inbox.
You can also use [offlineimap](http://offlineimap.org/) to fetch mails: 

     offlineimap -c vagrant/.offlineimaprc.example.org

WARNING: Use offlineimap *only* for testing/debugging, 
because it will save the mails *decrypted* locally to 
your disk !

