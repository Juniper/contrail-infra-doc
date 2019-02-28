Metrics
=======

Problem
-------

Zuul jobs are run on independent, created on-demand, nodes. No information is available about the
state changes of the nodes throughout the whole job run in terms of CPU, memory, storage usage
or - for that matter - any information related to the buildset. The only results coming
out of buildsets currently are test reports, run logs and built artifacts.

To have better understanding and control over the CI system and any insight on how changes in code
impact the building and testing processes, more information about the system the buildsets are being executed on
needs to be gathered and on the outputs of the builds themselves.

Metrics
-------

This section contains proposals of metrics to gather during and from buildsets.

#. General

   * run duration

#. Physical resources

   * CPU usage by time
   * memory usage by time
   * storage usage by time
   * disk IO by time
   * network IO by time
   * CPU hours
   * peak memory usage
   * peak disk usage

   *Note: in case of storage usage, might need to consider only specific mount points*

#. Test jobs specific metrics

   * number of tests run for a specific buildset - might be useful in enforcing that more tests are added
   * duration of each test case execution
   * run log size

#. Packaging jobs specific metrics

   * size of created packages

#. Container builder jobs specific metrics

   * size of created images

Technical requirements
----------------------

#. The metric gathering system should allow for storing two types of metrics:

   * time-series data (e.g. CPU usage at given point in time)
   * discrete values (e.g. built packages size)

#. Each buildsets metrics should be gathered separately, with consideration of review and patchset ID's.
   This would require for the metrics naming convention to be of the form:

      buildset.<pipeline>.<project-name>.<branch>.<job-name>.<gerrit-change-id>.<gerrit-patchset>.<resource>

   for the check, gate and experimental pipelines. For the other pipelines the git branch and a timestamp of the start
   of the jobs should be used instead of gerrit-change-id and gerrit-patchset:

      buildset.<pipeline>.<project-name>.<branch>.<job-name>.<build_number>.<resource>

   e.g.

      buildset.check.contrail-project-config.add-something-new.zuul-jobs-linters.42705.1.cpu
      buildset.check.contrail-project-config.fix-typo.zuul-jobs-linters.43333.3.mem

      buildset.periodic-sanity.contrail-analytics.master.contrail-systest-full-centos74-kolla-ocata.41414.2.notests
      buildset.periodic-sanity.contrail-analytics.R5_0.contrail-systest-full-centos74-kolla-ocata.43343.4.logsize

      buildset.periodic-nightly.contrail-analytics.R5_0_1.contrail-vnc-build-package-ubuntu-xenial.112.packagesize

      buildset.periodic-nightly.contrail-analytics.master.contrail-vnc-build-containers-rhel7-ocata.113.imagesize

   *Note: in case of release branches the dots would need to be replaced with underscores.*

#. The metric gathering system (or the corresponding frontend) should allow to aggregate and filter stored data
   per job/project/branch (e.g. average UT job disk usage for contrail-controller on branch 5.0). There should
   also be a frontend allowing to graph the stored metrics.

   Consider two test runs A and B (for a specific project) for which we gathered physical resources
   usage spread across 1000 points in time. A requirement would be to compute the average resource usage for the
   aligned data points, that is average CPU/mem etc usage in the 1, 2, 3 and so on second of the job run (assuming
   metrics are gathered each second).

   *Note: metric gathering systems typically save metric values for either provided unix timestamps or for the moment
   the value was received.*

#. Metrics should be stored for 14 days in case of *check* and *gate* and 60 days for other pipelines.