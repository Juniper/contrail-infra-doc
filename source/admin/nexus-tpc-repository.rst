Using Nexus TPC Repository
==========================

Nexus TPC repository is used to work with our source and binary rpms instead of old pulp server.

To prevent any misleading situations they had been splitting into two different repositories for each branch.

Nexus url link http://ci-nexus.englab.juniper.net/.

Direct link to RPM's directory:

::

  http://ci-nexus.englab.juniper.net/service/rest/repository/browse/yum-tungsten-tpc/

Uploading new RPM's to the Nexus server
---------------------------------------

To upload new RPM's to the Nexus server use curl command.

Please remember to provide the proper branch (master or 5.0), type (binary or source) and name of RPM in the URL.

Example:

::

  curl -u 'user:pass' -v --upload-file new.rpm http://ci-nexus.englab.juniper.net/repository/yum-tungsten-tpc/master/binary/new.rpm
