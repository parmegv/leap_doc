@title = 'Tapicero'
@summary = 'Creating per-user databases on the couch for soledad.'
@toc = true

Tapicero 
==============

**Create databases for the leap platform users**


Tapicero is part of the leap platform. It's deployed to the couch nodes and watches the users database as a daemon. When a user is add it creates a new database for that user. It also removes these databases on user destruction. This way neither the webapp nor soledad need couch admin privileges.

"Tapicero" is spanish for upholsterer - the person who creates your couch.

Running
--------------------

Tapicero is usually deployed with the leap platform and run as a daemon from an init script. It also serves as a tool to modify existing user databases. You can find it in `/srv/leap/tapicero` on the couch nodes or play with it on your own machine.

Run in foreground:

    bundle exec /bin/tapicero run

Run as a deamon:

    bundle exec /bin/tapicero start
    bundle exec /bin/tapicero stop

Run once, process all changes so far and then exit:

    bundle exec tapicero --run-once

Configuration
---------------------

Tapicero reads the following configurations files, in this order:

* ``$(tapicero_source)/config/default.yaml``
* ``/etc/leap/tapicero.yaml``
* Any file passed to ARGV like so ``tapicero start -- /etc/tapicero.yaml``

Files that come later will overwrite settings from the former.

### Sequence File

Tapicero keeps track of the last change processed in a sequence file. The location of the sequence file is configured as `seq_file` and defaults to `/var/log/leap/tapicero.seq`

After restarting Tapicero it will only process changes that happened after the change with the sequence id given in the sequence file. This behaviour can be altered by using the --rerun flag or removing the sequence file.

### Logging

Tapicero logs it's activity to syslog in a production environment. Logging details can be configured via `log_level`
Configure `log_file` if you want to log to a file instead of syslog.

Flags
---------------------

--run-once:
  process the existing users and then exit

--rerun:
  also work on users that have been processed before

--overwrite-security:
  write the security settings even if the user database already has some

Combining these flags you can migrate the security settings of all existing per user databases.


Installation
---------------------

Tapicero is normally deployed as part of the leap platform. If you want to install it outside of this context these instructions are for you.

Prerequisites:

    sudo apt-get install ruby ruby-dev couchdb
    # for development, you will also need git, bundle, and rake.

From source:

    git clone git://leap.se/tapicero
    cd tapicero
    bundle
    bundle exec bin/tapicero {run, start, status, ...}

From gem:

    sudo gem install tapicero

License
--------

This program is written in Ruby and is distributed under the following license:

> GNU Affero General Public License
> Version 3.0 or higher
> http://www.gnu.org/licenses/agpl-3.0.html
