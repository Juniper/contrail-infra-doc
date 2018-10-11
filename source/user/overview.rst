Overview
========

When you create a new change request on `Gerrit <https://review.opencontrail.org>`_, Zuul will start job execution. It will be indicated by a comment saying ``Starting jobs`` from ``Zuul v3 CI`` account.

The status of all currently running jobs can be verified on `Zuul's webpage <http://zuulv3.opencontrail.org>`_. Search for your change ID and click on the title to expand the list of the running jobs. Finished jobs will show their result on the list. By clickin on job's name, you will be redirected either to the log location (if job finished) or to log live stream (if the job is still running). When all of the jobs that are part of the buildset are finished, results are posted to Gerrit and job disappears from Zuul's live status page.

After a job finishes, the change (review) receives a vote and comment from Zuul.

Logs
----

Every job leaves a log directory on our `log server <http://logs.opencontrail.org>`_. Here's a brief breakdown of its contents:

* ``ara/`` - stores HTML-formatted report of the jobs. Failed playbooks/roles are marked with a red color.
* ``job-output.txt.gz`` - full, raw output of the job. It is automatically decompressed when downloaded or displayed in the browser.
* ``zuul-info`` - contains information about the job execution environment: Ansible variables, dmesg output, git logs etc.
* other files and directories - all other files are job specific e.g. dirs like ``unittest-logs`` or ``sanity-logs``.




