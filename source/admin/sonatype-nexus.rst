Sonatype Nexus
==============

`Sonatype Nexus <https://www.sonatype.com/nexus-repository-sonatype>`_, with the admin interface available at http://ci-nexus.englab.juniper.net/, hosts:

* RPM repositories

  * per-review
  * nightly
  * third-party cache (TPC)
  * CentOS upstream mirrors (in future)

* docker image registries (in future)

  * per-review (in future)
  * nightly (in future)


RPM Repositories
----------------

The general idea behind different RPM repos hosted on Nexus should be written here, how are they
configured (e.g. why the particular repo depth etc.).

Per-review (yum-tungsten)
*************************

A code patchset opened on Gerrit (for example ``47216,2`` or ``50538,7``) often eventually leads CI to run on it jobs named contrail-vnc-build-package_.

These jobs create for themselves a dedicated target RPM repository (example repositories: ``47216-2-centos`` or ``50538-7-rhel-queens``), placing it
underneath main repository `yum-tungsten`.

The main repository `yum-tungsten` has a repodata depth of '1'. Also, it allows for package redeploys, since a 'recheck' Gerrit comment would lead
to re-uploading to the same repository.

Nightly (yum-tungsten-nightly)
******************************

The repository `yum-tungsten-nightly` is similar to the per-review one. It also has a
repodata depth of '1', but it's purpose is storing packages built by the contrail-vnc-build-package_ job run
in nightly builds (periodic-nightly pipeline). There is a separate RPM repo created for each new
nightly run, with the repo names incorporating both the branch and build number.

The repository does not allow for package redeploys, as a form of protection from introducing an
error into the system, which would cause repository name reusal in consecutive nightly runs.

Third Party Cache (yum-tungsten-tpc)
************************************

The TPC repository (named `yum-tungsten-tpc`) hosts packages of two origins:

* .rpm files which we can build from source code (referred to as 'source'; can be re-build from
  contrail-third-party-packages_ repo)
* .rpm files which we only obtained in binary form (referred to as 'binary'; we do not have any source
  code to re-build these)

This Nexus repository has a repodata depth '2' to enable hosting 'source'/'binary' RPM repos for each
of the release branches (currently: master, R5.0, R5.1).

Uploading RPMs manually
***********************

To upload new RPMs to the Nexus server use ``curl`` command.

Within the URL you need to provide the proper branch (master, R5.0 or R5.1), the type ('binary' or 'source'), and the name of RPM.

Example:

.. code:: bash

  curl -u 'user:pass' -v --upload-file new.rpm http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/master/source/new.rpm

Maven Repositories
------------------

The 'maven-releases' repository, grouped under 'maven-public' repository is a mirror of vco-repo_.
This maven repository serves maven dependencies for e.g. contrail-vro-plugin_.
The procedure for mirroring is available at :doc:`maven-mirror`.

.. _contrail-third-party-packages: https://github.com/Juniper/contrail-third-party-packages
.. _vco-repo: https://sdnpoc-vrodev.englab.juniper.net:8281/vco-repo/
.. _contrail-vro-plugin: https://github.com/Juniper/contrail-vro-plugin/blob/master/playbooks/contrail-build-vro-plugin/run.yaml#L17
.. _contrail-vnc-build-package: https://github.com/Juniper/contrail-zuul-jobs/blob/master/zuul.d/contrail-jobs.yaml#L4
