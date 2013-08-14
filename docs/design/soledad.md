@title = 'Soledad'
@summary = 'A server daemon and client library to provide client-encrypted application data that is kept synchronized among multiple client devices.'
@toc = true

Introduction
=====================

Soledad is a system for to allow client applications the ability to securely share synchronized document databases. Soledad is based on Ubuntu's U1DB, "a cross-platform, cross-device, syncable database API", but with the addition of client-side encryption of documents stored on the server, and encryption of the local database replica.

Key aspects of Soledad include:

* **Client and server:** Soledad includes a server daemon and client application library.
* **Client-side encryption:** Soledad puts very little trust in the server by encrypting all data before it is synchronized to the server and by limiting ways in which the server can modify the user's data.
* **Local storage:** All data cached locally is stored in an encrypted database.
* **Document database:** An application using the Soledad client library is presented with a document-centric database API for storage and sync. Documents may be indexed, searched, and versioned.

The current reference implementation of Soledad is written in Python and distributed under a GPLv3 license.

Soledad is an acronym of "Synchronization of Locally Encrypted Documents Among Devices" and means "solitude" in Spanish.

Goals
======================

**Security goals**

* *Client-side encryption:* Before any data is synced to the cloud, it should be encrypted/decrypted on the client device.
* *Encrypted local storage:* Any data cached or stored on the client should be stored in an encrypted format.
* *Resistant to offline attacks:* Data stored on the server should be highly resistant to offline attacks (i.e. an attacker with a static copy of data stored on the server would have a very hard time discerning much from the data).
* *Resistant to online attacks:* Analysis of storing and retrieving data should not leak potentially sensitive information.
* *Resistance to data tampering:* The server should not be able to provide the client with old or bogus data for a document.

**Synchronization goals**

* *Consistency:* multiple clients should all get sync'ed with the same data.
* *Sync flag:* the ability to partially sync data. For example, so a mobile device doesn't need to sync all email attachments.
* *Multi-platform:* supports both desktop and mobile clients.
* *Quota:* the ability to identify how much storage space a user is taking up.
* *Scalable cloud:* distributed master-less storage on the cloud side, with no single point of failure.
* *Conflict resolution:* conflicts are flagged and handed off to the application logic to resolve.

**Usability goals**

* *Availability*: the user should always be able to access their data.
* *Recovery*: there should be a mechanism for a user to recover their data should they forget their password.

**Known limitations**

* Currently, the server knows when the contents of a document have changed.
* Currently, there is no facility for sharing documents among multiple users.

**Non-goals**

* Soledad is not for filesystem synchronization, storage or backup. It provides an API for application code to synchronize and store arbitrary schema-less JSON documents in one big flat document database. One could model a filesystem on top of Soledad, but it would be a bad fit.
* Soledad is not intended for decentralized peer-to-peer synchronization, although the underlying synchronization protocol does not require a server. Soledad takes a cloud approach in order to ensure that a client has quick access to an available copy of the data.

Related software
==================================

