Tapicero - Creating per user databases on the couch for soledad
------------------------------------------------------------

``tapicero`` is a daemon that creates per user databases when users are added to the LEAP Platform. It watches the changes made to the users database and creates new databases accordingly. This way soledad does not need admin privileges.

"Tapicero" is spanish for upholsterer - the person who creates your couch.

This program is written in Ruby and is distributed under the following license:

> GNU Affero General Public License
> Version 3.0 or higher
> http://www.gnu.org/licenses/agpl-3.0.html

Installation
---------------------

Prerequisites:

    sudo apt-get install ruby ruby-dev couchdb
    # for development, you will also need git, bundle, and rake.

From source:

    git clone git://leap.se/tapicero
    cd tapicero
    bundle
    rake build
    sudo rake install

From gem:

    sudo gem install tapicero

Running
--------------------

Run in foreground to see if it works:

    tapicero run -- test/config/config.yaml
    create a new record in the users database
    observe /var/log/syslog or the logfile you specified

Run as a deamon:

    tapicero start
    tapicero stop

Run once and then exit:

    tapicero --run-once
    This will create per user databases for all users created since
    the last run and then exit.

Flags
---------------------

--run-once:
  process the existing users and then exit

--rerun:
  also work on users that have been processed before

--overwrite-security:
  write the security settings even if the user database already has some

Combining these flags you can migrate the security settings of all existing per user databases.


Configuration
---------------------

``tapicero`` reads the following configurations files, in this order:

* ``$(tapicero_source)/config/default.yaml``
* ``/etc/leap/tapicero.yaml``
* Any file passed to ARGV like so ``tapicero start -- /etc/tapicero.yaml
