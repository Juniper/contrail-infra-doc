Overview
========

ci-repo.englab.juniper.net hosts the following components:
* per-review docker registries
* nightly docker registries (bound to ports 5000 and 5010)
* mirrors of centos and docker upstream repos setup using pulp

Operational Tasks
=================

Per-review docker registries cleanup
------------------------------------

This section should mention:
* artifact curator - what it is what it does (cleaned up per-review pulp-repos
  (when it was used for hosting per-review rpm repos) and per-review docker registries). Make sure
  the python script is under source control somewhere (ci-utils?)
* where and how it is configured (crontab -l -u root)

Nightly docker registries cleanup
---------------------------------

* write down the procedure to clean up :5000 and :5010 docker registries
* make sure the scripts for the cleanup are under source code control somewhere (ci-utils?)

RPM mirrors
-----------

* List the current mirrors
* Why do we not host these on Nexus? (probably are not synced every day, the client might've wanted
  these repos to be synced on-demand)
* Add some operational exemplary commands (list repos, list rpms for a repo, check when the last sync
  occurred etc.)
