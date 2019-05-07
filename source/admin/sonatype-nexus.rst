Overview
========

`Sonatype Nexus <https://www.sonatype.com/nexus-repository-sonatype>`_ is used to host
per-review, nightly and TPC RPM repositories and - in the future - will be used as a per-review
and nightly docker image hosting and for hosting CentOS RPM mirrors.

The admin interface is available under: http://ci-nexus.englab.juniper.net/

RPM Repositories
----------------

The general idea behind different RPM repos hosted on Nexus should be written here, how are they
configured (e.g. why the particular repo depth etc.).

Per-review
**********

This repository (named: yum-tungsten) has a repodata depth of '1'. All actual RPM repos hosted here
store RPMs uploaded in the packaging_ job run in CI (check pipeline). There is a separate RPM repo
created for each changeset published to Gerrit (example repo names: 47216-2-centos, 50538-7-rhel-queens).

The repository allows for package redeploys, since a 'recheck' issued on a review would use the same
repository name.

Nightly
*******

This repository (named: yum-tungsten-nightly) is similar to the per-review one. It also has a
repodata depth of '1' but it's purpose is storing packages built by the packaging_ job run
in nightly builds (periodic-nightly pipeline). There is a separate RPM repo created for each new
nightly run, with the repo names incorporating both the branch and build number.

The repository does not allow for package redeploys, as a form of protection from introducing an
error into the system, which would cause repository name reusal in consecutive nightly runs.

Third Party Cache
*****************

The TPC repository (named 'yum-tungsten-tpc') is used to host third party RPMs of two origins:

* ones which we can build from source code (referred to as 'source'; can be re-build from
  contrail-third-party-packages_ repo.
* ones, for which we only have binary .rpm files (referred to as 'binary'; we do not have any source
  code to re-build these)

This Nexus repository has a repodata depth '2' to enable hosting 'source'/'binary' RPM repos for each
of the release branches (currently: master, R5.0, R5.1).

Uploading RPMs
**************

To upload new RPM's to the Nexus server use curl command.

Remember to provide the proper branch (master, R5.0 or R5.1), type ('binary' or 'source') and name of RPM in the URL.

Example:

.. code:: bash

  curl -u 'user:pass' -v --upload-file new.rpm http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/master/source/new.rpm


Maven Repositories
------------------

The 'maven-releases' repository, grouped under 'maven-public' repository is a mirror of vco-repo_.
This maven repository serves maven dependencies for e.g. contrail-vro-plugin_.
The procedure for mirroring is available here_.

.. _contrail-third-party-packages: https://github.com/Juniper/contrail-third-party-packages
.. _vco-repo: https://sdnpoc-vrodev.englab.juniper.net:8281/vco-repo/
.. _contrail-vro-plugin: https://github.com/Juniper/contrail-vro-plugin/blob/master/playbooks/contrail-build-vro-plugin/run.yaml#L17
.. _here: https://github.com/tungsten-infra/ci-utils/tree/master/tungsten_ci_utils/mirror_maven_repo
.. _packaging: https://github.com/Juniper/contrail-zuul-jobs/blob/master/zuul.d/contrail-jobs.yaml#L4
