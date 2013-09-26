Where do i find the time a server was last deployed ?
-----------------------------------------------------

The puppet state file on the node indicates the last puppetrun:

    ls -la /var/lib/puppet/state/state.yaml

What resources are touched by puppet/leap_platform (services/packages/files etc.) ?
-----------------------------------------------------------------------------------

Log into your server and issue:

    grep -v '!ruby/sym' /var/lib/puppet/state/state.yaml | sed 's/\"//' | sort

How do i change the domain of my provider ?
-------------------------------------------

* First of all, you need to have access to the nameserver config of your new domain.
* Update domain in provider.json
* remove all ca and cert files: `rm files/cert/* files/ca/*`
* create ca, csr and certs : `leap cert ca; leap cert csr; leap cert dh; leap cert update`
* deploy
