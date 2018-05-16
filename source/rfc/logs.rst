Logs
====

Problem
-------

The log server is full way too often. We're currently running on a 14-day policy, even though we should be keeping at least 30 days of logs.

Current setup
~~~~~~~~~~~~~

We run on top of our Hetzner hypervisor with:

- 4 cores
- 8GB of RAM
- 400GB of disk (LVM)

Notes:

- there's only 600GB free on the hypervisor
- 2nd hypervisor has 800GB free
- we seem to be generating from 20 to 40GB of logs per day (as of 2018-05-04)

Retention policy
----------------

We should keep 30 days of logs for reviews - that should be enough to be able to merge a review properly.

Nightlies
~~~~~~~~~

Nightly logs should follow the very same policy that build artifacts (packages and containers) will follow to ensure that we have all information we need historically. What's more, we should consider keeping nightly logs for a longer period of time (60 or 90 days), as someone might use an older build (even though we don't provide any support for them).

Solution
--------

We should increase server size to 800GB (allowing some wiggle room for other machines to expand if necessary) which should last us during quiet time (20GB/day gives us 600GB with 30 days). In the meantime we should focus on a proper fix.


Proper fix
----------

Storage
~~~~~~~

We should move to an object storage solution as soon as possible. One possibility would be AWS with S3. Having object storage mounted as fs allows us to store the logs same as before, but with no limits other than our own enforced policies.

