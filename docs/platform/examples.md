@title = 'Examples'
@summary = 'Example provider setups using the LEAP platform.'

A minimal three node OpenVPN provider
=======================================

Our goal
------------------

We are going to create a minimal LEAP provider offering OpenVPN service. This basic setup can be expanded by adding more OpenVPN nodes to increase capacity, or more webapp and couchdb nodes to increase availability (performance wise, a single couchdb and a single webapp are more than enough for most usage, since they are only lightly used, but you might want redundancy).

Our goal is something like this:

    leap list
           NODES   SERVICES           TAGS
            clam   couchdb
        elephant   webapp
           snail   openvpn

NOTE: You won't be able to run those `leap list` commands yet, not until we actually create the node configurations.

Create configuration files
--------------------------------

Create the provider directory:

    leap new bitmask --domain bitmask.net --name bitmask --contacts root@bitmask.net

Add you ssh key:

    leap add-user --self

Add some nodes:

    leap node add clam ip_address:176.53.69.22 services:couchdb
    leap node add elephant ip_address:176.53.69.13 services:webapp
    leap node add snail ip_address:176.53.69.14 \
      openvpn.gateway_address:176.53.69.15 services:openvpn

NOTE: openvpn gateways must be assigned two IP addresses, one for the host itself and one for the openvpn gateway. We do this to prevent incoming and outgoing VPN traffic on the same IP. Without this, the client might send some traffic to other VPN users in the clear, bypassing the VPN.

Now that you have the nodes configured, you should create the DNS entries for these nodes.

Set up your DNS with these hostnames:

    leap list --print ip_address,domain.full,dns.aliases
        clam  176.53.69.22, clam.bitmask.net, null
    elephant  176.53.69.13, elephant.bitmask.net, api.bitmask.net
       snail  176.53.69.14, snail.bitmask.net, null

Alternately, you can adapt this zone file snippet:

    leap compile zone

Create certificates
------------------------------------

Create two certificate authorities, one for server certs and one for client certs:

    leap cert ca

Create a temporary cert for your main domain (you should replace with a real commercial cert at some point)

    leap cert csr

Create the Diffie-Hellman parameters file, needed for forward secret OpenVPN ciphers:

    leap cert dh

Create server certificates for all the nodes you have added:

    leap cert update

NOTE: the file `files/ca/ca.key` is extremely sensitive and must be carefully protected. The other key files are much less sensitive and can simply be regenerated if needed.

Deploy to nodes
------------------------

Initialize all nodes (only needs to be done once for each node):

    leap node init

Deploy to all nodes:

    leap deploy

Those two commands create pretty busy output, so it may be more clear to initial and deploy each node one by one:

    leap node init clam
    leap deploy clam
    leap node init elephant
    leap deploy elephant
    leap node init snail
    leap deploy snail

Testing
--------------------------

Automated testing is in the works, but for now you manually test to see if the OpenVPN gateways and the webapp are working like so.

OpenVPN:

    leap test init
    sudo openvpn test/openvpn/unlimited.ovpn

Webapp:

* run `leap list --print ip_address webapp` to remind yourself the ip address(es) of the webapp.
* edit your local `/etc/hosts` to add entries like `176.53.69.13 example.org`, for whatever domain is appropriate in your case.
* open your browser to `https://example.org`

