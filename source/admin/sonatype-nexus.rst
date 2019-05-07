Overview
========

`Sonatype Nexus <https://www.sonatype.com/nexus-repository-sonatype>` is used to host
per-review, nightly and TPC RPM repositories and - in the future - will be used as a per-review
and nightly docker image hosting and for hosting CentOS RPM mirrors.

The admin interface is available under: http://ci-nexus.englab.juniper.net/

RPM Repositories
----------------

The general idea behind different RPM repos hosted on Nexus should be written here, how are they
configured (e.g. why the particular repo depth etc.).

Per-review
**********

Nightly
*******

Third Party Cache
*****************

The TPC repository (named 'yum-tungsten-tpc') is used to third party RPMs of two origins:

* ones which we can build from source code (referred as 'source'; can be re-build from
  contrail-third-party-packages_ repo.
* ones, for which we only have binary files (.rpm)

This Nexus repository has a repodata depth '2' to enable hosting source/binary RPM repos for each
of the release branches (currently: master, R5.0, R5.1).

Uploading RPMs
**************

To upload new RPM's to the Nexus server use curl command.

Please remember to provide the proper branch (master or 5.0), type ('binary' or 'source') and name of RPM in the URL.

Example:

::

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
