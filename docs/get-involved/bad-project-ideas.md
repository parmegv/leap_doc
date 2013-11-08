### Import current GPG Key to be used with leap mail.

* Contact: drebs, chiiph
* Difficulty: Medium
* Description: Current GPG users have their key already, and they may not need or want to migrate to a new key with their bitmask user, so it would be great if instead of generating a new key, the client could ask for an alternative key to be imported. Another option would be to have the hability to have multiple keys for a user and have the client be configurable enough so that an advanced user can choose which to use.

### Certificate perspectives through Tor or other methods.

* Contact: chiiph
* Difficulty: Easy to medium
* Description: Properly trusting a certificate is not the easiest thing to do, if you are a target of a Man in the Middle in your network, chances are you are going to be in trouble. One way to solve this problem is to have a better network perspective. This can be accomplished by launching Tor, building 3 circuits that exit from different parts of the world, and downloading the certificate from each point and then comparing the outcomes of it.

### Contact list replacement for Android based on Soledad

* Dependencies: Soledad port on Android
* Contact: drebs, chiiph
* Difficulty: Easy to medium.
* Description: Having a client encrypted sync'ed solution for all your contacts in your devices is something that can be easily solved by using Soledad for storage and implementing a custom SyncAdapter for contacts and calendar.
* Resources: https://developer.android.com/training/sync-adapters/creating-sync-adapter.html

### Support for KVM, OpenVZ, LXC

* Contact: elijah, micah, varac
* Difficulty: -

### Add OAuth2 auth to soledad server or other methods.

* Contact: drebs, chiiph
* Difficulty: Easy to medium
* Description: One of the most used method for authentication is OAuth, currently Soledad server only supports our own token authentication methods but that won't be necessarily the case for other Soledad adopters, so it would be great to use our pluggable auth design in Soledad server to add as many authentication methods as possible, such as OAuth.

### Tor support

* Contact: chiiph, drebs
* Difficulty: Easy to medium
* Description: It would be great to be able to access a Soledad server through Tor, the idea is to add the necessary code in Soledad for this to be possible, and later on add that as a configuration option for the bitmask client.

### Encrypted filesystem based on Soledad (FUSE)

* Contact: chiiph, elijah
* Difficulty: Medium to hard.
* Description: There are certain issues with building a fully distributed secure file system solution, all of which can be solved with Soledad. One possible approach to this problem would be to use something like Tahoe-LAFS and use Soledad as the collector of your caps. Another approach could be using Soledad directly and handling problems like chunking by hand directly in this app.

### Calendar app

* Contact: chiiph, drebs
* Difficulty: Easy to medium
* Description: This task would involve basically building a UI for a calendar application that is Soledad backed, which would be easily sync'ed among all the user's devices.

### Add leap token auth to Vines XMPP server
* Contact: elijah
* Difficulty: -
* Skills: Ruby

### Add MUC to Vines XMPP server
* Contact: elijah
* Difficulty: -
* Description: -

### Port Soledad to Android
* Contact: drebs, chiiph
* Difficulty: Medium to hard
* Description: Soledad is currently built on top of U1DB's reference implementation which is in Python. It also uses OpenSSL and pycryptopp for the cryptography bits. So the possibilities for porting Soledad to Android are: Implement it in pure C and use cryptopp (since it's what pycryptopp is using underneath), do an pure Java implementation or try to run the Python code we are already using. It would be reasonable to not have the most fast implementation at first if running our Python code is possible and would shorten the development times.

### Port Keymanager to Android
* Dependencies: Soledad for Android
* Contact: drebs, chiiph
* Difficulty: Medium
* Description: The way we try to solve the key distribution problem is by having a NickServer and handling key logic through what we call the KeyManager. Currently, as most of our components, it's implemented in Python, so the same ideas apply here as for the Soledad port.
