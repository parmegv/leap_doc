@title = 'Known Limitations'
@toc = true
@summary = 'Known limitations, issues, and security problems with the LEAP platform'

Meta-data may be recorded by the provider
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

The provider can observe VPN traffic
--------------------------------------------------

The "Encrypted Internet" feature of LEAP currently works using OpenVPN secure tunnel to proxy all network traffic to a gateway operated by the provider. Once the traffic exits this gateway, it is cleartext (unless otherwise encrypted on the client, e.g. HTTPS) and could be observed and record by the provider, or any network observer able to monitor all traffic into and out of the gateway.

This limitation is mitigated by having the LEAP client authenticate with the VPN gateway using semi-anonymous or anonymous client certificates. A nefarious or compromised provider could attempt to record the moment that a user fetches a new client certificate, and record the IP address or authentication credentials of the user at that time.

In the future, we plan to remove this vulnerability in two ways:

* Allow the client to fetch new client certificates using blind signatures, so that there is no way for the provider to associate user with certificate, but we can also ensure that only valid users get client certificates.
* Use Tor as an alternate and optional transport for "Encrypted Internet". From the stand point of the user, it would work the same (using perhaps tun socks proxy and dnscrypt-proxy). This option would be slower and not support UDP traffic, but it would be much more secure. The Tor project prefers that every application that uses Tor be specifically designed for Tor so that it does not leak information in other ways. Using Tor as a default route like we do with OpenVPN would violate this, but would be more user friendly.

A compromised device is a sad device
----------------------------------------------

The LEAP client tries to minimize attacks related to physical access to the device. In particular, we try to be very resistant to offline attacks, where an attacker has captured the user's device while the LEAP client does not have an open session. For example, locally stored data is kept in an encrypted database that is only unlocked when the user authenticates with the application.

However, if an attacker gains access to the device, and then the device is returned to the user, they can do all kinds of nasty things, like install a keylogger that captures every keystroke.

This vulnerability is true of all software, not just LEAP, but it is worth noting.





