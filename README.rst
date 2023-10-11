TKLBAM hook script to work around an issue migrating MariaDB databases
======================================================================

It has been bought to my attention that migrating data via TKLBAM - from
v16.x or earlier will error when restoring the databases. The specific error
message is::

    ERROR 1050 (42S01) at line xxx: Table 'user' already exists

After a significant amount of googling and trial & error (and hand wringing
and gnashing of teeth) I discovered the cause of the issue. Namely `this
MariaDB bug`_. Namely, that from MariaDB v10.4 (TKL v16 has 10.3; v17 has
10.5) the mysql.user table is now a view (historically was a table).

The work around is to drop both the mysql.global_priv table (the new table
that holds MariaDB/MySQL user data) and the mysql.user view, load the backup
database and fix the permissions (so both root & mysql MariaDB/MySQL user can
log in via unix_socket) and force a DB upgrade (which will generate a new
mysql.global_priv table and mysql.user view.

All this is handled by the maria_db-changes TKLBAM hook script (in this repo).

How to install
--------------

My intention is to include into the relevant TKLBAM profiles, so that manually
installing this hook script wouldn't be neccessary. In the meantime, you need
to manually download and enable the script. Do that like this (running as root)
::

    URL=https://raw.githubusercontent.com/JedMeister/tklbam-restore-mariadb-bugfix/master/maria_db-changes
    wget -O /etc/tklbam/hooks.d/maria_db-changes $URL
    chmod +x /etc/tklbam/hooks.d/maria_db-changes

Usage
-----

Nothing special is required to use the script. It should auto detect if it's
required and only run when needed by 'tklbam-restore'.

Bug reports
-----------

Please report any issues relating to use of this hook script; either in our
forums_ (requires free website user account - please note that automated
account approval is disabled - please follow `instructions to get approved`_)
or as a `GitHub Issue`_ (on our consolidated issue tracker).


.. _this MariaDB bug: https://jira.mariadb.org/browse/MDEV-22127
.. _forums: https://www.turnkeylinux.org/forum/support
.. _instructions to get approved: https://www.turnkeylinux.org/forum/general/tue-20230418-1616/post-thread-if-you-are-awaiting-account-approval
.. _GitHub Issue: https://github.com/turnkeylinux/tracker/issues
