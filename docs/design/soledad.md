@title = 'Soledad'
@summary = 'A server daemon and client library to provide client-encrypted application data that is kept synchronized among multiple client devices.'
@toc = true

Introduction
=====================

Soledad allows client applications to securely share synchronized document databases. Soledad aims to provide a cross-platform, cross-device, syncable document storage API, with the addition of client-side encryption of database replicas and document contents stored on the server.

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

* *Client-side encryption:* Before any data is synced to the cloud, it should be encrypted on the client device.
* *Encrypted local storage:* Any data cached in the client should be stored in an encrypted format.
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

These are currently known limitations:

* The server knows when the contents of a document have changed.
* There is no facility for sharing documents among multiple users.
* Soledad is not able to prevent server from withholding new documents or new revisions of a document.
* Deleted documents are never deleted, just emptied. Useful for security reasons, but could lead to DB bloat.

**Non-goals**

* Soledad is not for filesystem synchronization, storage or backup. It provides an API for application code to synchronize and store arbitrary schema-less JSON documents in one big flat document database. One could model a filesystem on top of Soledad, but it would be a bad fit.
* Soledad is not intended for decentralized peer-to-peer synchronization, although the underlying synchronization protocol does not require a server. Soledad takes a cloud approach in order to ensure that a client has quick access to an available copy of the data.

Related software
==================================

