Repositories and Registries
===========================

ci-repo.englab.juniper.net hosts the following components:
* per-review docker registries
* nightly docker registries
* mirrors of centos and docker upstream repos setup (pulp)

Operational Tasks
=================

Per-review docker registries cleanup
------------------------------------

The per-review docker registries are cleaned up periodically using `artifact_curator.py
<https://github.com/Juniper/contrail-infra/blob/production/modules/opencontrail_ci/manifests/pulp_server.pp#L125>`_
script. The script removes only per-review registries which are not part of a running buildset.

Nightly docker registries cleanup
---------------------------------

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
    `here <https://github.com/tungsten-infra/ci-utils/tree/master/tungsten_ci_utils/nightly_images_cleanup>`_.

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

RPM mirrors
-----------

Pulp server serves mirrors of upstream RPM repos:

* centos74
* centos74-updates
* centos74-extras
* epel
* docker

It was decided by the client that these repos should be updated manually (that is: synchronization
with upstream should be manually triggered). They should be in the end (and if possible) moved to Nexus
(Nexus might not support disabling auto sync).
