Using Nexus TPC Repository
==========================

Nexus TPC repository is used to work with our source and binary rpms instead of old pulp server.

To prevent any missleading situations they have been splited into two different repositories for each branch.

Nexus url link http://ci-nexus.englab.juniper.net/.

Full path to repositories:
::
  http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/{{ branch }}/binary/rpm_binary_file.rpm
  http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/{{ branch }}/source/rpm_source_file.rpm

Uploading new rpm's to the Nexus server
---------------------------------------

To upload new rpm's to the Nexus server use curl command.

::
  curl -u 'user:pass' -v --upload-file new.rpm http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/{{ branch }}/binary|source/new.rpm

