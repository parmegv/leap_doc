@title = 'Bitmask known issues'
@nav_title = 'Known issues'
@summary = 'Known issues in Bitmask.'
@toc = true

Here you can find documentation about known issues and potential work-arounds in the current Leap Platform release.

0.5
===

In this release the following issues are known, work-arounds are noted when available.

General Issues
--------------

- If you have received a big ammount of mails (tested with more than 400), you may experience that Thunderbird won't respond.

That problem does not happen if you have the client open and Thunderbird loading mails while are reaching your inbox.

- You may get the error: "Unable to connect: Problem with provider" in situations when the problem is the network instead of the provider. (see: https://leap.se/code/issues/4023)

- Opening the same account from more than one box at the same time will possibly break your account.

- Managing a huge ammount of mails (e.g.: moving mails to a folder) will block the UI (see https://leap.se/code/issues/4837)

Special Environments
--------------------

- You may experience problems related to an Unavailable polkit agent in gnome3. (see https://leap.se/code/issues/4144)

