Zuul
====

The following sections explain the more tricky operational tasks and topics in relation to
administering and/or using Zuul.

Adding new project
------------------

This procedure also involves Gerrit:

1. Make sure the project is created on Gerrit.

  A. If it's not a new project (was hosted on GitHub previously), then make sure it's state on
     Gerrit is the same as on GitHub.
  B. If it's a new project, make sure that the repository contains at least one commit, Zuul will
     fail to clone it otherwise.

2. Add the project to Zuul's known projects in project config zuul/main.yaml.
3. Apply the 'noops' template for the new project in project config zuul.d/projects.yaml.
4. Run puppet on Zuulv3 to deploy the newest configuration (verify at /etc/zuul/layouts/main.yaml).
5. Reload the zuul-scheduler service to read the new config

  .. code:: bash

    $ systemctl reload zuul-scheduler

6. Make sure Gerrit replication works for the project.

  A. If commits are not being replicated and there is no mention of the project in the replication.log
     then most likely the replication config is invalid (check $GERRIT_HOME/review_site/etc/replication.config).

Logging on the builder VMs
--------------------------

You can login to the auto-spawned VM while it is running a zuul job. But expect to be
logged out at any moment: the VM is automatically deleted after the job finishes
(unless caught by autohold as described in the following sections).

First you need a Node ID, from which you get'll the IP address and ssh to it.

Filter the scheduler’s log:

  .. code:: bash

    $ ssh zuulv3             # scheduler
    $ grep Execute /var/log/zuul/zuul.log | less -S

    Execute job contrail-vnc-build-package-centos74 (uuid: b37ecc4231a1400fb1d35b7fb996f970) on nodes <NodeSet OrderedDict([('builder', <Node 0000247169 builder:centos-7-4-builder>)])OrderedDict()> for change <Change 0x7fe2111cbbe0 51790,1> with ...
                                                                   scroll ----->    scroll ----->                                     ----->  ^^^^^^^^^^  <-----

This message appears in the log just before the job starts.
In this example Node ID is ``0000247169`` (and the originating gerrit changeset is ``51790``).
Having obtained it, now login to machine that has ``nodepool`` utility and access to Zookeeper:

  .. code:: bash

    $ ssh nl02-jnpr          # nodepool launcher
    $ sudo -u nodepool -i
    $ nodepool list --detail

Use the obtained Node ID to find the corresponding public IP address.
Now login to the executor:

  .. code:: bash

    $ ssh ze03-jnpr          # executor
    $ sudo ssh -i /var/lib/zuul/ssh/id_rsa zuul@<node_public_ip>

You are now on the host that is a client to the actual ansible playbooks
governed by zuul.

Autoholding VMs
---------------

VMs which are used to run jobs can be marked to be held (meaning: to not be deleted after a job finishes).
For the current Zuul installation this can only be done when a job fails in the ``run`` playbook.
A VM `will` be still deleted if:

- a job has a status other than `FAILED`
- or it failed in a ``pre-run`` playbook
- or it failed in a ``post-run`` playbook
- or it failed in both ``run`` and ``post-run`` playbook

To hold a node perform the following:

1. Determine the review id e.g. for https://review.opencontrail.org/#/c/12345/ it will be 12345.
2. SSH into the zuul-scheduler node and execute:

  .. code:: bash

    $ zuul autohold --tenant opencontrail --project <project> --job <job_name> --change <review_id> \
        --reason <string> --count 1

  NOTE: include your name/login in the reason to make it possible to determine which nodes are held
  by whom. To see the active autohold list, issue:

  .. code:: bash

    $ zuul autohold-list

3. After the job finishes, you can find the IP of the worker node in Zuul's inventory file, available
   in the job logs. Example `link <http://logs.opencontrail.org/31/51231/1/check/contrail-sanity-centos7-k8s/27e7009/zuul-info/inventory.yaml>`_.

4. SSH into the VM from the executor node:

  .. code:: bash

    $ ssh -i /var/lib/zuul/ssh/id_rsa zuul@<ip>

Deleting a held VM
******************

Autoheld VMs need to be removed manually:

1. SSH into the nodepool node and execute:

  .. code:: bash

    $ sudo -i -u nodepool
    $ nodepool list --detail | grep <ip_address>

  Where ip_address is the address of the VM you no longer need.

2. Delete the VM using the VM ID from the first column of the output from the last step:

  .. code:: bash

    $ nodepool delete <ID>

Managing secrets
----------------

Encryption
**********

To encrypt a single secret value perform the following:

1. SSH into the zuul-scheduler node.
2. Create a plaintext file for encryption. Make sure the file does not contain a new line at the end
   (vim automatically appends newline characters at the end). Example:

  .. code:: bash

    $ cat > /tmp/plaintext

  Exit `cat` mode by hitting Ctrl-d Ctrl-d.

  Another way is to use vim in binary mode and setting the 'noeol' option:

  .. code:: bash

    $ vim -b /tmp/plaintext
    (vim) :set noeol
    (vim) :wq

