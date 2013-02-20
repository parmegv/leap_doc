@nav_title = "User Guide"
@title = "Client User Guide"

Installation
==================

Note: currently, this does not completely work. See [testing](testing)

You can get the code from LEAP public git repository

    git clone git://leap.se/leap_client

Or from the github mirror

    git clone git://github.com/leapcode/leap_client.git

Once you have grabbed a copy of the sources, you can install it into your site-packages easily

    $ pyton setup.py install


Running
==================

This document dovers how to launch the LEAP Client.

Launching the client
--------------------

After a successful installation, there should be a launcher called leap-client somewhere in your path:

    $ leap-client

Debug mode
----------

If you are happy having lots of output in your terminal, you will like to know that you can run the client in debug mode:

    $ leap-client --debug

If you ask for it, you can also have all that debug info in a beautiful file ready to be attached to your bug reports:

    $ leap-client --debug --logfile /tmp/leap.log

> the following is broken since it will clutter your stdout with all the commands sent to the management interface. See bug #1232

If you want to increment the level of verbosity passed to openvpn, you can do:

    $ leap-client --openvpn-verbosity 4

Options
------------

To see all the available command line options:

    $ leap-client --help