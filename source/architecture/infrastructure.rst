Infrastructure
==============

This document lists all physical and virtual servers, their IPs and gives information on how to access
them. Some servers are placed under puppet control, with each member being provided a personal
account, access to others is done manually to a specific account, by either SSH key or password.
Access information for the latter ones is available in `password storage <link>`.

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
The IP address is a virtual IP address which only one control node
uses at a given time. This is achieved using keepalived.

=========================  ======================  ====================
IP                         Purpose                 Access
=========================  ======================  ====================
10.84.26.23                control node            password storage
10.84.26.24                control node            password storage
10.84.26.25                control node            password storage

10.84.26.5                 compute node (ci zone)  password storage
10.84.26.6                 compute node (ci zone)  password storage
10.84.26.8                 compute node (ci zone)  password storage
10.84.26.9                 compute node (ci zone)  password storage
10.84.26.10                compute node (ci zone)  password storage
10.84.26.11                compute node (ci zone)  password storage
10.84.26.12                compute node (ci zone)  password storage
10.84.26.14                compute node (ci zone)  password storage
10.84.26.15                compute node (ci zone)  password storage
10.84.26.16                compute node (ci zone)  password storage
10.84.26.26                compute node (ci zone)  password storage
10.84.26.27                compute node (ci zone)  password storage
10.84.26.28                compute node (ci zone)  password storage
10.84.26.30                compute node (ci zone)  password storage
10.84.26.31                compute node (ci zone)  password storage
10.84.26.32                compute node (ci zone)  password storage
10.84.26.33                compute node (ci zone)  password storage
10.84.26.34                compute node (ci zone)  password storage
10.84.26.35                compute node (ci zone)  password storage
10.84.26.36                compute node (ci zone)  password storage

10.84.26.7                 compute node (dev zone) password storage
10.84.26.13                compute node (dev zone) password storage
10.84.26.17                compute node (dev zone) password storage
10.84.26.18                compute node (dev zone) password storage
10.84.26.29                compute node (dev zone) password storage

10.84.26.251               control virtual IP      -
=========================  ======================  ====================

Cluster C
*********

The horizon interface can be found at: http://10.87.71.100/horizon.
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