[Crypton](https://crypton.io/) - Similar goals to Soledad, but in javascript for HTML5 applications.

[Mylar](https://github.com/strikeout/mylar) - Like Crypton, Mylar can be used to write secure HTML5 applications in javascript. Uniquely, it includes support for homomorphic encryption to allow server-side searches.

[Firefox Sync](https://wiki.mozilla.org/Services/Sync) - A client-encrypted data sync from Mozilla, designed to securely synchronize bookmarks and other browser settings.

[U1DB](http://pythonhosted.org/u1db/) - Similar API as Soledad, without encryption.

Soledad protocol
===================================

Document API
-----------------------------------

Soledad's document API is similar to the [API used in U1DB](http://pythonhosted.org/u1db/reference-implementation.html).

* Document storage: `create_doc()`, `put_doc()`, `get_doc()`.
* Synchronization with the server replica: `sync()`.
* Document indexing and searching: `create_index()`, `list_indexes()`, `get_from_index()`, `delete_index()`.
* Document conflict resolution: `get_doc_conflicts()`, `resolve_doc()`.

For example, create a document, modify it and sync:

    sol.create_doc({'my': 'doc'}, doc_id='mydoc')
    doc = sol.get_doc('mydoc')
    doc.content = {'new': 'content'}
    sol.put_doc(doc)
    sol.sync()

Storage secret
-----------------------------------

The `storage_secret` is a long, randomly generated key used to derive encryption keys for both the documents stored on the server and the local replica of these documents. The `storage_secret` is block encrypted using a key derived from the user's password and saved locally on disk in a file called `<user_uid>.secret`, which contains a JSON structure that looks like this:

    {
      "storage_secrets": {
        "<secret_id>": {
          "kdf": "scrypt",
          "kdf_salt": "<b64 repr of salt>",
          "kdf_length": <key_length>,
          "cipher": "aes256",
          "length": <secret_length>,
          "secret": "<encrypted storage_secret>",
        }
      }
      'kdf': 'scrypt',
      'kdf_salt': '<b64 repr of salt>',
      'kdf_length: <key length>
    }

The `storage_secrets` entry is a map that stores information about available storage keys. Currently, Soledad uses only one storage key per provider, but this may change in the future.

The following fields are stored for one storage key:

* `secret_id`: a handle used to refer to a particular `storage_secret` and equal to `sha256(storage_secret)`.
* `kdf`: the key derivation function to use. Only scrypt is currently supported.
* `kdf_salt`: the salt used in the kdf. The salt for scrypt is not random, but encodes important parameters like the limits for time and memory.
* `kdf_length`: the length of the derived key resulting from the kdf.
* `cipher`: what cipher to use to encrypt `storage_secret`. It must match `kdf_length` (i.e. the length of the derived_key).
* `length`: the length of `storage_secret`, when not encrypted.
* `secret`: the encrypted `storage_secret`, created by `sym_encrypt(cipher, storage_secret, derived_key)` (base64 encoded).

Other variables:

* `derived_key` is equal to `kdf(user_password, kdf_salt, kdf_length)`.
* `storage_secret` is equal to `sym_decrypt(cipher, secret, derived_key)`.

When a client application first wants to use Soledad, it must provide the user's password to unlock the `storage_secret`:

    from leap.soledad.client import Soledad
    sol = Soledad(
        uuid='<user_uid>',
        passphrase='<user_passphrase>',
        secrets_path='~/.config/leap/soledad/<user_uid>.secret',
        local_db_path='~/.config/leap/soledad/<user_uid>.db',
        server_url='https://<soledad_server_url>',
        cert_file='~/.config/leap/providers/<provider>/keys/ca/cacert.pem',
        auth_token='<auth_token>',
        secret_id='<secret_id>')  # optional argument


Currently, the `storage_secret` is shared among all devices with access to a particular user's Soledad database. See [Recovery and bootstrap](#Recovery.and.bootstrap) for how the `storage_secret` is initially installed on a device.

We don't use the `derived_key` as the `storage_secret` because we want the user to be able to change their password without needing to re-key.

Document encryption
------------------------

Before a JSON document is synced with the server, it is transformed into a document that looks like this:

    {
      "_enc_json": "<ciphertext>",
      "_enc_scheme": "symkey",
      "_enc_method": "aes256ctr",
      "_enc_iv": "<initialization_vector>",
      "_mac": "<auth_mac>",
      "_mac_method": "hmac"
    }

About these fields:

* `_enc_json`: The original JSON document, encrypted and hex encoded. Calculated as:
    * `doc_key = hmac(storage_secret[MAC_KEY_LENGTH:], doc_id)`
    * `ciphertext = hex(sym_encrypt(cipher, content, doc_key))`
* `_enc_scheme`: Information about the encryption scheme used to encrypt this document (i.e.`pubkey`, `symkey` or `none`).
* `_enc_method`: Information about the block cipher that is used to encrypt this document.
* `_mac`: A MAC to prevent the server from tampering with stored documents. Calculated as:
    * `mac_key = hmac(storage_secret[:MAC_KEY_LENGTH], doc_id)`
    * `_mac = hmac(doc_id|rev|ciphertext|_enc_scheme|_enc_method|_enc_iv, mac_key)`
* `_mac_method`: The method used to calculate the mac above (currently hmac).

Other variables:

* `doc_key`: This value is unique for every document and only kept in memory. We use `doc_key` instead of simply `storage_secret` in order to hinder possible derivation of `storage_secret` by the server. Every `doc_id` is unique.
* `content`: equal to `sym_decrypt(cipher, ciphertext, doc_key)`.

When receiving a document with the above structure from the server, Soledad client will first verify that `_mac` is correct, then decrypt the `_enc_json` to find `content`, which it saves as a cleartext document in the local encrypted database replica.

The document MAC includes the document revision and the client will refuse to download a new document if the document does not include a higher revision. In this way, the server cannot rollback a document to an older revision. The server also cannot delete a document, since document deletion is handled by removing the document contents, marking it as deleted, and incrementing the revision. However, a server can withhold from the client new documents and new revisions of a document (including withholding document deletion).

The currently supported encryption ciphers are AES256 (CTR mode) and XSalsa20. The currently supported MAC method is HMAC with SHA256.

Document synchronization
-----------------------------------

Soledad follows the U1DB synchronization protocol, with some changes:

* Add the ability to flag some documents so they are not synchronized by default (not fully supported yet).
* Refuse to synchronize a document if it is encrypted and the MAC is incorrect.
* Always use `https://<soledad_server_url>/user-<user_uid>` as the synchronization URL.


    doc = sol.create_doc({'some': 'data'})
    doc.syncable = False
    sol.sync()  # will not send the above document to the server!

Document IDs
--------------------

Like U1DB, Soledad allows the programmer to use whatever ID they choose for each document. However, it is best practice to let the library choose random IDs for each document so as to ensure you don't leak information. In other words, leave the second argument to `create_doc()` empty.

Re-keying
-----------

Sometimes there is a need to change the `storage_secret`. Rather then re-encrypt every document, Soledad implements a system called "lazy revocation" where a new `storage_secret` is generated and used for all subsequent encryption. The old `storage_secret` is still retained and used when decrypting older documents that have not yet been re-encrypted with the new `storage_secret`.

Authentication
-----------------------

Unlike U1DB, Soledad only supports token authentication and does not support OAuth. Soledad itself does not handle authentication. Instead, this job is handled by a thin HTTP WSGI middleware layer running in front of the Soledad server daemon, which retrieves valid tokens from a certain shared database and compares with the user-provided token. How the session token is obtained is beyond the scope of Soledad.

Bootstrap and recovery
------------------------------------------

Because documents stored on the server's database replica have their contents encrypted with keys based on the `storage_secret`, initial synchronizations of newly configured provider accounts are only possible if the secret is transferred from one device to another. Thus, installation of Soledad in a new device or account recovery after data loss is only possible if specific recovery data has previously been exported and either stored on the provider or imported on a new device.

Soledad may export a recovery document containing recovery data, which may be password-encrypted and stored in the server, or stored in a safe environment in order to be later imported into a new Soledad installation.

**Recovery document**

An example recovery document:

    {
        'storage_secrets': {
            '<secret_id>': {
                'kdf': 'scrypt',
                'kdf_salt': '<b64 repr of salt>'
                'kdf_length': <key length>
                'cipher': 'aes256',
                'length': <secret length>,
                'secret': '<encrypted storage_secret>',
            },
        },
        'kdf': 'scrypt',
        'kdf_salt': '<b64 repr of salt>',
        'kdf_length: <key length>,
        '_mac_method': 'hmac',
        '_mac': '<mac>'
    }

About these fields:

* `secret_id`: a handle used to refer to a particular `storage_secret` and equal to `sha256(storage_secret)`.
* `kdf`: the key derivation function to use. Only scrypt is currently supported.
* `kdf_salt`: the salt used in the kdf. The salt for scrypt is not random, but encodes important parameters like the limits for time and memory.
* `kdf_length`: the length of the derived key resulting from the kdf.
* `length`: the length of the secret.
* `secret`: the encrypted `storage_secret`.
* `cipher`: what cipher to use to encrypt `secret`. It must match `kdf_length` (i.e. the length of the `derived_key`).
* `_mac_method`: The method used to calculate the mac above (currently hmac).
* `_mac`: Defined as `hmac(doc_id|rev|ciphertext, doc_key)`. The purpose of this field is to prevent the server from tampering with the stored documents.

Currently, scrypt parameters are:

     N (CPU/memory cost parameter) = 2^14 = 16384
     p (paralelization parameter) = 1
     r (length of block mixed by SMix()) = 8
     dkLen (length of derived key) = 32 bytes = 256 bits

Other fields we might want to include in the future:

* `expires_on`: the month in which this recovery document should be purged from the database. The server may choose to purge documents before their expiration, but it should not let them linger after it.
* `soledad`: the encrypted `soledad.json`, created by `sym_encrypt(cipher, contents(soledad.json), derived_key)` (base64 encoded).
* `reset_token`: an optional encrypted password reset token, if supported by the server, created by `sym_encrypt(cipher, password_reset_token, derived_key)` (base64 encoded). The purpose of the reset token is to allow recovery using the recovery code even if the user has forgotten their password. It is only applicable if using recovery code method.

**Recovery database**

In order to support easy recovery, the Soledad client stores a recovery document in a special recovery database. This database is shared among all users.

The recovery database supports two functions:

* `get_doc(doc_id)`
* `put_doc(doc_id, recovery_document_content)`

Anyone may preform an unauthenticated `get_doc` request. To mitigate the potential attacks, the response to queries of the discovery database must have a long delay of X seconds. Also, the `doc_id` is very long (see below).

Although the database is shared, the user must authenticate via the normal means before they are allowed to put a recovery document. Because of this, a nefarious server might potentially record which user corresponds to which recovery documents. A well behaved server, however, will not retain this information. If the server supports authentication via blind signatures, then this will not be an issue.


**Recovery code (yet to be implemented)**

We intend to offer data recovery by specifying username and a recovery code. The choice of type of recovery (using password or a recovery code) must be made in advance of attempting recovery (e.g. at some point after the user has Soledad successfully running on a device).

About the optional recovery code:

* The recovery code should be randomly generated, at least 16 characters in length, and contain all lowercase letters (to make it sane to type into mobile devices).
* The recovery code is not stored by Soledad. When the user needs to bootstrap a new device, a new code is generated. To be used for actual recovery, a user will need to record their recovery code by printing it out or writing it down.
* The recovery code is independent of the password. In other words, if a recovery code is generated, then a user changes their password, the recovery code is still be sufficient to restore a user's account even if the user has lost the password. This feature is dependent on the server supporting a password reset token. Also, generating a new recovery code does not affect the password.
* When a new recovery code is created, and new recovery document must be pushed to the recovery database. A code should not be shown to the user before this happens.
* The recovery code expires when the recovery database record expires (see below).

The purpose of the recovery code is to prevent a compromised or nefarious Soledad service provider from decrypting a user's storage. The benefit of a recovery code over the user password is that the password has a greater opportunity to be compromised by the server. Even if authentication is performed via Secure Remote Password, the server may still perform a brute force attack to derive the password.

Reference implementation of client
===================================

https://github.com/leapcode/soledad

Dependencies:

* [U1DB](https://launchpad.net/u1db) provides an API and protocol for synchronized databases of JSON documents.
* [SQLCipher](http://sqlcipher.net/) provides a block-encrypted SQLite database used for local storage.
* python-gnupg
* scrypt
* pycryptopp

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
* Correctly representing and handling new Document properties (e.g. the `sync` flag).

Part of the Soledad `storage_key` is used directly as the key for the SQLCipher encryption layer. SQLCipher supports the use of a raw 256 bit keys if provided as a 64 character hex string. This will skip the key derivation step (PBKDF2), which is redundant in our case. For example:

    sqlite> PRAGMA key = "x'2DD29CA851E7B56E4697B0E1F08507293D761A05CE4D1B628663F411A8086D99'";

**Classes**

SQLCipher backend classes:

* `SQLCipherDatabase`: An extension of `SQLitePartialExpandDatabase` used by Soledad Client to store data locally using SQLCipher. It implements the following:
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

* `SoledadSyncTarget`: an extension of `HTTPSyncTarget` modified to encrypt documents' content before sending them to the network and to have more control of the syncing process.

Reference implementation of server
======================================================

https://github.com/leapcode/soledad

Dependencies:

* [CouchDB](https://couchdb.apache.org/] for server storage, via [python client library](https://pypi.python.org/pypi/CouchDB/0.8).
* [Twisted](http://twistedmatrix.com/trac/) to run the WSGI application.
* scrypt
* pycryptopp
* PyOpenSSL

CouchDB backend
-------------------------------

In the server side, Soledad stores its database replicas in CouchDB servers. Soledad's CouchDB backend implementation is built on top of U1DB's `CommonBackend`, and stores and fetches data using a remote CouchDB server. It lacks indexing first because we don't need that functionality on server side, but also because if not very well done, it could lack sensitive information about document's contents.

CouchDB backend is responsible for:

* Initializing and maintaining the following U1DB replica data in the database:
  * Transaction log.
  * Conflict log.
  * Synchronization log.
* Mapping the U1DB API to CouchDB API.

**Classes**

* `CouchDatabase`: A backend used by Soledad Server to store data in CouchDB.
* `CouchSyncTarget`: Just a target for syncing with Couch database.
* `CouchServerState`: Interface of the WSGI server with the CouchDB backend.

WSGI Server
-----------------------------------------

The U1DB server reference implementation provides for an HTTP API backed by SQLite databases. Soledad extends this with token-based auth HTTP access to CouchDB databases.

* Soledad makes use of `twistd` from Twisted API to serve its WSGI application.
* Authentication is done by means of a token.
* Soledad implements a WSGI middleware in server side that:
  * Uses the provided token to verify read and write access to each user's private databases and write access to the shared recovery database.
  * Allows reading from the shared remote recovery database.
  * Uses CouchDB as its backend.

**Classes**

* `SoledadAuthMiddleware`: implements the WSGI middleware with token based auth as described before.
* `SoledadApp`: The WSGI application. For now, not different from `u1db.remote.http_app.HTTPApp`.

**Authentication**

Soledad Server authentication middleware controls access to user's private databases and to the shared recovery database. Soledad client provides a token for Soledad server that can check the validity of this token for this user's session by querying a certain database.

A valid token for this user's session is required for:

* Read and write access to this user's database.
* Read and write access to the shared recovery database.

Tests
===================

To be sure the new implemented backends work correctly, we included in Soledad the U1DB tests that are relevant for the new pieces of code (backends, document, http(s) and sync tests). We also added specific tests to the new functionalities we are building.
