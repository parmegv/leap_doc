@nav_title = "Bundle QA"
@title = "Guidelines for bundle QA"

Recommended setup
-----------------

VirtualBox (or similar) with virtual machines installed for supported OSs

For each system that you are going to test, you should do:

- Install the VM
- Restart the VM and check that the process is finished.
- Turn it off and make a snapshot named 'fresh install' or similar.

The OS should be installed with the default settings and no extra packages. However, you can choose your language, username, timezone, etc


Test process
------------

- roll back the virtual machine to its *fresh install* state, to make sure that you're testing against a reproducible environment.
- download the bundle, verify signature (if apply), extract and run the app
- test the application (see next section)


Tests to do
-----------

- **check if the version number is the same as the current bundle version**
    - 'Help->About Bitmask'
    - `./bitmask --version`
- **correct installation of files to 'better protect privacy'**
    - `/etc/leap/resolv-update`
    - `/usr/share/polkit-1/actions/net.openvpn.gui.leap.policy`

    You should check that they get copied when the user says 'yes' and they don't get copied if the user says 'no'.
- **installation of tun/tap in Windows and MAC**
    TODO: explain more here

- **account creation**

    Recommended username template: test_bundleversion_os_arch, that way you avoid conflicts between test iterations.
    e.g.: 'test_036_debian7_64', 'test_036_win7_32', etc

    If you need to create extra users in order to test a bug or whatever, you can use 'test_036_ubuntu1204_32a', 'test_036_ubuntu1204_32b', etc

    In case of being a lot of users testing a version you may want to use your username instead of test, e.g.: 'johndoe_036_ubuntu1204_32'.
- **eip connection**

    You can check if the vpn is working entering to the site: http://wtfismyip.com

    or using the console:
    `shell> wget -qO- wtfismyip.com/json`
- **Soledad key generation**
- **Thunderbird configuration manually and using add-on**
- **Send and receive mail**

    You need to test communication between inside and outside users, e.g.: someuser@bitmask.net and otheruser@gmail.com

    A good thing to do is to subscribe to a mailing list that have a lot of activity.

- **Check if the account data is correctly synced.**

    After the account creation, have everything working and the app closed:
    - remove the configuration files created by the app (`~/.config/leap` in linux)
    - log in with your recently created credentials and check that everything is working and your mails are there too.


Problems report
---------------

You should to create an issue with the followinw information:

- OS, version, architecture, desktop environment (if relevant).
- bitmask.log file located in the root folder of the uncompressed bundle
- steps to reproduce

If you find a problem, try to reproduce and take note of the steps needed to get the same error.

Also, in some cases, a failure appears but if you run again is not there anymore (e.g.: some initialization issue), please report that too.

For more details look at [Reporting bugs](client/testers-howto) section.


Utils
-----

Download, extract and run helper script for linux:

    shell> ./download-extract-run-bitmask.sh

Script contents:

    #!/bin/bash
    HOST="https://dl.bitmask.net/client/linux/"
    VERSION="0.3.7"
    # FOLDER="Bitmask-linux32-${VERSION}"
    FOLDER="Bitmask-linux64-${VERSION}"
    FILE="${FOLDER}.tar.bz2"

    wget ${HOST}${FILE} && tar xjf ${FILE} && cd ${FOLDER} && ./bitmask
