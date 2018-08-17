Mirrors in CI
=============

Central proxy
-------------

**Address:** 
  - 10.84.56.27
  - mirror.regionone.jnpr-contrail-ci.opencontrail.org (in CI)

**Description:** This is a central point for all the mirroring, but it does not contain all the packages.

**Software:** Nginx configured as reverse proxy for the other servers.

**Configuration location:** `/etc/nginx/sites-available/default`

**URL endpoints:**

======================  =======================================  ============
Endpoint                Target                                   Description
======================  =======================================  ============
/ubuntu/                10.84.72.46 (`Ubuntu mirror`_)           Ubuntu packages mirror
/rhel/                  10.84.5.83 (`RHEL mirror`_)              RHEL packages mirror
/centos/                `ci-repo.englab.juniper.net`_            CentOS packages mirror
/epel/                  `ci-repo.englab.juniper.net`_            Epel packages mirror
/oc-centos-tpc/         `ci-repo.englab.juniper.net`_            old opencontrail tpc repo
/oc-centos-tpc-         `ci-repo.englab.juniper.net`_            per-branch opencontrail tpc packages
/contrail-third-party/  `ci-repo.englab.juniper.net`_            cached contrail-third-party-cache
/pypi/                  `PyPi <https://pypi.org>`_               local caching mirror for pypi packages
======================  =======================================  ============

ci-repo.englab.juniper.net
--------------------------

**Address:** 10.84.5.81

**Description:** This server is hosting pulp repositories and docker registries.

**Software:** Pulp, docker, apache (for contrail-third-party)

**Configuration location:**
  - Pulp: `/etc/pulp` (puppet-managed)
  - contrail-third-party: `/opt/opencontrail_ci/`

**Notes:**
  - `opencontrail-tpc-*` repositories are managed manually via pulp interface
  - `contrail-third-party` is populated via cron script from `github repository <https://github.com/Juniper/contrail-third-party-cache>`_.

Ubuntu mirror
=============

**Address:** 10.84.72.46

**Description:** Puppet-managed repository host for ubuntu, using `reprepro <https://wiki.debian.org/DebianRepository/SetupWithReprepro>`_. Setup taken from openstack's puppet repository.

**Software:** reprepro, apache

**Configuration location:**
  - `contrail-infra <https://github.com/Juniper/contrail-infra>`_ repository.
  - `/etc/reprepro` - reprepro configuration on the host

RHEL mirror
===========

**Address:** 10.84.5.83

**Description:** A RHEL machine connected to RedHat servers and mirroring specific repositories locally for the CI to use.

**Software:** subscription-manager, reposync, createrepo, apache

**Configuration location:** 
  - `/opt/sync.sh` - executable for mirroring and configuration
