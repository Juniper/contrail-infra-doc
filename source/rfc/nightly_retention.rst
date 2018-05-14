Nightly Artifacts Retention Policy
==================================

Every 24 hours we start a full contrail build which we later on publish to various registries (DockerHub and internal ones). However, that gives us no control over the stability or other issues of the build. In order to do so, we need to provide a lifecycle for the builds.

Proposed policy
---------------

We should keep last 31 days (days, not builds, in case we fire multiple of them during a day) of builds. At the end of the month/beginning of the next one, QA team selects a build that becomes the build of the month. We will store last 12 (full year) of them independently from 31-day policy.


