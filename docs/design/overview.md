@nav_title = "Overview"
@title = "Overview of LEAP architecture"
@summary = "Bird's eye view of how all the pieces fit together."

The LEAP Platform allows an organization to deploy and manage a complete infrastructure for providing user communication services.

This document gives a brief overview of how the pieces fit together.

LEAP Client
===================

The LEAP Client is an application that runs on a user's own device and is responsible for all encryption of user data. The client must be installed a user's device before they can access any LEAP services (except for user support via the web application).

Desktop Client
--------------------------

LEAP Client for Linux, Windows, and Mac.

Written in: Python

Libraries used: QT, PyQT, OpenVPN, Sqlite, Sqlcipher, U1DB, OpenSSL, GPG.

User interface:

* First run wizard: walks the user through the bootstrap process when the client is first run (either registering a new user or authenticating as an existing user)
* Preferences panel: A mac system-preferences-like place to edit all the LEAP client settings (does not exist yet).
* Task bar: Show the status of LEAP services (connected? syncing?), and lets the user open the preferences panel.
* Update wizard: a dialog that shows the code update progress.

Android Client
------------------------------

LEAP Client for Android.

Written in: Java (possibly with with some Python in the future)

Libraries used: sqlcipher, sqlite, bouncycastle, U1DB, OpenVPN.

User interface:

* Single button to connect or disconnect encrypted internet
* A notification drawer item indicating status of VPN
* A first run wizard

Features (planned):

* a sync provider to allow contacts and calendar data to be sync'ed via Soledad.
* eventually, match the desktop client in features.


LEAP Admin Tools
====================================

Platform Recipes
------------------------------

The LEAP platform recipes define an abstract service provider. It consists of puppet modules designed to work together to provide a system administrator everything they need to manage a service provider infrastructure that provides secure communication services.

Typically, a system administrator will not need to modify the LEAP platform recipes, although they are free to fork and merge as desired. Most service providers using the LEAP platform will use the same platform recipes.

The recipes are abstract. In order to configure settings for a particular service provider, a system administrator creates a provider instance. The platform recipes also include a base provider that provider instances inherit from.

Provider Instance
----------------------------------

A "provider instance" is a directory tree (typically tracked in git) containing all the configurations for a service provider's infrastructure. A provider instance primarily consists of:

* A configuration file for each server (node) in the provider's infrastructure (e.g. nodes/vpn1.json)
* A global configuration file for the provider (e.g. provider.json).
* Additional files, such as certificates and keys (e.g. files/nodes/vpn1/vpn1_ssh.pub).
* A pointer to the platform recipes (as defined in "Leapfile")

A minimal provider instance directory looks like this:


    └── bitmask                 # provider instance directory.
        ├── common.json         # settings common to all nodes.
        ├── Leapfile            # various settings for this instance.
        ├── provider.json       # global settings of the provider.
        ├── files/              # keys, certificates, and other files.
        ├── nodes/              # a directory for node configurations.
        └── users/              # public key information for privileged sysadmins.

A provider instance directory contains everything needed to manage all the servers that compose a provider's infrastructure. Because of this, you can use normal git development work-flow to manage your provider instance.

Command line program
-------------------------------

The command line program `leap` is used by sysadmins to manage everything about a service provider's infrastructure. Except when creating an new provider instance, `leap` is run from within the directory tree of a provider instance.

The `leap` command line has many capabilities, including:

* create an initial provider instance
* create, initialize, and deploy nodes (e.g. servers)
* manage keys and certificates
* query information about the node configurations

Traditional system configuration automation systems, like puppet or chef, deploy changes to servers using a pull method. Each server pulls a manifest from a central master server and uses this to alter the state of the server.

Instead, LEAP uses a masterless push method: The user runs 'leap deploy' from the provider instance directory on their desktop machine to push the changes out to every server (or a subset of servers). LEAP still uses puppet, but there is no central master server that each node must pull from.

One other significant difference between LEAP and typical system automation is how interactions among servers are handled. Rather than store a central database of information about each server that can be queried when a recipe is applied, the `leap` command compiles static representation of all the information a particular server will need in order to apply the recipes. In compiling this static representation, `leap` can use arbitrary programming logic to query and manipulate information about other servers.

