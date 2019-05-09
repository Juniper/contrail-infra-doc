Infrastructure
==============

This document lists all physical and virtual servers, their IPs and gives information on how to access
them. Some servers are placed under puppet control, with each member being provided a personal
account, access to others is done manually to a specific account, by either SSH key or password.
Access information for the latter ones is available in `password storage <link>`_.

Hetzner Hypervisors
-------------------

The two hypervisor servers host the .opencontrail.org servers e.g. zuulv3, logs etc.

=========================  ====================== =============
Server                     IP                     Access
=========================  ====================== =============
ci-host.opencontrail.org   148.251.46.180         SSH key
ci-host2.opencontrail.org  148.251.51.182         SSH key
=========================  ====================== =============

Openstack Clusters
------------------

There are two openstack clusters: A and C. From CI point of view both are used for spawning
VMs used in CI jobs, but there are also other projects set up on them for various purposes.

Cluster A
*********

The horizon interface can be found at: http://10.84.26.251/horizon.
The cluster is deployed with Contrail controller enabled as networking plugin for Neutron.
Contrail web interface is accessible at: https://10.84.25.251:8143/ and the service status page
at: http://10.84.26.251:9110/services.

The IP address (10.84.26.251) is a virtual IP address which only one control node
uses at a given time. This is achieved using keepalived.

=========================  =======================  ====================
IP                         Purpose                  Access
=========================  =======================  ====================
10.84.26.23                control node             password storage
10.84.26.24                control node             password storage
10.84.26.25                control node             password storage

10.84.26.5                 compute node (ci zone)   password storage
10.84.26.6                 compute node (ci zone)   password storage
10.84.26.8                 compute node (ci zone)   password storage
10.84.26.9                 compute node (ci zone)   password storage
10.84.26.10                compute node (ci zone)   password storage
10.84.26.11                compute node (ci zone)   password storage
10.84.26.12                compute node (ci zone)   password storage
10.84.26.14                compute node (ci zone)   password storage
10.84.26.15                compute node (ci zone)   password storage
10.84.26.16                compute node (ci zone)   password storage
10.84.26.26                compute node (ci zone)   password storage
10.84.26.27                compute node (ci zone)   password storage
10.84.26.28                compute node (ci zone)   password storage
10.84.26.30                compute node (ci zone)   password storage
10.84.26.31                compute node (ci zone)   password storage
10.84.26.32                compute node (ci zone)   password storage
10.84.26.33                compute node (ci zone)   password storage
10.84.26.34                compute node (ci zone)   password storage
10.84.26.35                compute node (ci zone)   password storage
10.84.26.36                compute node (ci zone)   password storage

10.84.26.7                 compute node (dev zone)  password storage
10.84.26.13                compute node (dev zone)  password storage
10.84.26.17                compute node (dev zone)  password storage
10.84.26.18                compute node (dev zone)  password storage
10.84.26.29                compute node (dev zone)  password storage

10.84.26.251               control virtual IP       -
=========================  =======================  ====================

Cluster C
*********

The horizon interface can be found at: http://10.87.71.100/horizon.
The cluster is deployed with Contrail controller enabled as networking plugin for Neutron.
Contrail web interface is accessible at: https://10.87.71.100:8143/ and the service status page
at: http://10.87.71.100:9110/services.

The IP address is a virtual IP address which only one control node
uses at a given time. This is achieved using keepalived.

=========================  ======================  ====================
IP                         Purpose                 Access
=========================  ======================  ====================
10.87.71.6                 control node            password storage
10.87.71.7                 control node            password storage
10.87.71.8                 control node            password storage

10.87.71.1                 compute node (ci zone)  password storage
10.87.71.2                 compute node (ci zone)  password storage
10.87.71.3                 compute node (ci zone)  password storage
10.87.71.9                 compute node (ci zone)  password storage
10.87.71.10                compute node (ci zone)  password storage
10.87.71.11                compute node (ci zone)  password storage
10.87.71.12                compute node (ci zone)  password storage

10.87.71.100               control virtual IP      -
=========================  ======================  ====================

