Webapp node
===========

Places to look for errors
-------------------------

* `/var/log/apache2/error.log`
* `/srv/leap/webapp/log/production.log`
* `/var/log/syslog` (watch out for stunnel issues)

Is haproxy ok ? 
---------------


    curl -X  GET "http://127.0.0.1:4096"

Is couchdb accessible through stunnel ?
---------------------------------------


    curl -X  GET "http://127.0.0.1:4000"


Check couchdb acl
-----------------


    mkdir /etc/couchdb
    cat /srv/leap/webapp/config/couchdb.yml.admin  # see username and password
    echo "machine 127.0.0.1 login admin password <PASSWORD>" > /etc/couchdb/couchdb-admin.netrc
    chmod 600 /etc/couchdb/couchdb-admin.netrc

    curl --netrc-file /etc/couchdb/couchdb-admin.netrc -X GET "http://127.0.0.1:4096"
    curl --netrc-file /etc/couchdb/couchdb-admin.netrc -X GET "http://127.0.0.1:4096/_all_dbs"
    

Couchdb node
============

Places to look for errors
-------------------------

* `/opt/bigcouch/var/log/bigcouch.log`
* `/var/log/syslog` (watch out for stunnel issues)


Bigcouch membership
-------------------

* Do all nodes show up both in and ?


    curl --netrc-file /etc/couchdb/couchdb.netrc 'http://127.0.0.1:5984/_membership'



Databases
---------

Design Documents
----------------

* Is User `_design doc` available ?


    curl --netrc-file /etc/couchdb/couchdb.netrc -X  GET "http://127.0.0.1:5984/users/_design/User"



MX node
=======

Query leap-mx
-------------

* for useraccount 


    postmap -v -q  "joe@dev.bitmask.net" tcp:localhost:2244


* for mailalias


    postmap -v -q  "joe@dev.bitmask.net" tcp:localhost:4242