3. Encrypt the secret:

  .. code:: bash

    $ /opt/zuul/tools/encrypt_secret.py http://zuulv3.opencontrail.org/ gerrit \
      Juniper/contrail-project-config --infile=/tmp/plaintext --outfile=encrypted.yaml
    $ rm /tmp/plaintext

  You can now place the encrypted secret in the zuul.d/secrets.yaml file in project config.

Decryption
**********

1. SSH to the zuul-scheduler node and create a file with the encrypted secret (e.g. /tmp/cyphertext).
2. Remove the leading spaces from the cyphertext.
3. Execute:

  .. code:: bash

    $ base64 -d /tmp/cyphertext > /tmp/secret.bin
    $ openssl rsautl -inkey /var/lib/zuul/keys/gerrit/Juniper/contrail-project-config.pem -decrypt \
        -oaep -in /tmp/secret.bin

Alternatively use this one-liner after step 1:

  .. code:: bash

    $ cat /tmp/cyphertext | sed -rn 's/^ {8}[- ] *//;/^[^-][^ ]/p' | base64 -d | openssl rsautl \
        -inkey /var/lib/zuul/keys/gerrit/Juniper/contrail-project-config.pem -decrypt -oaep

zuul_return
-----------

zuul_return is a way for passing data down to the dependent jobs. Zuul executors automatically load
the data returned by zuul_return in dependent jobs. So if job B is dependent on job A and job A
returns data with zuul_return, that data can be used in job B.

If you'd want to pass data between playbook in the same job e.g. use variables defined in a pre-playbook
in the run playbook, you'll need to first load the saved results json file using the
`zuul-include-vars <https://github.com/Juniper/contrail-zuul-jobs/tree/master/roles/zuul-include-vars>`_
role.

Restarting Zuul
---------------

Restarting Zuul aborts all of the running jobs. It is advisable to save the information on
the running jobs on zuulv3.opencontrail.org and to requeue them after the restart is done:

.. code:: bash

   /root/zuul_restarts/dump_queues.sh
   ls -lart

Verify that `queue_<TIMESTAMP>.txt` file with a fresh timestamp has been created.

.. code:: bash

   systemctl restart zuul-scheduler
   less /var/log/zuul/zuul.log

The restart should be attended, because it frequently fails. You should see at first log message saying `Starting scheduler`.
Then many messages mentioning `Merge`. If you see any `Exception` or `ERROR` in the meantime, it probably means the restart will not be successful
and so consider reattempting the restart immediately. The `Merge` messages will continue whether or not restart can succeed - don't be confused by that.

If the `Merge` activity completed (there are only two TCP connections to gerrit, nothing is being git-downloaded) and nothing else happens for 30 seconds or more, restart
will probably not succeed and your should consider reattempting it immediately.

Restart can be considered successful after this line: `zuul.Pipeline.opencontrail.check: Configured Pipeline Manager check`

After Zuul is correctly started, re-launch the old jobs from the beginning:

.. code:: bash

   systemctl restart zuul-scheduler
   source queue_TIMESTAMP.txt

Go to the zuul's status page and verify the progress of the jobs, then clean up:

.. code:: bash

   rm queue_TIMESTAMP.txt


Aborting Builds
---------------

In the currently deployed Zuul version there is no way to abort Zuul buildsets with a push of a button.
At least Zuul-wise there is no clean way to do it. For this purpose a Python script was written which,
given a buildset ID (or a branch name in case of nightly runs), aborts the matching, running buildset.
In short, the script kills system processes which are associated to the buildset.

Aborting changeset-related builds the Zuul way
**********************************************

Suppose you have a review open and a `check` pipeline buildset is running for that. The running
buildset will be aborted when you submit a new patchset to the review. But Zuul will also enqueue
a new buildset for the latest patchset.

If you want Zuul not to start any new buildset for a review, you can submit a patchset with an
invalid Zuul configuration (.zuul.yaml file in the project root directory) e.g.:

.. code:: bash

  - invalid_zuul_configuration

Aborting any builds
*******************

To execute this you will need:

* the IP addresses of the Zuul executors
* the SSH key to log in to the executors to `zuul` user

The procedure is the following:

#. Check out the ci_utils_ repository. The relevant scripts are in tungsten_ci_utils/zuul_abort.
#. Create a `config.yaml` file according to the given `config.yaml.template` template file.
#. If you want to abort a nightly build execute:

  .. code:: bash

    $ python kill_buildset.py --forever R5.1

#. If you want to abort a running changeset-related buildset, pass in the buildset ID instead of
   the branch name. See section `Retrieving a buildset ID`_.

  .. code:: bash

    $ python kill_buildset.py --forever 5af9952dadf54457800aed96b3da8f61

NOTE: The repository and a proper `config.yaml` are already present on ci-repo.englab.juniper.net
in /root/ci-utils/tungsten_ci_utils/zuul_abort directory. You can run the script directly from
there.

