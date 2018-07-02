Nightly Builds dashboard
========================

Purpose of this document
------------------------

This document contains technical requirements and basic information about possible implementation of **Build Dashboard** for Zuul v3-based nightly builds of Tungsten Fabric.

Problem
-------

Build system has a periodic pipeline for creating nightly builds, which are run once a day, per-branch (master, and any active RX.Y branches) and identified by a numeric ID, incremented with each build. 

There are some cases in which we want to run these periodic builds manually and it requires logging in to the zuul server and running a set of commands.

After the build has finished, there is no easy way of getting to the log files and artifacts of the build if the user does not know the correct URL and directory schema for that.

Requirements
------------

Provide a web-based dashboard which allows an authenticated user to (on a per-branch basis):

1. start build,

2. see status of a build,

3. abort the build

4. see the history of builds.

   Each build consists of variable number of build jobs, which can be divided in separate categories by operating system + OpenStack release. Each category can be considered as a *sub-pipeline*, during which packages and containers are built and finally the containers are published. Each of the parts should have it's status displayed.

Resources
---------

1. Connection and api operations to zuul server are managed by zuul client application: `docs <https://docs.openstack.org/infra/zuul/admin/client.html>`_.

2. Documentation about build infrastructure: `infra-doc <http://logs.opencontrail.org/static/infra-doc/>`_.
