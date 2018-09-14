Manifests usage in CI
=====================

Workflow
--------

1. After spawning a slave VM(s), Zuul Executor downloads repositories on them.
#. `Manifest translator <https://github.com/Juniper/contrail-project-config/blob/master/roles/repo-sandbox-prepare/files/manifest_translator.py>`_ is used to change github remotes to local repositories in manifest from contrail-vnc.
#. git-repo is used to arrange repositories in proper folder hierarchy.

Thanks to that approach all VMs have the exact same versions of repositories without duplicating code to arrange them.
