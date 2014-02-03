@title = "Graph Resistant Routing"
@summary = "LEAP's plans for protecting routing meta-data"

# A social graph is highly sensitive data

As messages are sent and delivered, they contain "meta-data" describing where these messages should be routed. With existing protocols for email and chat, this meta-data is sent in the clear and can be used to build a social graph of how people interact.

As the field of network analysis has advanced in recent years, the social graph has become highly sensitive and critical information. Knowledge of the social graph can give an attacker a blueprint into the inner workings of an organization or reveal surprisingly intimate personal details, such as sexual orientation or even health status.

In the short term, LEAP is opportunistically encrypting the message transport whenever possible. This protects the meta-data routing information from an external observer, but does not protect against a nefarious or compromised service provider. This page is about our plans for a better way.

# Possible solutions

There are four strategies we might employ to add protection of routing information to email and chat:

* Auto-alias-pairs: Each party auto-negotiates aliases for communicating with each other. Behind the scenes, the client then invisibly uses these aliases for subsequent communication. The advantage is that this is backward compatible with existing routing. The disadvantage is that the user's server stores a list of their aliases. As an improvement, you could add the possibility of a third party service to maintain the alias map.
* Onion-routing-headers: A message from user A to user B is encoded so that the "to" routing information only contains the name of B's server. When B's server receives the message, it unwraps (unencrypts) a supplementary header that contains the actual user "B". Like aliases, this provides no benefit if both users are on the same server. As an improvement, the message could be routed through intermediary servers.
* Third-party-dropbox: To exchange messages, user A and user B negotiate a unique "dropbox" URL for depositing messages, potentially using a third party. To send a message, user A would post the message to the "dropbox". To receive a message, user B would regularly polls this URL to see if there are new messages.
* Mixmaster-with-signatures: Messages are bounced through a mixmaster-like set of anonymization relays and then finally delivered to the recipient's server. The user's client only displays the message if it is encrypted, has a valid signature, and the user has previously added the sender to a 'allow list' (perhaps automatically generated from the list of validated public keys).
* Direct-delivery: Instead of relaying messages from client to server to server to client, in this model the sender's client delivers the message directly to the recipient's server. This delivery would need to happen over an anonymization network akin to Tor, or through proxies set up for this purpose. In order to prevent spam, the recipient server would only accept messages delivered in this manner if the message was signed using a group signature (this ensures that the server doesn't know who the sender is, but can confirm that they are allowed to deliver to a particular user). This would require advanced confirmation on the part of both users that they may send messages to one another. This is how Pond works.


None of these are currently used for email or chat.

# Auto alias pairs

How would auto alias pairs work? Imagine users Alice and Bob. Alice wants to correspond with Bob but doesn't want either her service provider or Bob's service provider to be able to record the association. She also doesn't want a network observer to be able to map the association.

When Alice first sends a message to Bob, Alice's client will initiate the following chain of events on her behalf, automatically and behind the scenes.

* Alice's client requests a new alias from her service provider. If her normal address is alice@domain.org, she receives an alias hjj3fpwv84fn@domain.org that will forward messages to her account.
* Alice's client uses automatic key discovery and validation to find Bob's public key and discover if Bob's service provider supports map resistant routing.
* If Bob does support it, Alice's client will then send a special message, encrypted to Bob's key, that contains Alice's public address and her private alias.
* When Bob's client encounters this special message, it records a mapping between Alice's public address (alice@domain.org) and the private alias she has created for Bob's use (hjj3fpwv84fn@domain.org).
* Bob's client then creates an alias for Bob and sends it to Alice.
* Alice's client receives this alias, and records the mapping between Bob's public address and his private alias.
* Alice's client then relays the original message to Bob's alias.

Subsequently, whenever Alice or Bob want to communicate, they use the normal public addresses for one another, but behind the scenes their clients will rewrite the source and recipient of the messages to use the private aliases.

This scheme is backwards compatible with existing messaging protocols, such as SMTP and XMPP.

## Limitations

There are five major limitations to this scheme:

1. **Alias unmasking attacks:** because each service provider maintains an alias map for their own users, an attacker who has gained access to this alias map can de-anonymize all the associations into and out of that particular provider, for the past and future.
2. **Timing attacks:** a statistical analysis of the time that messages of a particular size are sent, received, and relayed by both service providers could reveal the map of associations.
3. **Log correlation problem:** a powerful attacker could gain access to the logs of both service providers, and thereby reconstruct the associations between the two providers.
4. **Single provider problem:** this scheme does not protect associations between people on the same service provider.
5. **Client problem:** if the user's device is compromised, the record of their actual associations can be discovered.

At this point, we feel that it is OK to live with these limitations for the time being in order to simply the logic of the user's client application and to ensure backward compatibility with existing messaging protocols.

Possible future enhancements could greatly mitigate these attacks:

* We could use temporarily aliases that rotate daily, perhaps based on an HMAC counter, based on a shared secret between two users.
* The 'alias service' could be run by a third party, so that providers don't have access to the alias maps (thus migitating problems 1, 3, and 4).
* A service provider with sufficient traffic could be in a very good position to be able to aggregate and time-shift the messages it relays in order to disrupt timing attacks.

Ultimately, however, most of these attacks are a problem when faced with an extremely powerful adversary. This scheme is not designed for these situations. Instead, it is designed to prevent the casual, mass surveillance of all communication that currently takes place in repressive and democratic countries alike, by both governments and corporations. It greatly reduces the capacity for association mapping of both traffic over the wire and in stored communication. It is not designed to make a particular user anonymous when specifically targeted by a powerful adversary.

# How would X work?

How would onion-routing-headers, third-party-dropbox, or mixmaster-with-signatures work? To be written.
