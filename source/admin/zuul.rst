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

Autoholding VMs
---------------

VMs which are used to run jobs can be marked to be held (meaning: to not be deleted after a job finishes).
For the current Zuul installation this can only be done when a job fails in the run playbook
so a VM will NOT be held if it has a status other than FAILED or it only failed in a pre or post
playbook. The VM will also not be held if it failed in run AND a post playbook.

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
