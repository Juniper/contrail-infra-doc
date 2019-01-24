Using Nexus TPC Repository
==========================

Nexus TPC repository is used to work with our source and binary rpms instead of old pulp server.

To prevent any misleading situations they had been splitting into two different repositories for each branch.

Nexus url link http://ci-nexus.englab.juniper.net/.

Direct links to RPM's:
::

  http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/master/binary/rpm_binary_file.rpm
  http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/master/source/rpm_binary_file.rpm
  http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/R5.0/binary/rpm_source_file.rpm
  http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/R5.0/source/rpm_source_file.rpm

Uploading new rpm's to the Nexus server
---------------------------------------

To upload new rpm's to the Nexus server use curl command.

Example:

::

  curl -u 'user:pass' -v --upload-file new.rpm http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/master/binary/new.rpm
