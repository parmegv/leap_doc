@title = 'Hard problems in secure communication'
@nav_title = 'Hard problems'
@summary = "How LEAP addresses the difficult problems in secure communication"

## The big seven

If you take a survey of interesting initiatives to create more secure communication, a pattern starts to emerge: it seems that any serious attempt to build a system for secure message communication eventually comes up against the following list of seven hard problems.

1. **Authenticity problem**: Public key validation is very difficult for users to manage, but without it you cannot have confidentiality.
2. **Meta-data problem**: Existing protocols are vulnerable to meta-data analysis, even though meta-data is often much more sensitive than content.
3. **Asynchronous problem**: For encrypted communication, you must currently choose between forward secrecy or the ability to communicate asynchronously.
4. **Group problem**: In practice, people work in groups, but public key cryptography doesn't.
5. **Resource problem**: There are no open protocols to allow users to securely share a resource.
6. **Availability problem**: People want to smoothly switch devices, and restore their data if they lose a device, but this is very difficult to do securely.
7. **Update problem**: Almost universally, software updates are done in ways that invite attacks and device compromises.

These problems appear to be present regardless of which architectural approach you take (centralized authority, distributed peer-to-peer, or federated servers).

It is possible to safely ignore many of these problems if you don't particularly care about usability or matching the features that users have grown accustomed to with contemporary methods of online communication. But if you do care about usability and features, then you are stuck with finding solutions to these problems.

## Our solutions

In our work, LEAP has tried to directly face down these seven problems. In some cases, we have come up with solid solutions. In other cases, we are moving forward with temporary stop-gap measures and investigating long term solutions. In two cases, we have no current plan for addressing the problems.

### Authenticity problem

The problem:

> Public key validation is very difficult for users to manage, but without it you cannot have confidentiality.

If proper key validation is a precondition for secure communication, but it is too difficult for most users, what hope do we have? We have developed a unique federated system called [Nicknym](/nicknym) that automatically discovers and validates public keys allowing the user to take advantage of public key cryptography without knowing anything about keys or signatures.

The standard protocol that exists today to solve this problem is [DANE](https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities). DANE might be the better option in the long run, but currently DANE is complex to set up, complex for clients to consume, leaks association information to a network observer, and relies on trusting the DNS root zone and TLD zones.

### Meta-data problem

The problem:

> Existing protocols are vulnerable to meta-data analysis, even though meta-data is often much more sensitive than content.

As a short term measure, we are integrating opportunistic encrypted transport (TLS) for email and chat messages when relayed among servers. There are two important aspects to this:

* Relaying servers need a solid way to discover and validate the keys of one another. For this, we are initially using DNSSEC/DANE.
* An attacker must not be able to downgrade the encrypted transport back to cleartext. For this, we are modifying software to ensure that encrypted transport cannot later be downgraded.

This approach is potentially effective against external network observers, but does not protect the meta-data from the service providers themselves. Also, it does not, by itself, protect against more advanced attacks involving timing and traffic analysis.

In the long term, we plan to adopt one of several different schemes for securely routing meta-data. These include:

* Auto-alias-pairs: Each party auto-negotiates aliases for communicating with each other. Behind the scenes, the client then invisibly uses these aliases for subsequent communication. The advantage is that this is backward compatible with existing routing. The disadvantage is that the user's server stores a list of their aliases. As an improvement, you could add the possibility of a third party service to maintain the alias map.
* Onion-routing-headers: A message from user A to user B is encoded so that the "to" routing information only contains the name of B's server. When B's server receives the message, it unwraps (unencrypts) a supplementary header that contains the actual user "B". Like aliases, this provides no benefit if both users are on the same server. As an improvement, the message could be routed through intermediary servers.
* Third-party-dropbox: To exchange messages, user A and user B negotiate a unique "dropbox" URL for depositing messages, potentially using a third party. To send a message, user A would post the message to the "dropbox". To receive a message, user B would regularly polls this URL to see if there are new messages.
* Mixmaster-with-signatures: Messages are bounced through a mixmaster-like set of anonymization relays and then finally delivered to the recipient's server. The user's client only displays the message if it is encrypted, has a valid signature, and the user has previously added the sender to a 'allow list' (perhaps automatically generated from the list of validated public keys).