[Crypton](https://crypton.io/) - Similar goals to Soledad, but in javascript for HTML5 applications.

[U1DB](http://pythonhosted.org/u1db/) - Similar API as Soledad, without encryption.

Soledad protocol
===================================

Storage secret
-----------------------------------

When a client application first wants to use Soledad, it must provide the user's password to unlock the `storage_secret`. The `storage_secret` is a long, randomly generated symmetric key used to encrypt both the documents stored on the server and the local replica of these documents.

TO ADD: example code

The `storage_secret` is saved locally on disk in the file `soledad.json`, block encrypted using a derived key. The derived key is obtained from the user's password.

The file `soledad.json` has a field `storage_secrets` that looks like so:

    {
      "storage_secrets": {
        "<secret_id>": {
          "kdf": "scrypt",
          "kdf_salt": "400$8$5fb$61b499fe3366d947",
          "kdf_length": 128,
          "cipher": "aes128",
          "length": 512,
          "secret": "<encrypted storage_secret 1>",
        }
      }
    }

The `storage_secrets` entry is a map that stores information about each storage key, indexed by the id of each key. For each storage key, the following fields are stored:

* `kdf`: the key derivation function to use. Only scrypt is currently supported (so for now, this value is ignored).
* `kdf_salt`: the salt used in the kdf. The salt for scrypt is not random, but encodes important parameters like the limits for time and memory.
* `kdf_length`: the length of the derived key resulting from the kdf.
* `secret`: the encrypted `storage_secret`, created by `sym_encrypt(cipher, storage_secret, derived_key)` (base64 encoded).
* `length`: the length of `storage_secret`, when not encrypted.
* `cipher`: what cipher to use to encrypt `storage_secret`. It must match kdf_length (i.e. the length of the derived_key).
* `secret_id`: a handle used to refer to a particular storage_secret and equal to `md5(storage_secret)`.

Other variables:

* `derived_key` is equal to `kdf(user_password, kdf_salt, kdf_length)`.
* `storage_secret` is equal to `sym_decrypt(cipher, secret, derived_key)`.

In the current version, only one `storage_secret` is supported.

The `storage_secret` is shared among all devices with access to a particular user's Soledad database. See [Recovery and bootstrap](#Recovery.and.bootstrap) for how the storage_secret is initially installed on a device.

We don't use the derived_key as the storage_secret because we want the user to be able to change their password without needing to re-key.

TO DO: Settle on a block cipher. Define how changes to storage secret are sync'ed.

Document API
-----------------------------------

This is unchanged and identical to the [API used in U1DB](http://pythonhosted.org/u1db/reference-implementation.html).

* Document storage: `create_doc()`, `put_doc()`, `get_doc()`.
* Synchronization between database replicas: `sync()`.
* Document indexing and searching: `create_index()`, `list_indexes()`, `get_from_index()`, `delete_index()`.
* Document conflict resolution: `get_doc_conflicts()`, `resolve_doc()`.

TO ADD: code examples

Document encryption
------------------------

Before a JSON document is synced with the server, it is transformed into a document that looks like so:

    {
      "scheme": "aes128",
      "secret_id": "1",
      "ciphertext": "xxxxxxxxx",
      "mac": "xxxxxxx"
    }

About these fields:

* `ciphertext`: The original JSON document, encrypted and base64 encoded. `ciphertext` is equal to `sym_encrypt(cipher, content, document_secret)`.
* `scheme`: Information about the block cipher that is used to encrypt this document.
* `secret_id`: The id of the storage_secret that was used to generate the `document_key`.
* `mac`: Defined as `HMAC(doc_id|rev|ciphertext, document_secret)`. The purpose of this field is to prevent the server from tampering with the stored documents.

Other variables:

* `document_secret`: equal to `HMAC(doc_id, storage_secret)`. This value is unique for every document and only kept in memory. We use document_secret instead of simply storage_secret in order to hinder possible derivation of storage_secret by the server. Every `doc_id` is unique.
* `content`: equal to `sym_decrypt(cipher, ciphertext, document_secret)`.

When receiving a document with the above structure from the server, Soledad client will first verify that `mac` is correct, then decrypt the `ciphertext` to find `content`, which it saves as a cleartext document in the local database replica.

TO DO: specify supported ciphers

TO DO: specify supported HMAC

Document synchronization
-----------------------------------

Soledad follows the U1DB synchronization protocol, with two changes:

* Soledad adds the ability to flag some documents so they are not synchronized by default.
* Soledad will refuse to synchronize a document if it is encrypted and the MAC is incorrect.

TO ADD: code examples

Document IDs
--------------------

Like U1DB, Soledad allows the programmer to use whatever ID they choose for each document. However, it is best practice to let the library choose random IDs for each document so as to ensure you don't leak information. In other words, leave the second argument to `create_doc()` empty.

Re-keying
-----------

Sometimes there is a need to change the `storage_secret`. Rather then re-encrypt every document, Soledad implements a system called "lazy revocation" where a new storage_secret is generated and used for all subsequent encryption. The old storage_secret is still retained and used when decrypting older documents that have not yet been re-encrypted with the new storage_secret.

Implementation status: not yet.

TO DO: code example

Authentication
-----------------------

Unlike U1DB, Soledad only supports token authentication and does not support not support OAuth. Soledad itself does not handle authentication. Instead, this job is handled by a thin HTTP middleware layer running in front of the Soledad server daemon. How the session token is obtained is beyond the scope of Soledad.

Recovery and bootstrap
------------------------------------------

In order to bootstrap Soledad on a new device, or to recover data if all your devices have been wiped clean, the user has two options:

1. password method: recover by specifying username and password.
2. recovery code method: recover by specifying username and recovery code.

This choice must be made in advance of attempting recovery (e.g. at some point after the user has Soledad successfully running on a device).

**Recovery code**

About the optional recovery code:

* The recovery code should be randomly generated, at least 16 characters in length, and contain all lowercase letters (to make it sane to type into mobile devices).
* The recovery code is not stored by Soledad. When the user needs to bootstrap a new device, a new code is generated. To be used for actual recovery, a user will need to record their recovery code by printing it out or writing it down.
* The recovery code is independent of the password. In other words, if a recovery code is generated, then a user changes their password, the recovery code is still be sufficient to restore a user's account even if the user has lost the password. This feature is dependent on the server supporting a password reset token. Also, generating a new recovery code does not affect the password.
* When a new recovery code is created, and new recovery document must be pushed to the recovery database. A code should not be shown to the user before this happens.
* The recovery code expires when the recovery database record expires (see below).

The purpose of the recovery code is to prevent a compromised or nefarious Soledad service provider from decrypting a user's storage. The benefit of a recovery code over the user password is that the password has a greater opportunity to be compromised by the server. Even if authentication is performed via Secure Remote Password, the server may still perform a brute force attack to derive the password.

**Recovery database**

In order to support easy recovery, the Soledad client stores a recovery document in a special recovery database. This database is shared among all users.

The recovery database supports two functions:

* `get_doc(doc_id)`
* `put_doc(doc_id, recovery_document_content)`

Anyone may preform an unauthenticated `get_doc` request. To mitigate the potential attacks, the response to queries of the discovery database must have a long delay of X seconds. Also, the `doc_id` is very long (see below).

Although the database is shared, the user must authenticate via the normal means before they are allowed to put a recovery document. Because of this, a nefarious server might potentially record which user corresponds to which recovery documents. A well behaved server, however, will not retain this information. If the server supports authentication via blind signatures, then this will not be an issue.

**Recovery document**

An example recovery document:

    {
      "doc_id": "xxxxx"
      "kdf": "scrypt",
      "kdf_salt": "400$8$5fb$61b499fe3366d947",
      "kdf_length": 128,
      "cipher": "aes128",
      "expires_on": "2014-07",
      "soledad": "xxxxx",
      "reset_token": "xxxxx"
    }

About these fields:

* `doc_id` is determined by the client and computed from `hmac(username@domain, passcode)`.
* `kdf`: the key derivation function to use. Only scrypt is currently supported (so for now, this value is ignored).
* `kdf_salt`: the salt used in the kdf. The salt for scrypt is not random, but encodes important parameters like the limits for time and memory.
* `kdf_length`: the length of the derived key resulting from the kdf.
* `cipher`: what cipher to use to encrypt `soledad`. It must match `kdf_length` (i.e. the length of the derived_key).
* `expires_on`: the month in which this recovery document should be purged from the database. The server may choose to purge documents before their expiration, but it should not let them linger after it.
* `soledad`: the encrypted `soledad.json`, created by `sym_encrypt(cipher, contents(soledad.json), derived_key)` (base64 encoded).
* `reset_token`: an optional encrypted password reset token, if supported by the server, created by `sym_encrypt(cipher, password_reset_token, derived_key)` (base64 encoded). The purpose of the reset token is to allow recovery using the recovery code even if the user has forgotten their password. It is only applicable if using recovery code method.

Other variables:

* `passcode`: either the user password or the recovery code, depending on the option selected by the user.
* `derived_key`: equal to `kdf(passcode, kdf_salt, kdf_length)`.


Reference implementation of client
===================================

https://github.com/leapcode/soledad

Dependencies:

* [U1DB](https://launchpad.net/u1db) provides an API and protocol for synchronized databases of JSON documents.
* [SQLCipher](http://sqlcipher.net/) provides a block-encrypted SQLite database used for local storage.
* python-gnupg

Local storage
--------------------------

U1DB reference implementation in Python has an SQLite backend that implements the object store API over a common SQLite database residing in a local file. To allow for encrypted local storage, Soledad adds a SQLCipher backend, built on top of U1DB's SQLite backend, which adds [SQLCipher API](http://sqlcipher.net/sqlcipher-api/) to U1DB.

**Responsibilities**

The SQLCipher backend is responsible for:

* Providing the SQLCipher API for U1DB (`PRAGMA` statements that control encryption parameters).
* Guaranteeing that the local database used for storage is indeed encrypted.
* Guaranteeing secure synchronization:
  * All data being sent to a remote replica is encrypted with a symmetric key before being sent.
  * Ensure that data received from remote replica is indeed encrypted to a symmetric key when it arrives, and then that it is decrypted before being included in the local database replica.
* Correctly representing and handling new Document properties as sync flag.

The Soledad `storage_key` is used directly as the key for the SQLCipher encryption layer. SQLCipher supports the use of a raw 256 bit keys if provided as a 64 character hex string. This will skip the key derivation step (PBKDF2), which is redundant in our case. For example:

    sqlite> PRAGMA key = "x'2DD29CA851E7B56E4697B0E1F08507293D761A05CE4D1B628663F411A8086D99'";

**Classes**

SQLCipher backend classes:

* `SQLCipherDatabase`: An extension of SQLitePartialExpandDatabase used by Soledad Client to store data locally using SQLCipher. It implements the following:
  * Need of a password to instantiate the db.
  * Verify if the db instance is indeed encrypted.
  * Use a LeapSyncTarget for encrypting content before synchronizing over HTTP.
  * "Syncable" option for documents (users can mark documents as not syncable, so they do not propagate to the server).

Encrypted synchronization target
--------------------------------------------------

To allow for database synchronization among devices, Soledad uses the following conventions:

* Centralized synchronization scheme: Soledad clients always sync with a server, and never between themselves.
* The server stores its database in a CouchDB database using a REST API over HTTP.
* All data sent to the server is encrypted with a symmetric secret before being sent. Note that this ensures all data received by the server and stored in the CouchDB database has been encrypted by the client.
* All data received from the server is validated as being an encrypted blob, and then is decrypted before being stored in local database. Note that the local database provides a new encryption layer for the data through SQLCipher.

**Responsibilities**

Provide sync between local and remote replicas:

* Encrypt outgoing content.
* Decrypt incoming content.

**Classes**

Synchronization-related classes:

* `LEAPDocument`: an extension of @u1db.Document@ with methods to:
  * Return a symmetric encrypted version of Documents JSON representation.
  * Set document's content by symmetric decrypting an encrypted JSON representation.
* `LEAPSyncTarget`: an extension of `HTTPSyncTarget` with the following modified methods:
  * `sync_exchange`: request encrypted version of Document's content before sending it to the network.
  * `_parse_sync_stream`: set Document's content based on encrypted version right after it arrives as a response from the network.

Reference implementation of server
======================================================

https://github.com/leapcode/soledad

Dependencies:

* [CouchDB](https://couchdb.apache.org/] for server storage, via [python client library](https://pypi.python.org/pypi/CouchDB/0.8).
* WSGI middleware for authentication.
* [Twisted](http://twistedmatrix.com/trac/) to run the WSGI application.

CouchDB backend
-------------------------------

In the server side, Soledad stores its database replicas in CouchDB servers. Soledad's CouchDB backend implementation is built on top of the reference `InMemory` implementation, but forces storage and fetch of U1DB data on a remote couch server for every write and read operation, respectively.

CouchDB backend is responsible for:

* Initializing and maintaining the following U1DB replica data in the database:
  * Transaction log.
  * Conflict log.
  * Synchronization log.
  * Indexes.
* Mapping the U1DB API to CouchDB API.

**Classes**

* `CouchDatabase`: A backend used by Soledad Server to store data in CouchDB.
* `CouchSyncTarget`: Just a target for syncing with Couch database.
* `CouchServerState`: Inteface of the WSGI server with the CouchDB backend.

WSGI Server
-----------------------------------------

The U1DB server reference implementation provides for an HTTP api backed by SQLite databases (of minimal usefulness in production environment!). Soledad extends this with token-based auth HTTP access to CouchDB databases.

* Soledad makes use of @twistd@ from Twisted API to serve its WSGI application.
* Authentication is done by means of a token.
* Soledad implements a WSGI middleware in server side that:
  * Uses the provided token to verify read and write access to each user's private databases and write access to the shared recovery database.
  * Allows reading from the shared remote recovery database.
  * Uses CouchDB as its backend.

**Classes**

* `SoledadAuthMiddleware`: implemnets the WSGI middleware with token based auth as described before.
* `SoledadApp`: The WSGI application. For now, not different from `u1db.remote.http_app.HTTPApp`.

**Authentication**

Soledad Server authentication middleware controls access to user's private databases and to the shared recovery database. Soledad client provides a token for Soledad server that can check the validity of this token for this user's session by querying a certain database.

A valid token for this user's session is required for:

* Read and write access to this user's database.
* Read and write access to the shared recovery database.

Tests
===================

To be sure the new implemented backends work correctly, we included in Soledad the U1DB tests that are relevant for the new pieces of code (backends, document, http(s) and sync tests). We also added specific tests to the new functionalities we are building.
