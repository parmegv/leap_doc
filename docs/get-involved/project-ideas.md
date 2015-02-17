@title = "Project Ideas"
@summary = "Ideas for discrete, unclaimed development projects that would greatly benefit the LEAP ecosystem."

Project Ideas
============================================

Interested in helping with LEAP? Not sure where to dive in? This list of project ideas is here to help.

These are discrete projects that would really be a great benefit to the LEAP development effort, but are separate enough that you can dive right in without stepping on anyone's toes.

If you are interested [contact us on IRC or the mailing list](communication). We will put you in touch with the contact listed under each project.

If you have your own ideas for projects, we would love to hear about it!

Email
=======================================

Mac OS
---------------------------

### Apple Mail plugin

We have an extension for Thunderbird to autoconfigure for use with Bitmask. It would be great to do the same thing for Apple Mail. [Some tips to get started](http://blog.adamnash.com/2007/09/17/getting-ready-to-write-an-apple-mailapp-plug-in-for-mac-os-x/) and a "links to many existing Mail.app plugins"[http://www.tikouka.net/mailapp/]

* Priority: Low
* Contact: drebs
* Difficulty: Medium
* Skills: MacOS programming, Objective-C or Python (maybe other languages too?)

Windows
---------------------------

### Add Windows support for Soledad and all the different bundle components

We dropped Windows support because we couldn't keep up with all the platforms, Windows support should be re-added, which means making sure that the gpg modules, Soledad and all the other components are written in a proper multiplatform manner.

* Priority: High
* Contact: ivan, drebs
* Difficulty: Easy to Medium
* Skills: Windows programming, Python

### Microsoft Outlook plugin

We have an extension for Thunderbird to autoconfigure for use with Bitmask. It would be great to do the same thing for Outlook.

* Priority: Low
* Contact: drebs
* Difficulty: Medium
* Skills: Windows programming

Android
----------------------------------------------

### Migrate a subset of soledad to C/C++ with android in mind

Python is not well integrated in android, and other platforms. To have email support on them we need to have soledad. Some parts of soledad can be rewritten in C or C++ to add portability to them. Soledad uses u1db that has two implementations on python and C, we are currently using the Python implementation, we could switch to the C one.

* Priority: High
* Contact: meskio, kali, drebs
* Difficulty: Medium to Hard
* Skills: Python, C or C++

Other
----------------------------------------------

### Improve integration with Thunderbird

We have an extension for Thunderbird to autoconfigure for use with Bitmask. But it's lacking feedback to the user about bitmask specific things. For example it should have nice icons to notify if the email came encrypted, the validity of the signature, if the sent email was encrypted, ...

* Priority: High
* Contact: drebs
* Difficulty: low
* Skills: graphic design, Mozilla Add-Ons programming, JavaScript

### Support for offline storage private keys

Right now the private keys are stored in Soledad, encrypted with your password. For a bit more advanced users we should allow to store them in your computer/pen-drive and make easy to sync them between devices. We could maybe use a online syncing method and/or just by hand moving the keys from one device to another.

* Priority: Medium
* Contact: meskio
* Difficulty: Medium
* Skills: Python

### Support for OpenPGP smart cards

A really nice piece of hardware is OpenPGP smart cards. What would be needed is a way to save the generated key in the smart card instead of in Soledad (or both, should be configurable enough) and then migrate the regular OpenPGP workflow to support these change.

* Priority: Low
* Contact: meskio, drebs
* Difficulty: Medium
* Skills: Python

### Key management GUI

All the key management (discover of keys, update, ...) is done automagically. We could have a graphical interface, part of Bitmask, that displays what keys are we using and allows the user to add or remove keys.

* Priority: Meidum
* Contact: meskio
* Difficulty: Medium
* Skills: Python, Qt

VPN
=======================================

Linux
---------------------------

### Network manager support

The bitmask is split between front-end and backend. Will be nice to have a network manager plugin that communicates with the backend and controls the VPN.

* Priority: Low
* Contact: meskio, ivan
* Difficulty: Medium
* Skills: Python

Mac OS
-------------------------

### Proper privileged execution on Mac

We are currently running openvpn through cocoasudo to run OpenVPN with admin privs, we should not depend on a third party app and handle that ourselves. The proper way to do this is with [Service Management framework](https://developer.apple.com/library/mac/#samplecode/SMJobBless/Introduction/Intro.html).

* Priority: High
* Contact: ivan, kali
* Difficulty: Medium
* Skills: Mac programming

### Prevent DNS leakage on Mac OS

Currently, we block DNS leakage on the OpenVPN gateway. This works, but it would be better to do this on the client. The problem is there are a lot of weird edge cases that can lead to DNS leakage. See [dnsleaktest.com](http://www.dnsleaktest.com/) for more information.

* Priority: High
* Contact: ivan, kali
* Difficulty: Medium
* Skills: Mac programming

Windows
-------------------------------

### Proper privileged execution on Windows

Right now we are building OpenVPN with a manifest so that it's run as Administrator. Perhaps it would be better to handle this with User Account Control.

* Priority: High
* Contact: ivan, kali
* Difficulty: Medium
* Skills: Windows programming

### Prevent DNS leakage on Windows

Currently, we block DNS leakage on the OpenVPN gateway. This works, but it would be better to do this on the client. The problem is there are a lot of weird edge cases that can lead to DNS leakage. See [dnsleaktest.com](http://www.dnsleaktest.com/) for more information.

* Priority: High
* Contact: ivan, kali
* Difficulty: Medium
* Skills: Windows programming

### Support Windows 64bits

We have support for Windows 32bits, 64bits seems to be able to use that, except for the TAP driver for OpenVPN. So this task is either really easy because it's a matter of calling the installer in a certain way or really hard because it involves low level driver handling or something like that.

* Priority: High
* Contact: ivan
* Difficulty: Either hard or really easy.
* Skills: Windows programming

Android
----------------------------------------------

### Dynamic OpenVPN configuration

Right now there is one gateway, different protocols (tcp/udp) and ports => Bitmask tests with every protocol/port available until success. We would like different gateways, ... " ... => Bitmask tests with every gateway defined by the provider until success, in proximity order.

* Priority: Medium
* Difficulty: easy
* Contact: parmegv
* Skills: Android programming

### Ensure OpenVPN fails closed

Right now Bitmask blocks traffic until the vpn is up, but if Bitmask dies traffic flows. We would like set up the blocking vpn interface before dying.

* Priority: Medium
* Difficulty: easy
* Contact: parmegv
* Skills: Android programming

### Beautify UI and code tests

Our test coverage is so low, a lot of UI flows are not tested. And for sure, UI can be improved. We would like more than 80% of test coverage, corresponding refactors and beautiful UI.

* Priority: Low
* Difficulty: medium-hard => TDD, refactors and tests corner cases.
* Contact: parmegv
* Skills: Android programming

Installer and Build Process
=======================================

Linux
----------------------------------------------

### Package application for non-Debian linux flavors

The Bitmask client application is entirely ported to Debian, with every dependency library now submitted to unstable. However, many of these packages are not in other flavors of linux, including RedHat/Fedora, SUSE, Arch, Gentoo.

* Priority: Low
* Contact: kali, micah, ivan
* Difficulty: Medium
* Skills: Linux packaging

Windows
-------------------------------

### Code signing on Windows

The bundle needs to be a proper signed application in order to make it safer and more usable when we need administrative privileges to run things like OpenVPN.

* Priority: High
* Contact: ivan
* Difficulty: Easy to medium
* Skills: Windows programming

### Create proper Windows installer for the bundle

We are aiming to distributing bundles with everything needed in them, but an amount of users will want a proper Windows installer and we should provide one.

* Priority: Medium
* Contact: ivan
* Difficulty: Medium
* Skills: Windows programming

### Document how to build everything with Visual Studio Express

All the python modules tend to be built with migw32. The current Windows bundle is completely built with migw32 for this reason. Proper Windows support means using Visual Studio (and in our case, the Express edition, unless the proper licenses are bought).

* Priority: Low
* Contact: ivan
* Difficuty: Medium to Hard
* Skills: Windows programming

Other
----------------------------------------------

### Reproducible builds with Gitian for bundles

We rely on a group of binary components in our bundles, these include libraries like boost, Qt, PySide, pycryptopp among many others. All these should be built in a reproducible way in order to be able to sign the bundles from many points without the need to actually having to send the bundle from the main place it gets built to the rest of the signers. This will also allow a better integration with our automatic updates infrastructure.

* Priority: Medium
* Contact: ivan
* Difficulty: Medium to hard

### Lightweight network installer

The bundles are big. It would be great if we could reduce its size, but that's not always possible when you are providing so many different things in one application. One way to work around this would be to have a really tiny application that runs [TUF](http://theupdateframework.com/), has the proper certificates and has a tiny lightweight UI so that the user can install the bundle's packages one by one and even pick parts that the user might not want. Just want to run Email? Then there's no need to download OpenVPN and all the chat and file sync code.

* Priority: Low
* Contact: meskio
* Difficulty: Medium to hard
* Skills: C/C++, Python

### Package application for BSD

The Bitmask client application is entirely ported to Debian, with every dependency library now submitted to unstable. However, many of these packages are not in *BSD.

* Priority: Low
* Contact: ivan, kali
* Difficulty: Medium
* Skills: BSD packaging

LEAP Platform
===========================

Soledad
---------------------------

### Add support for quota

Soledad server only handles authentication and basic interaction for sync, it would be good to have a way to limit the quota each user has to use and enforce it through the server.

* Priority: Medium
* Contact: drebs
* Difficulty: Medium to hard
* Skills: Python

### Add support for easier soledad server deployment

Currently Soledad relies on a fairly complex CouchDB setup. It can be deployed with just one CouchDB instance, but may be if you are just using one instance you might be good enough with SQLite or other easy to setup storage methods. The same applies to authentication, may be you want a handful of users to be able to use your Soledad sever, in which case something like certificate client authentication might be enough. So it would be good to support these non-scalable options for deploying a Soledad server.

* Priority: Low
* Contact: drebs
* Difficulty: Medium
* Skills: Python

### A soledad management tool

Bootstrapping Soledad and being able to sync with it is not a necessarily easy task, you need to take care of auth and other values like server, port, user id. Having an easy to use command line interface application that can interact with Soledad would ease testing both on the client as on the server.

* Priority: Low
* Contact: drebs, meskio
* Difficulty: Easy to medium
* SKills: Python

### Federated Soledad

Currently, each user's Soledad database is their own and no one else ever has access. It would be mighty useful to allow two or more users to share a Solidad database.

* Priority: Low
* Contact: drebs, elijah
* Difficult: Hard
* Skills: Python

DNS
--------------------------------

### Add DNSSEC entries to DNS zone file

We should add commands to the leap command line tool to make it easy to generate KSK and ZSK, and sign DNS entries.

* Priority: Low
* Contact: elijah, micah, varac
* Difficulty: Easy
* Skills: Ruby

### Add DANE entries to DNS zone file

Every node one or more server certificates. We should publish these using DANE.

* Priority: Low
* Contact: elijah, micah, varac
* Difficulty: Easy

### Add DKIM entries to DNS zone file

We need to generate and publish [DKIM](https://en.wikipedia.org/wiki/DKIM) keys.

* Priority: Medium
* Contact: elijah, micah, varac
* Difficulty: Easy

OpenVPN
-----------------------------------

### OpenVPN with ECC PFS support

Currently, OpenVPN gets configured to use a non-ECC DH cipher with perfect forward secrecy, but it would be nice to get it working with an Elliptical Curve Cipher. This greatly reduces the CPU load of the OpenVPN gateway.

* Priority: Low
* Contact: elijah, varac
* Difficulty: Medium
* Skills: OpenVPN, X.509

Email
--------------------------

### Mailing list support

Adapt the PSELS mailing list for use with the LEAP platform. PSELS uses OpenPGP in a novel way to achieve proxy re-encryption, allowing for a mailing list in which the server does not ever have access to messages in cleartext, but subscribers don't need to encrypt each message to the public key of all subscribers. For more information, read the [paper](http://www.ncsa.illinois.edu/people/hkhurana/ICICS.pdf).

* Priority: Low
* Contact: elijah
* Difficulty: Extremely hard
* Skills: Cryptography, Python

SSL
-----------------------------------

### Integrate smtp-fp on leap_cli

[smtp-fp](https://git.autistici.org/ale/smtp-fp) is a decentralized way of authenticate SSL keys. Will be nice to have it integrated in the leap platform.

* Priority: Medium
* Contact: elijah, micah
* Difficulty: Easy
* Skills: Ruby, maybe some Go

### Add support for letsencrypt.org

SSL key validation can be authomatic with [letsencrypt.org](https://letsencrypt.org/).

* Priority: Low
* Contact: elijah, micah
* Difficulty: Easy to Medium
* Skills: Ruby


LEAP Webapp
============================

### Add support for bitcoin payments to the billing module

The webapp has a payment infrastructure setup (Braintree), but it only supports credit card and bank wire payments. The webapp should be extended to also accept payments from bitcoin.

* Priority: Low
* Contact: elijah
* Difficulty: Easy

### Add support for newsletter

Sometimes simple push notifications aren't enough, you may want to mail a newsletter to your users or more descriptive notifications, it should be possible for an administrator of a provider to use the webapp to quickly send mail to all its users.

* Priority: Medium
* Contact: elijah
* Difficulty: Easy

### Add support for quota

Description: Once the Soledad server quota enforcement code is in place, it would be good to have the ability to configure the quota for a user and check the user's quota via the webapp.

* Dependency: Soledad server quota enforcement.
* Contact: elijah
* Difficulty: Easy
* Skills: Ruby


Miscellaneous
=======================================

### Improve test coverage

We have some basic unit-testing coverage on the code. Big parts of the code are not covered by them. Adding test will improve the code as well forcing to redesign some parts that are not easy to test.

* Priority: Low
* Contact: meskio
* Difficulty: Easy
* Skills: Python

### Nicer UI

Make nicer the graphical interface of bitmask. Maybe using html or Qt5.

* Priority: Low
* Contact: ivan, elijah, kali
* Difficulty: Medium
* Skills: design, Python, qt, html

### Ncruses UI

The bitmask is split between front-end and backend. The front-end right now is a Qt program. For the command line nerds will be nice to have a ncurses front-end.

* Priority: Low
* Contact: meskio, ivan
* Difficulty: Medium to Hard
* Skills: Python

### General QA

One thing that we really need is a team of people that is constantly updating their versions of the code and testing the new additions.

* Priority: Low
* Contact: mcnair, elijah, ivan
* Difficulty: Easy to medium, depending on the QA team that is managed.
* Skills: basic knowledge of Git and Python

### Translations

Do you speak a language that's not English? Great! We can use your help! We are always looking for translators for every language possible.

* Priority: Medium
* Contact: ivan, kali
* Difficulty: Easy

### Device blessing

Add the option to require a one-time code in order to allow an additional device to be synchronized with your account.

* Priority: Low
* Contact: elijah
* Difficulty: Hard
* Skills: Python

### Push notifications from the server

There are situations where the service provider you are using through the bitmask client might want to notify some event to all its users. May be some downtime, or any other problems or situations. There should be an easy way to push such notifications to the client.

* Priority: Medium
* Contact: elijah
* Difficulty: Easy to medium
* Skills: Python

### Quick wipe of all data

Some users might be in situations where being caught with software like OpenVPN is illegal or basically just problematic. There should be a quick way to wipe the existence of the whole bundle and your identity from provider.

* Priority: Low
* Contact: kali, ivan, elijah
* Difficulty: Medium to hard
* Skills: Python

New Services
----------------------------------

Our primary focus are email and VPN services, we will not be prioritizing any new service for sometime. However, if you are hell bent on creating a new LEAP service please contact us and we will help as much as our capacity allows us to.

### Password keeper

There are multiple password keepers that exist today, but they don't necessarily have a way to sync your passwords from device to device. Building a Soledad backed password keeper would solve all these problems implicitly, it's only a matter of UI and random password generation.

* Priority: Low
* Contact: drebs, meskio, elijah
* Difficulty: Easy to medium.
* Skills: Python

### Notepad app

This idea is basically a simple note pad application that saves all its notes as Soledad documents and syncs them securely against a Soledad server.

* Priority: Low
* Contact: meskio, kali, drebs
* Difficulty: Easy to medium
* Skills: Python