For a great discussion comparing mix networks and onion routing, see [Tom Ritter's blog post on the topic](https://ritter.vg/blog-mix_and_onion_networks.html).

### Asynchronous problem

The problem:

> For encrypted communication, you must currently choose between forward secrecy or the ability to communicate asynchronously.

With the pace of growth in digital storage and decryption, forward secrecy is increasingly important. Otherwise, any encrypted communication you engage in today is likely to become cleartext communication in the near future.

In the example of email and chat, we have OpenPGP with email and OTR with chat: the former provides asynchronous capabilities, and the latter forward secrecy, but neither one supports both abilities. We need both better security for email and the ability to send/receive offline chat messages.

In the short term, we are layering forward secret transport for email and chat relay on top of traditional object encryption (OpenPGP). This approach is identical to our stop-gap approach for the meta-data problem, with the one addition that relaying servers need the ability to not simply negotiate TLS transport, but to also negotiate forward secret ciphers and to prevent a cipher downgrade.

This approach is potentially effective against external network observers, but does not achieve forward secrecy from the service providers themselves.

In the long term, we plan to work with other groups to create new encryption protocol standards that can be both asynchronous and forward secret:

* [Forward Secrecy Extensions for OpenPGP](http://tools.ietf.org/html/draft-brown-pgp-pfs-03)
* [Triple elliptical curve Diffie-Hellman handshake](https://whispersystems.org/blog/simplifying-otr-deniability/)

### Group problem

The problem:

> In practice, people work in groups, but public key cryptography doesn't.

We have a lot of ideas, but we don't have any solutions yet to fix this. Essentially, the question is how to use existing public key primitives to create strong cryptographic groups, where membership and permissions are based on keys and not arbitrary server-maintained access control lists.

Most of the interesting work in this area has been done by companies working on secure file backup/sync/sharing, such as Wuala and Spideroak. Unfortunately, there are not yet any good open protocols or free software packages that can handle group cryptography.

At the moment, probably the best approach is the simple approach: a protocol where the client encrypts each message to each recipient individually, and has some mechanism to verify the transcript to ensure that all parties received the same messages.

There is some free software work on some of interesting building blocks that could be useful in building group cryptography. For example:

* [Proxy re-encryption](https://en.wikipedia.org/wiki/Proxy_re-encryption): This allows the server to re-encrypt to new recipients without gaining access to the cleartext. The [SELS mailing list manager](http://sels.ncsa.illinois.edu/) uses OpenPGP to implement a [clever scheme for proxy re-encryption](http://spar.isi.jhu.edu/~mgreen/proxy.pdf).
* [Ring signatures](https://en.wikipedia.org/wiki/Ring_signature): This allows any member of a group to sign, withing anyone knowing which member.

### Resource problem

The problem:

> There are no open protocols to allow users to securely share a resource.

For example, when using secure chat or secure federated social networking, you need some way to link to external media, such as an image, video or file, that has the same security guarantees as the message itself. Embedding this type of resource in the messages themselves is prohibitively inefficient.

We don't have a proposal for how to address this problem. There are a lot of great initiatives working under the banner of read-write-web, but these do not take encryption into account. In many ways, solutions to the resource problem are dependent on solutions to the the group problem.

As with the group problem, most of the progress in this area has been by people working on encrypted file sync (e.g. strategies like Lazy Revocation and Key Regression).

### Availability problem

The problem:

> People want to smoothly switch devices, and restore their data if they lose a device, but this very difficult to do securely.

Users today demand the ability to access their data on multiple devices and to have piece of mind that their data will not be lost forever if they lose a device. In the free software world, only Firefox has addressed this problem adequately and in a secure way (with Firefox Sync).

At LEAP, we have worked to solve the availability problem with a system we call [Soledad](/soledad) (for Synchronization of Locally Encrypted Documents Among Devices). Soledad gives the client application an encrypted, synchronized, searchable document database. All data is client encrypted, both when it is stored on the local device and synced with the cloud. As far as we know, there is nothing else like it, either in the free software or commercial world.

Soledad tries to solve the problem of general data availability, but other initiatives have tried to tackle the more narrow problem of availability of private keys and discovered public keys. These initiatives include:

* Ben Laurie's [proposed protocol for storing secrets in the cloud](http://www.links.org/files/nigori/nigori-protocol-01.html)
* Experimental [code for similar cloud storage of keys](https://github.com/mettle/nilcat)
* Phillip Hallam-Baker's [thoughts along similar lines](http://tools.ietf.org/html/draft-hallambaker-prismproof-key-00)

### Update problem

The problem:

> Almost universally, software updates are done in ways that invite attacks and device compromises.

The sad state of update security is especially troublesome because update attacks can now be purchased off the shelf by repressive regimes. The problem of software update is particular bad on desktop platforms. In the case of mobile and HTML5 apps, the vulnerabilities are not as dire, but the issues are also harder to fix.

To address the update problem, LEAP is adopting a unique update system called Thandy from the Tor project. Thandy is complex to manage, but is very effective at preventing known update attacks.

Thandy, and the related [TUF](https://updateframework.com), are designed to address the many [security vulnerabilities in existing software update systems](https://updateframework.com/projects/project/wiki/Docs/Security). In one example, other update systems suffer from an inability of the client to confirm that they have the most up-to-date copy, thus opening a huge vulnerability where the attacker simply waits for a security upgrade, prevents the upgrade, and launches an attack exploiting the vulnerability that should have just been fixed. Thandy/TUF provides a unique mechanism for distributing and verifying updates so that no client device will install the wrong update or miss an update without knowing it.

Related to the update problem is the backdoor problem: how do you know that an update does not have a backdoor added by the software developers themselves? Probably the best approach is that taken by [Gitian](https://gitian.org/), which provides a "deterministic build process to allow multiple builders to create identical binaries". We hope to adopt Gitian in the future.
