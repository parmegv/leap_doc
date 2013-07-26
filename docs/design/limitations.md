@title = 'Known Limitations'
@toc = true
@summary = 'Known limitations, issues, and security problems with the LEAP platform'

Herein lie the know limitations, issues, and security problems of the LEAP platform.

Provider problems
==========================================

Meta-data can be recorded by the provider
-------------------------------------------------

Currently, the service provider is able to observe the meta-data routing information of messages in transit of their own users. This information is not stored, but a nefarious provider could observe this information in transit and record it.

We have several plans to eliminate this, but these are not part of the initial release.

The provider can undermine the security of the web application
-------------------------------------------------------------------------

Both the client and the web application use something called SRP (Secure Remote Password) to prevent the server from ever seeing a cleartext copy of the password. This is in contrast to normal password systems, where the password is hashed on the server, so the server could record a copy of the password when it is initially set.

However, because all the javascript cryptographic libraries used by the user's web browser to perform the SRP negotiation are loaded from the provider's server, a nefarious or compromised provider could give the user's browser bad libraries that secretly sent a cleartext copy of the user's password.

There are two methods that can be used to prevent this:

* We could offer the option of first visiting a third party website that loads the authentication libraries before redirecting to the provider's website. Unfortunately, this user experience is a bit awkward.
* We could allow providers the option of not allowing authentication or signup through the webapp. Instead, the client could be distributed with an HTML page with all the javascript needed to authenticate. When the user wants to authenticate, the client loads this page in a web browser, authenticates with the provider's API, and then redirects to the actual website.

It is not either or, we could support both options.

The details for both of these are bit more tricky than in these simple descriptions, because of the need to work around the single origin policy, but it is still entirely possible to do this securely (using either CORS or PostMessage).

Still possible to brute force the password verifier
-----------------------------------------------------------------

SRP (Secure Remote Password) does not store a hash of the password in the provider's user database, but instead a password verifier. However, if an attacker gains access to the database of password verifiers (plus salts) they can perform a brute force attack similar to a normal attack on a database of password hashes. The attack in the case of SRP is more difficult, since there is more cryptographic work involved, but it is still possible.

To mitigate exposure of the password verifiers, we plan to separate them out into a separate database with only a separate single-purpose authentication API daemon granted access.

The provider can observe VPN traffic
--------------------------------------------------

The "Encrypted Internet" feature of LEAP currently works using OpenVPN secure tunnel to proxy all network traffic to a gateway operated by the provider. Once the traffic exits this gateway, it is cleartext (unless otherwise encrypted on the client, e.g. HTTPS) and could be observed and record by the provider, or any network observer able to monitor all traffic into and out of the gateway.

This limitation is mitigated by having the LEAP client authenticate with the VPN gateway using semi-anonymous or anonymous client certificates. A nefarious or compromised provider could attempt to record the moment that a user fetches a new client certificate, and record the IP address or authentication credentials of the user at that time.

In the future, we plan to remove this vulnerability in two ways:

* Allow the client to fetch new client certificates using blind signatures, so that there is no way for the provider to associate user with certificate, but we can also ensure that only valid users get client certificates.
* Use Tor as an alternate and optional transport for "Encrypted Internet". From the stand point of the user, it would work the same (using perhaps tun socks proxy and dnscrypt-proxy). This option would be slower and not support UDP traffic, but it would be much more secure. The Tor project prefers that every application that uses Tor be specifically designed for Tor so that it does not leak information in other ways. Using Tor as a default route like we do with OpenVPN would violate this, but would be more user friendly.

User problems
=================================

A compromised device is a sad device
----------------------------------------------

The LEAP client tries to minimize attacks related to physical access to the device. In particular, we try to be very resistant to offline attacks, where an attacker has captured the user's device while the LEAP client does not have an open session. For example, locally stored data is kept in an encrypted database that is only unlocked when the user authenticates with the application.

However, if an attacker gains access to the device, and then the device is returned to the user, they can do all kinds of nasty things, like install a keylogger that captures every keystroke.

This vulnerability is true of all software, not just LEAP, but it is worth noting.

Passwords are never going to be very good
---------------------------------------------------

LEAP relies on the user's password to unlock access to the user's client encrypted data storage. It does this the right way, using a solid KDF, but passwords are pretty limited and offer marginal security if an attacker gains offline access to the user's encrypted storage (for example, if they obtain the device).

In the future, we hope to add support for OpenPGP smart cards in order to overcome many of the problems associated with passwords.

Design problems
============================================

Enumeration of usernames
-----------------------------

The system LEAP uses to validate the public keys of users is inherently vulnerable to an attacker enumerating usernames. Because requests for public keys may be proxy'ed through other providers, there is no good method of preventing an attacker from launching many queries for public keys and eventually mapping most of the usernames.

This is unfortunate, but this is also a problem with all other such systems of key discovery and validation (i.e. DANE). For now, we consider this to be an acceptable compromise.

Much trust is placed in LEAP
-------------------------------------------

In order to shield the service provider from being pressured by a host government or criminal organization to add a backdoor into the client, the model with the LEAP platform is that the client is normally downloaded from the leap.se website and subsequent updates are signed by LEAP developers.

This is good for the provider, but not so good for LEAP, since this system could potentially place pressure on LEAP. Because LEAP does not have a provider-customer relationship with any user, LEAP cannot target compromised applications for particular users. LEAP could, however, introduce a backdoor in the client used by all users.

To prevent this, we place to adopt [Gitian](https://gitian.org/) or something equivalent. Gitian allows for a way to standardize the entire build environment and build process in order for third parties to be able to verify that the released binary application does indeed match the correct source code.

External authority problems
=================================================

Certificate authorities considered dangerous
---------------------------------------------------

The long term goal with LEAP is to entirely rid ourselves of reliance on the x.509 certificate authority system. However, there are a few places where the platform still relies on it:

* When the client first validates a new provider, it will assume the provider's TLS connection is valid if presented with a server certificate signed by a commercial CA recognized by the operating system.
* When nicknym discovers new public keys for users, it uses a TLS connection validated by a commercial CA recognized by the operating system.
* Currently, the web application does not get deployed with any other TLS validation than the standard commercial CA method. Eventually, we plan to support [DNS-based Authentication of Named Entities (DANE)](https://datatracker.ietf.org/wg/dane/), [Trust Assertions for Certificate Keys (TACK)](http://tack.io/), or [Public Key Pinning Extension for HTTP](https://datatracker.ietf.org/doc/draft-ietf-websec-key-pinning/).
