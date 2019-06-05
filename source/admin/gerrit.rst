Gerrit
======

Upgrades
--------

2.14 -> 2.15
------------

The official documentation for 2.16_ release requests to upgrade to 2.15 first if the currently
deployed version is an older one:

  *When a gerrit site older than 2.15 is migrated to 2.16, then schema migration should be first
  performed to the latest 2.15.x release.*

Pre-migration steps
*******************

It is best to have a backup of Gerrits home / site directory and of the databases Gerrit uses.

On Gerrit node backup gerrit users home directory (as root)

  .. code:: bash

    $ rsync -az /home/gerrit2 /root/
    $ tar zcvf /root/gerrit2.tgz /root/gerrit2

  Copy the resulting /root/gerrit2.tgz to a remote location.

On the database node backup gerrits databases:

   * mysqldump reviewdb > reviewdb-dump.sql
   * mysqldump accountPatchReviewDb > accountPatchReviewDb-dump.sql

Migration steps
***************

Perform the following to upgrade gerrit to a new version (all actions on gerrit node):

Stop gerrit on root user:

  .. code:: bash

    $ /home/gerrit2/review_site/bin/gerrit.sh stop

On gerrit user:

#. Download the new 2.15_ binary:

  .. code:: bash

    $ wget https://gerrit-releases.storage.googleapis.com/gerrit-2.15.13.war

#. Perform the migration:

  .. code:: bash

    $ java -jar gerrit-2.15.13.war init -d /home/gerrit2/review_site --batch --no-auto-start

      During this step if you encounter a migration issue like below, perform the steps described
      in `MySQL migration issues`_.

        *Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Invalid default
        value for 'last_updated_on'*

#. Upgrade delete-project plugin:

 .. code:: bash

    $ ssh -p 29418 -i id_rsa -l gerrit localhost gerrit plugin rm delete-project
    $ rm /home/gerrit2/review_site/plugins/delete-project.jar.disabled
    $ ssh -p 29418 -i id_rsa -l gerrit localhost gerrit plugin install -n delete-project.jar https://gerrit-ci.gerritforge.com/view/Plugins-stable-2.15/job/plugin-delete-project-bazel-stable-2.15/lastSuccessfulBuild/artifact/bazel-genfiles/plugins/delete-project/delete-project.jar

Using root user start the service:

  .. code:: bash

    $ /home/gerrit2/review_site/bin/gerrit.sh start

The output from the migration step suggests to drop the unneeded tables in gerrits reviewdb database.
On the database node after logging in to mysql issue:

  .. code:: bash

    mysql> use reviewdb;
    mysql> DROP TABLE account_external_ids;
    mysql> DROP TABLE accounts;
    mysql> ALTER TABLE patch_sets DROP COLUMN draft;

MySQL migration issues
**********************

On the database node reconfigure mysql. Adjust the /etc/mysql/mysql.cnf file to contain the following
configuration and restart the service.

  .. code:: bash

    [mysqld]
    sql_mode = "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

2.15 -> 2.16
------------


.. _2.16: https://www.gerritcodereview.com/2.16.html
.. _2.15: https://www.gerritcodereview.com/2.15.html
