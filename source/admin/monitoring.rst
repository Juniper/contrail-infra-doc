Monitoring
==========

CI monitoring solution consists of several parts:

* Grafana for data visualization
* Graphite as a metrics storage
* Zuul reporters for reporting information about jobs status
* MySQL for storing Zuul jobs information and unit test statistics
* collectd as a metric aggregator agent
* unit test statistics gathering script_

Grafana & Graphite
------------------

Both Grafana and Graphite are set up on the same server. You can view the Grafana interface at
http://grafana.opencontrail.org/grafana while Graphite's at http://grafana.opencontrail.org/graphite.

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

Collectd exec scripts
^^^^^^^^^^^^^^^^^^^^^

Collectd has an ability to execute external scripts and submit its results to Graphite.

Utilized scripts:

* For checking locally if services were listening on specified ports.
* For checking (via Zuul API) if there were Zuul jobs running for too long.
* For checking nightly build status (obsoleted).

Such scripts are stored in `/etc/collectd/scripts/` directory.
Their execution is done by exec_ plugin (which is collectd's internal plugin).

In `/etc/init.d/collectd/collectd.conf`:

.. code-block::

   LoadPlugin exec

   <Plugin exec>
     Exec "nobody" "/etc/collectd/scripts/<scriptname>.sh"
   </Plugin>

Remote http/https checks by blackbox_exporter
---------------------------------------------

Http and https services are remotely checked by blackbox_exporter which is a part of internal
(private) monitoring.

Simple auto-healing solutions
-----------------------------

Some of errors are hard to predict and avoid, but easy to fix.
So we introduced simple auto-fixing ideas into work.

Monitoring and automatic recovery of nodepool service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

From time to time, long-lasting SSH connection between nodepool-launcher and zookeeper services
is being tear down (possibly network issue). Nodepool-launcher lacks an ability of automatic
reconnection in a such situation (at least in version, we use), so new VM's are not spawned
and CI slowly stop (not being able to process new jobs).

It was observed, in a such situation on our monitoring, Zuul running jobs count is slowly decreasing,
while nodepool requests count is sky-rocketing. So subtraction of this two, allowed us to create an alert.

Alert from Grafana executes webhook setting locally "nodepool-launcher needs restart" flag.
Script (run by cron) on nodepool-launcher host monitor that flag's status and do
nodepool-launcher service restart. After that, remotely resets that flag to its default value
and sends info to Slack channel.

Another, much simpler, approach (which was not introduced for now) is to monitor `nodepool-launcher.log`
and do restart when in one of two newest lines occurs string "INFO nodepool.NodePool: ZooKeeper suspended. Waiting".
It exists in cron, but it's commented out.

Auto-healing for Gerrit
^^^^^^^^^^^^^^^^^^^^^^^

Sometimes Gerrit Java process crashes. Then its restart is required.

Simple script run from cron, in every 5 min checks if Gerrit was listening on its network port.
If not, is doing restart of Gerrit Java service and sends info to Slack channel.

Another approach could be to monitor `/etc/init.d/gerrit status` command output.
It was observed (and is quite possible it is recurring), during such crash, string
`Gerrit running pid=xxxx` is missing from output.
So, simple script could monitor this and restart Gerrit Java service when needed.

Test Runner
-----------

The contrail-vnc-unittest-centos7-tntestr_ job sends unit test statistics information using the
statistics gathering script_. More information about the solution, on what kind of statistics are
being gathered can be found in the README file in test runner_ repository.

.. _script: https://github.com/tungsten-infra/ci-utils/blob/master/tungsten_ci_utils/test_statistics/test-analyzer.py
.. _contrail-vnc-unittest-centos7-tntestr: https://github.com/Juniper/contrail-zuul-jobs/blob/master/zuul.d/contrail-jobs.yaml#L25
.. _runner: https://github.com/tungstenfabric/tungsten-test-runner
.. _exec: https://collectd.org/wiki/index.php/Plugin:Exec
