Repositories and Registries
===========================

ci-repo.englab.juniper.net hosts the following components:
* per-review docker registries
* nightly docker registries
* mirrors of centos and docker upstream repos setup (pulp)

Operational Tasks
-----------------

Per-review docker registries cleanup
************************************

The per-review docker registries are cleaned up periodically using `artifact_curator.py
<https://github.com/Juniper/contrail-infra/blob/production/modules/opencontrail_ci/manifests/pulp_server.pp#L125>`_
script. The script removes only per-review registries which are not part of a running buildset.

The script still contains the logic to also remove per-review RPM pulp repos. This is legacy code
from back when per-review RPM repos were still being setup on pulp - currently Sonatype Nexus is
being used for this.

Nightly docker registries cleanup
*********************************

Caution: restarting registry container during this procedure may cause some jobs to fail:

* sanity (downloading kolla images)
* nightly publish (publishing nightly images)

1.  Find the registry container name

  .. code:: bash

    docker ps -a | grep 5000

2. Make sure that the tag removal API is enabled in the container

  * exec into the container:

    .. code:: bash

       docker exec -it registry /bin/sh

  * modify the /etc/docker/registry/config.yml to have the following entry:

    .. code:: bash

       storage:
         delete:
           enabled: true

  * if the entry was not present restart the container:

    .. code:: bash

       docker stop registry && docker start registry

3. Remove the tags from the registry (make sure there are no running nightly builds):

  * enable virtualenv:

    .. code:: bash

       cd /root/docker_tagtool && source venv/bin/activate

  * generate the list of existing tags:

    .. code:: bash

       python docker_tagtool.py list_tags > tags

  * edit the tag list and **leave only the tags that you want to delete**

    Alternatively you can use a script to generate a list of tags to remove located
    `here <https://github.com/tungsten-infra/ci-utils/tree/master/tungsten_ci_utils/dockerregistry_cleanup>`_.

  * remove the tags:

    .. code:: bash

       cat tags | while read tag; do python docker_tagtool.py --dry-run "" --tag $tag remove_tag_from_registry; done

  * prune the unused docker volumes

    .. code:: bash

       docker volume prune

4. Switch the registry to read-only mode:

  * add this entry to /etc/docker/registry/config.yml in the container:

    .. code:: bash

       storage:
         maintenance:
           readonly:
             enabled: true

  * restart the registry container

5. Run garbage collection:

    .. code:: bash

       docker exec -it registry bin/registry garbage-collect /etc/docker/registry/config.yml

5. Re-enable write mode, reverting changes done in step 4.

TPC sync
********

There are two third party caches:

* internal - RPM repository set up on Nexus. Packages are uploaded here after being built after every
  merge to contrail-third-party-packages_
* external - RPM repository set up using `createrepo` directly. Packages are synced here every hour
  from the internal TPC. This is done through a cron job set up on ci-repo.englab.juniper.net at
  /etc/cron.hourly. The scripts used are version controlled in ci-utils_

RPM CentOS mirrors
------------------

Pulp server serves mirrors of upstream RPM repos:

* centos74
* centos74-updates
* centos74-extras
* epel
* docker

Why not Nexus?
**************

It was decided by the client that these repos should be updated manually (that is: synchronization
with upstream should be manually triggered). Nexus might not support disabling auto sync.

Example Command Reference
*************************

* check status

  .. code:: bash

     pulp-admin status

* list rpm repos

  .. code:: bash

     pulp-admin rpm repo list

* show upstream synchronisation schedule for a specific repo

  .. code:: bash

     pulp-admin rpm repo sync schedules list --repo-id <repo_id>

RPM RedHat mirrors
------------------

RedHat upstream mirrors are hosted on ci-rhel-mirror.englab.juniper.net. The mirrors are synchronised
using `reposync` and created by running `createrepo`. The sync_ bash script is run in a cronjob
configured for the `root` user (located at /opt/sync.sh).

Repository proxy
----------------


.. _contrail-third-party-packages: https://github.com/Juniper/contrail-third-party-packages
.. _ci-utils: https://github.com/tungsten-infra/ci-utils/reposync
.. _sync: https://github.com/tungsten-infra/ci-utils/tungsten_ci_utils/rh-reposync/sync.sh
