Overview
========


.. image:: ../_static/architecture.png

Workflow
--------

1. A review on Gerrit instance is created.
#. Zuul Scheduler listens to event stream from Gerrit and creates node creation request for NodePool.
#. NodePool spawns a slave VM(s) needed to run a job and returns it’s details to Zuul. Uses OpenStack as a provider cloud.
#. Zuul Executor downloads repositories on the slave VM(s).
#. Repositories are arranged in proper folder hierarchy using VNC and manifests.
#. Jobs are run on the slave VM(s).
#. Artifacts are uploaded to Docker Registry and YUM repository.
#. Logs and Ansible Run Analyzer (ARA) report get uploaded to the log server later to be available via http.
#. Job results are posted to Gerrit and Verified vote is casting accordingly to the result.
#. The review receives Code-Review +2 vote.
#. Review gets approved (Approved +1 vote).
#. After all voting requirements are met, Zuul automatically submits the change for merge and Gerrit merges it.