These two approaches, masterless push and pre-compiled static configuration, allow the sysadmin to manage a set of LEAP servers using traditional software development techniques of branching and merging, to more easily create local testing environments using virtual servers, and to deploy without the added complexity and failure potential of a master server.

Server-side Components
=======================================

These are components where most of the code and logic runs on a server (as opposed to client-side components, where most of the code runs on the client).

Databases
------------------------------------

All user data is stored using BigCouch, a decentralized and high-availability version of CouchDB.

The databases are used by the different services and sometimes work as communication channels between the services.

These are the databases we currently use:

* customers -- payment information for the webapp
* identities -- alias information, written by the webapp, read by leap_mx and nickserver
* keycache -- used by the nickserver
* sessions -- web session persistance for the webapp
* shared -- used by soledad
* tickets -- help tickets issued in the webapp
* tokens -- created by the webapp on login, used by soledad to authenticate
* users -- user records used by the webapp including the authentication data
* user-...id... -- client-encrypted data for the user with the given id accessed from the client via soledad

Like many NoSQL databases, BigCouch is inspired by [Amazon's Dynamo paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) and works by sharding each database among many servers using a circular ring hash. The number of shards might be greater than the number of servers, in which case each server would have multiple shards of the same database. Each server in the BigCouch cluster appears to contain the entire database, but actually it will just proxy the request to the actual database that has the content (if it does not have the document itself).

Important BigCouch constants:

* Q -- The number of shards over which a database will spread.
* N -- The number of redundant copies of each document. Default is 3.
* W -- The number of document copies that must be saved before document is 'written'. Default is 2.
* R -- The number of document copies that must be found before document is 'read'. Default is 2.
* Z -- The number of zones in the cluster. Each zone will have a complete copy of all the data. Default is 1.

In LEAP, every service that needs to interact with the database runs a local HTTP load balancer that distributes database requests randomly to the BigCouch cluster. If a BigCouch node dies, the load balancer detects this and takes it out of rotation (this usage is typical of BigCouch installations).

Web App
------------------------------

The LEAP Web App provides the following functions:

* User registration and management
* Help tickets
* Client certificate renewal
* Webfinger access to user's public keys
* Email alias and forwarding

Written in: Ruby, Rails.

The Web App communicates with:

* CouchDB is used for all data storage.
* Web browsers of users accessing the user interface in order to edit their settings or fill out help tickets. Additionally, admins may delete users.
* LEAP Clients access the web app's REST API in order to register new users, authenticate existing ones, and renew client certificates.
* tokens are stored upon successful authentication to allow the client to authenticate against other services

Nickserver
------------------------------

Written in: Ruby
Libaries: EventMachine, GPG

Nickserver is the opposite of a key server. A key server allows you to lookup keys, and the UIDs associated with a particular key. A nickserver allows you to query a particular 'nick' (e.g. username@example.org) and get back relevant public key information for that nick.

Nickserver has the following properties:

* Written in Ruby, licensed GPLv3
* Lightweight and scalable (high concurrency, reasonable latency)
* Uses asynchronous network IO for both server and client connections (via EventMachine)
* Attempts to reply to queries using four different methods:
    * Cached key in CouchDB
    * Webfinger
    * DNS
    * HKP keyserver pool (https://hkps.pool.sks-keyservers.net)

Why bother writing Nickserver instead of just using the existing HKP keyservers?

* Keyservers are fundamentally different: Nickserver is a registry of 1:1 mapping from nick (uid) to public key. Keyservers are directories of public keys, which happen to have some uid information in the subkeys, but there is no way to query for an exact uid.
* Support clients: the goal is to provide clients with a cloud-based method of rapidly and easily converting nicks to keys. Client code can stay simple by pushing more of the work to the server.
* Enhancements over keyservers: the goal with Nickserver is to support future enhancements like webfinger, DNS key lookup, mail-back verification, network perspective, and fast distribution of short lived keys.
* Scalable: the goal is for a service that can handle many simultaneous requests very quickly with low memory consumption.

Miscellaneous
------------------------------

A LEAP service provider might also run servers with the following services:

* git -- private git repository hosting.
* Domain Name Server -- Authoritative name server for the provider's domain.
* Tapicero -- headless daemon that watches couch changes for new users and creates their databases

Client-side Components
======================================

Most of the code and processing for these components happens on the client-side, although they all include some interaction with cloud services.

Soledad
------------------------------

Soledad stands for "Synchronization Of Locally Encrypted Data Among Devices". On the client side, Soledad is responsible for client-encrypting user data, keeping it in sync with the copy in the cloud, and for providing local applications with a simple API for data storage. This "client-side Soledad" is essentially a local database that is kept in sync with the cloud. The "Soledad Server" is the cloud-based component that the client syncs with.

Written in: Python (on desktops and servers), possibly Java (on android, not yet written).

Libraries used:

* Client-side: U1DB, Sqlite, Sqlcipher, GPG.
* Server: U1DB (forked), CouchDB.

Client-side Soledad communicates with:

* Other client application code, providing a storage API.
* Soledad Server via the U1DB synchronization protocol.

Soledad Server communicates with:

* LEAP Client via the U1DB synchronization protocol
* CouchDB or OpenStack Object Storage for backend storage.

Client-side Soledad Notes:

* Soledad is an modification of U1DB python reference implementation with changes to support client-side encryption and to replace sqlite with sqlcipher.
* Local data is stored on disk as an SQLite DB file(s) that is block-encrypted with sqlcipher (AES128).
* Before being synced to the server, a document is block-encrypted using a symmetric key composed from HMAC of the document id and a long secret (Soledad secret).
* Soledad secret is stored on-disk encrypted to the user's OpenPGP key. A copy is stored on the server as well. The same secret is shared among all the clients a user has activated.
* Soledad inherits these traits from U1DB:
    * The storage API used by client code is similar to couchdb (schema-less document storage with indexes).
    * Application code using Soledad is responsible for resolving sync conflicts.

Secrets Manager
------------------------------

Not yet written.

Written in: Python
Libraries used: GPG, GnuTLS

Communicates with: Nickserver (cloud), Soledad (local).

The Secrets Manager is a library that exposes to local client code an API for managing cryptographic material. It is responsible for:

* private secrets: the user's private and public keys and certificates.
* public keys: discovering, registering, and trusting the public keys of other people.
* creation: creating keys as needed.
* renewal: fetch a new client certificate when the current one is about to expire.
* recovery: allow the user to recover their data if they lose everything except for a recovery code.
* crypto hardware: allow the user to unlock secrets via an OpenPGP smart card like cryptostick.

**Example secrets**

* A user's OpenPGP keypair
* The symmetric key used to encrypt local data (used by sqlcipher)
* The client certificate used to auth with OpenVPN gateway.
* The client certificate used to auth with the SMTP gateway.

**Public key management**

Some functionality of public key management:

* Discover the public keys of recipients and senders via a Nickserver.
* "Register" the discovered keys, either using a federated path through the provider, directly, or via trust on first use (TOFU). For now, we will start initially with TOFU.
* Allow the user to choose between two competing keys when a recipient has multiple candidate keys.
* Allow the user to specify keys that should be not used.
* Allow the user to manually specify a user's public key.

**Recovery**

* Allow the user to generate and print out a recovery code. This creates a record on the server, in an anonymized way, that can be used to restore all the secrets stored by the key/secret manager and thus recover all your data. The provider should not know what recovery information maps to which user.
* Eventually, perhaps allow the user to specify other users who have the power to recover their lost secrets in the event that the user forgets their password.
* Allow the user to enter this recovery code when they have lost their username and password. If this is enabled, the user's private keys are stored in the cloud, albeit encrypted and anonymized.
* Give some users the option of full recovery via email reset by storing the user's password on the server. This would be a very low security option, but one that some users may wish to opt-in for.

**Notes**

* All secrets are stored in Soledad, except the secret to unlock Soledad storage. This way, all clients will have access to the same secrets. For some things, like validated public keys, this is exactly what we want. For other things, this could be a problem, and should be refined in future revisions.
* The current scheme is to store the user's private keys and private secrets in their Soledad storage. This allows a user to login with a different device and be all set up. There are, however, certainly problems with this approach.


Bootstrap
------------------------------

Parts of this are written.

Written in: Python

* Register new accounts or authenticate via the REST API, using SRP.
* Download the providers definition file, and various service definition files.
* Validate the CA certificate of the service provider.
* If using an existing account on a new device, fetch user's secrets from the cloud (not yet written).
* If creating a new account, generate a key pair and store in the cloud (not yet written).

Update Manager
------------------------------

Not yet written.

Handles upgrading the client by downloading and installing signed code.

Three goals:

* Frequent Updates: we want to be able to push out small and frequent updates should the need arise.
* Secure Updates: we want to ensure that the update mechanism cannot be used as an attack vector.
* Third Party Updates: we want a third party to be responsible for updates, NOT the service provider itself.

End User Services
=========================================

Email
------------------------------

Not yet working, some of the parts are written.

Written in: Python

Email in the client consists of three parts:

* SMTP Proxy: for outgoing mail.
    * Communicates with user's MUA (local), Key Manager (local), Nickserver (cloud), and SMTP relay (cloud).
* Message Receiver: for incoming mail.
    * Communicates with Soledad (local), Key Manager (local).
* IMAP Server: for reading and writing to user's mailbox.
    * Communicates with Soledad (local), user's MUA (local).

Outgoing mail workflow:

* LEAP client runs a thin SMTP proxy on the user's device, bound to localhost.
* User's MUA is configured outgoing SMTP to localhost
* When SMTP proxy receives an email from MUA
    * SMTP proxy queries Key Manager for the user's private key and public keys of all recipients
    * Message is signed by sender and encrypted to recipients.
    * If recipient's key is missing, email goes out in cleartext (unless user has configured option to send only encrypted email)
    * Finally, message is relayed to provider's SMTP relay

Incoming email workflow:

* Incoming message is received by provider's MX servers.
* Message is encrypted to the user's public key (if not already so), and stored in the user's incoming message queue.
* Message queue is synced to client device via Soledad.
* "Message Receiver" in the LEAP Client empties message queue, unencrypting each message and saving it in the user's inbox, stored in local Soledad database.
* Local database gets client-encrypted and sync'ed to cloud and other devices owned by the user via Soledad.

Mail storage workflow:

* LEAP client runs a thin IMAP server on the user's device, bound to localhost.
* User's MUA is configured to use localhost for the mail account.
* Local IMAP server runs against a local database the user's email data (access via Soledad).
* Soledad will sync changes made to mailboxes with the cloud and other clients.

Encrypted Internet
------------------------------

The goal behind the encrypted internet service is to provide an automatic, always on, trouble free way to encrypt a user's network traffic. For now, we use OpenVPN for the transport (OpenVPN uses TLS for session negotiation and IPSec for data).

Written in: C (OpenVPN binary), Python (desktop controlling code), Java (android controlling code)
Libraries: QT
Uses: OpenVPN

Communicates with:

* All traffic is routed through one of the provider's OpenVPN gateways
* OpenVPN binary and LEAP client communicate via a telnet administration interface to OpenVPN.
* Client discovers gateways and fetches client certificate from the provider's HTTP API.

User Interface:

* Initial connection attempt takes place in the first run wizard, displaying any errors along the way.
* After first run, the client will display the status of the encrypted internet in the task tray (windows, linux), menu bar (mac), or notification drawer (android).
* The three main UI functions of the encrypted internet will be: connect/disconnect, choose gateway, view errors.

Notes:

* OpenVPN must be started with superuser privileges (or have the ability to execute network changes as superuser). Afterwards, it can drop the privileges.
* OpenVPN authentication with the gateway uses an x.509 client certificate. This certificate is short lived, and is acquired by the client from the provider's HTTP API as needed.

Workflow:

* user installs client
* on first run
    * client downloads and validates service provider's definition file, CA cert, and encrypted internet service definition file.
    * user registers new account or authenticates with provider's webapp REST API
        * SRP is used, server never sees the password and does not store a hash of the password.
        * if registering, new record is created for user in distributed users db.
* client gets a new client certificate from webapp, if missing or expired
    * authenticate via SRP with webapp
    * webapp retrieves client cert from a pool of pre-generated certificates.
    * cert pool is filled as needed by background CA deamon.
* client connects to openvpn gateway, picked from among those listed in service definition file, authenticating with client certificate.
* by default, when user starts computer the next time, client autoconnects.