NOTE2: Keep in mind that if you abort a job when it is running its pre-playbooks, the job will be
requeued up to 2 times. This will most likely extend the time it takes to abort the whole buildset,
due to the time it takes to wait for a new machine to be spawned for the next retry of the job.

Retrieving a buildset ID
************************

There are a number of ways to retrieve a running buildset ID:

#. If at least one job in the buildset already finished:

  * Open the job link from zuulv3.opencontrail.org.
  * Open the zuul-info/inventory.yaml file.
  * Search for `buildset`. The value of the key will be the ID.

#. If no jobs finished:

  * Open https://zuulv3.opencontrail.org/status.json (you'll probably want to open it in a browser
    which allow to comfortably examine JSON data; or use a CLI tool or an online site).
  * Search through the JSON, looking for data matching your running buildset.

    * First choose the right `pipeline`.
    * Then search through the running jobs data and match on the `id` value, which is the review ID
      and the patchset number.
    * The `zuul_ref` value is the buildset ID prefixed with a 'Z' e.g. `Z248eae43105144a6bb7b70d56c58e664`.
      Thus for this exemplary zuul_ref you would be interested in the `248eae43105144a6bb7b70d56c58e664`
      value.

Streaming Job Execution Logs
----------------------------

There are two possible ways to view running job output:

1. Through a web browser:

  A. Open http://zuulv3.opencontrail.org.
  B. Find your running buildset and expand it to view all the jobs.
  C. Click on the desired job link (must be running) to open a stream of the logs.

2. Using telnet on zuulv3.opencontrail.org, if for some reason you're not getting any stream logs
   from a running job in the browser. You'll need the UUID of the running build for this, which
   is present in the link mentioned in step 1C. Example link:

     .. code:: bash

       http://zuulv3.opencontrail.org/static/stream.html?uuid=66da8c82e8fc4304865a9ba3bc1fe7f5&logfile=console.log

   So the UUID is `66da8c82e8fc4304865a9ba3bc1fe7f5`.

  A. SSH to zuulv3.opencontrail.org.
  B. Execute the following:

    .. code:: bash

      $ telnet localhost 7900

  C. After the connection is setup just paste the UUID and press `enter`.

  NOTE: The 7900 is a reversely forwarded connection from ze01-jnpr.opencontrail.org. The job might
  not be running on this particular executor so if you get disconnected after pasting the UUID, try
  connecting to ports 7901-7903 (which are tunnels to other executors).


Nodepool Builder
----------------

Images used for spawning nodepool VMs are rebuilt every day by the nodepool builder service, which
is a daemon which uses disk-image-builder_. This approach ensures disk image builder code is current
and the functionality to build the base images for CI is always available.

Nodepool builder holds the last two successfully built images, rotating them with each new
successful build. To list the built images and their age perform the following:

#. SSH into the nodepool builder node.
#. List the images after logging in to nodepool user:

  .. code:: bash

    $ sudo -i -u nodepool
    nodepool@nb01:~$ nodepool dib-image-list

If the age of any image indicates it's older than 1 day, then there's something wrong with rebuilding
the image and it needs to be investigated.

.. _ci_utils: https://github.com/tungsten-infra/ci-utils
.. _disk-image-builder: https://docs.openstack.org/diskimage-builder/latest/

Manual Image Builds
*******************

For troubleshooting purposes it might be necessary to mimic nodepools building logic manually.
Prior to running disk image builder, nodepool builder sets up the environment. The variables and
values for a particular image can be found in nodepool.yaml_ configuration file. Suppose we would
want to build an ubuntu image like nodepool does it:

#. Prepare an environment file and source it. You'd want to use the environment variables and their
   values which nodepool uses e.g.:

  .. code:: bash

    export DIB_DISTRIBUTION_MIRROR=http://10.84.5.38:8000/ubuntu
    export DIB_DISABLE_APT_CLEANUP=1
    export DIB_RELEASE=xenial
    export DIB_IMAGE_CACHE=/opt/dib_cache
    export DIB_APT_LOCAL_CACHE=0
    export DIB_DEBIAN_COMPONENTS=main,universe
    export DIB_CHECKSUM=1
    export DIB_DEBOOTSTRAP_EXTRA_ARGS=--no-check-gpg
    export DIB_GRUB_TIMEOUT=0
    export ELEMENTS_PATH=/etc/nodepool/elements

#. Execute:

  .. code:: bash

    $ mkdir /tmp/ubuntu-xenial-manual
    $ disk-image-create -x -t qcow2 --checksum --no-tmpfs --qemu-img-options 'compat=0.10' \
      -o /tmp/ubuntu-xenial-manual/image ubuntu vm growroot pip-and-virtualenv tox \
      nodepool-base contrail-builder

The image will be available in the temporary directory.

.. _nodepool.yaml: https://github.com/Juniper/contrail-project-config/blob/master/nodepool/nodepool.yaml