Data Stores
-----------

This section contains information on any data stores which are used in the
project e.g. repository locations, docker registries etc.

+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| Name                              | IP             | Purpose                                          | Access                     |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| ci-repo.englab.juniper.net        | 10.84.5.81     | * per-review docker registries                   | SSH key (personal account) |
|                                   |                | * nightly docker registry                        |                            |
|                                   |                | * CentOS upstream mirrors (pulp)                 |                            |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| ci-nexus.englab.juniper.net       | 10.84.5.85     | * per-review RPM repos                           | SSH key (personal account) |
|                                   |                | * nightly RPM repositories                       |                            |
|                                   |                |                                                  |                            |
|                                   |                | * TPC RPM repositories                           |                            |
|                                   |                | * Maven mirror                                   |                            |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| mirrors.englab.juniper.net        | 10.87.72.46    | * ubuntu APT upstream mirrors (reprepro)         | SSH key (personal account) |
|                                   |                | * ubuntu APT contrail static repos (reprepro)    |                            |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| ci-rhel-mirror.englab.juniper.net | 10.84.5.83     | * RedHat mirrors (reposync)                      | SSH key                    |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| logs.opencontrail.org             | 148.251.110.22 | * storing output from CI jobs and nightly builds | SSH key (personal account) |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| stats.opencontrail.org            | 148.251.110.24 | * Zuul and build statistics database             | SSH key (personal account) |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+
| repo01-dev.opencontrail.org       | 148.251.5.90   | * Public third party cache for dev-env           | SSH key (personal account) |
+-----------------------------------+----------------+--------------------------------------------------+----------------------------+

CI infrastructure
-----------------

This section contains information on all of the nodes which comprise the actual CI and build system.

+--------------------------------+--------------------------+-----------------------------+----------------------------+
| Name                           | IP                       | Purpose                     | Access                     |
+--------------------------------+--------------------------+-----------------------------+----------------------------+
| review.opencontrail.org        | 148.251.110.21           | Gerrit server               | SSH key (personal account) |
+--------------------------------+--------------------------+-----------------------------+----------------------------+
| zuulv3.opencontrail.org        | 148.251.110.23           | Zuul scheduler              | SSH key (personal account) |
+--------------------------------+--------------------------+-----------------------------+----------------------------+
| ze0[1-4]-jnpr.opencontrail.org | 10.84.56.49              | Zuul executors              | SSH key (personal account) |
|                                | 10.84.56.129             |                             |                            |
|                                |                          |                             |                            |
|                                | 10.87.72.42              |                             |                            |
|                                |                          |                             |                            |
|                                | 10.87.72.25              |                             |                            |
+--------------------------------+--------------------------+-----------------------------+----------------------------+
| nl0[1-2]-jnpr.opencontrail.org | 10.84.35.178 (shut down) | Nodepool launchers          | SSH key (personal account) |
|                                | 10.84.35.185             |                             |                            |
+--------------------------------+--------------------------+-----------------------------+----------------------------+
| nb01-jnpr.opencontrail.org     | 10.84.35.154             | Nodepool disk image builder | SSH key (personal account) |
+--------------------------------+--------------------------+-----------------------------+----------------------------+

Other
-----

This section contains information on any other nodes not mentioned in previous sections

+--------------------------------------+----------------+-------------------------------------+----------------------------+
| Name                                 | IP             | Purpose                             | Access                     |
+--------------------------------------+----------------+-------------------------------------+----------------------------+
| mirror.sj01.juniper.opencontrail.org | 10.84.56.27    | Repo proxy to various repositories  | SSH key (personal account) |
|                                      |                | (e.g. yum, apt, pypi), mentioned in |                            |
|                                      |                | `Data Stores`_                      |                            |
+--------------------------------------+----------------+-------------------------------------+----------------------------+
| ci-puppetmaster.opencontrail.org     | 148.251.110.19 | Puppet master for CI infra          | SSH key (personal account) |
+--------------------------------------+----------------+-------------------------------------+----------------------------+
