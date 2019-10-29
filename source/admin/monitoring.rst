Monitoring
==========

CI monitoring solution consists of several parts:

* Prometheus as the main system and service monitoring tool
* Alertmanager as the alerting toolkit
* Grafana for data visualization
* Graphite as a metrics storage
* Zuul reporters for reporting information about jobs status
* MySQL for storing Zuul jobs information and unit test statistics
* collectd as a metric aggregator agent
* unit test statistics gathering script_

Grafana & Graphite
------------------

Both Grafana and Graphite are set up on the same server. You can view the Grafana interface at
http://148.251.5.91/grafana while Graphite's at http://148.251.5.91/graphite.

MySQL
-----

The database is used:

* By zuul-scheduler itself for storing information about jobs.
* By test statistics gathering script_ for storing unit test results.
* By Grafana as a data source for various dashboards.

Collectd
--------

Collectd is a metric collector and aggregator, running on the infra nodes. It sends metric data
periodically to Graphite. It currently mostly gathers physical resource utilisation but it is
possible to use it to send metric data from your own application (or even a bash script).

Test Runner
-----------

The contrail-vnc-unittest-centos7-tntestr_ job sends unit test statistics information using the
statistics gathering script_. More information about the solution, on what kind of statistics are
being gathered can be found in the README file in test runner_ repository.

.. _script: https://github.com/tungsten-infra/ci-utils/blob/master/tungsten_ci_utils/test_statistics/test-analyzer.py
.. _contrail-vnc-unittest-centos7-tntestr: https://github.com/Juniper/contrail-zuul-jobs/blob/master/zuul.d/contrail-jobs.yaml#L25
.. _runner: https://github.com/tungstenfabric/tungsten-test-runner

