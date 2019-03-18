Services monitoring
===================

Problem
-------

Services in Zuul infrastructure are not monitored in sufficient manner.
So there is a need of some better monitoring of e.g. Gerrit service,
Zuul executors, Zuul web, Zuul merger, Zuul scheduler, Nodepool launchers.

**Monitoring solution should have:**

* monitoring daemon (Collectd*, Telegraf, Node Exporter, Statsd)
* stats collector (Graphite*, Prometheus)
* database to store monitoring data (Graphite*, InfluxDB)
* alerting (Grafana plugin*, Prometheus, Kapacitor)
* some nice visualization (Grafana*, Prometheus)

| * currently used

**We also need to have auto-healing of services. Some ideas on this:**

* simple bash scripts
   * run from cron in specified periods of time
   * run by trigger (remotely from alerting service?)
* simple application with REST API allowing remote restarting
  specified services
* some nice guy (or girl) watching Slack alert channel and restarting
  failed services when he/she sees an alert.

(obsolete) Questions to be asked (and hopefully answered)
---------------------------------------------------------

* Do we need continuous states (working/not working) history or just information about events (fail/recovery)?
* How long history do we want to have to search in it (data retention)?


Solutions proposal (draft)
--------------------------

Staying with Grafana/Graphite (as already known and working solution)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
But extending the monitoring to processes.
(Now monitored are just filesystems).

That was the first idea, as something most straightforward.

#. | For hosts with systemd i.e.
   | https://github.com/mbachry/collectd-systemd

#. | But Gerrit host is Ubuntu 14.04 and does not have systemd.
   | Then (some loose propositions):

   * Curling some URL (if Gerrit has URL that can be tested) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_curl).
   * Executing some test command on system i.e. `pidoff GerritCodeReview` (need installation of sysvinit-utils) or `netstat -tulpn | grep 29418` (works OOTB) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_exec).
   * Checking existing TCP connections on specific port (i.e. 29418) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_tcpconns).
   * | Processes plugin
     | "If processes are selected (the collectd.conf(5) manpage can tell
        you how) the following information is gathered.
        All this information is aggregated by the process name.
     | [...]
     | * The number of processes by that name"
     | (https://collectd.org/wiki/index.php/Plugin:Processes).
#. Auto-healing can be accomplished by independent simple bash scripts
   run by cron.

**Pros**

* Using existing infrastructure.
* Continuous history of changes.

**Cons**

* Continuous flow of unneeded data being submitted to Graphite.
  (If we would be needed only info about events).
* No integrated auto-healing.


Old-school solution (simple bash scripts)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#. Simple bash scripts run by cron in given time

   * Checking if service is working by listing running processes
     or open ports.
   * Restarting service when it is not working.
   * Sending alerts by mail or/and submitting them to Slack.

**Pros**

* Easy to implement and maintain.
* Integrated auto-healing.
* Not centralized (no SPOF).

**Cons**

* No continuous state monitoring

  * run by cron in given time periods
  * just information about changes - no change tracking (i.e. in Grafana).

* Not centralized (no state database/dashboard).
* If cron/script does not work, we assume everything is OK
  (alerts are not sent).


Full blown monitoring solution (i.e. Icinga, Zabbix, Nagios)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pros and cons just like staying with Grafana/Graphite, but it also need to
implement a new system besides existing one (maybe replacing it?).

Worth noticing: no auto-healing also.

Monitoring with Prometheus and InfluxDB (new idea from email about Tech-Talk)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---+
|WIP|
+---+

Prometheus is kind of monitoring system,
that could be used as a data source for Grafana visualizations
(https://prometheus.io/docs/introduction/overview/; https://prometheus.io/docs/visualization/grafana/)

It's dedicated to dynamic infrastructure, but could be used to static also.

It has integrated alerting functionality.


Sensu
^^^^^
+---+
|WIP|
+---+

   Sensu is a comprehensive monitoring solution that is powerful enough to solve complex monitoring problems at scale, yet simple enough to use for traditional monitoring scenarios and small environments. It achieves this broad appeal via building upon two simple, yet powerful monitoring primitives: Service Checks and Event Processing. These building blocks also provide infinitely extensible pipelines for composing monitoring solutions.

https://docs.sensu.io/sensu-core/1.6/overview/what-is-sensu/

Work a bit like Zabbix or Nagios: Server-Client architecture (https://docs.sensu.io/sensu-core/1.6/overview/how-sensu-works/#publishing-subscription-check-requests)



https://github.com/sensu/sensu


Other ideas (list to check)
---------------------------

* `StatsD <https://github.com/statsd/statsd>`_

Some links
----------

https://prometheus.io/docs/introduction/comparison/ (Prometheus to other one-by-one)

https://www.loomsystems.com/blog/single-post/2017/06/07/prometheus-vs-grafana-vs-graphite-a-feature-comparison

http://dieter.plaetinck.be/post/on-graphite-whisper-and-influxdb/

https://www.wavefront.com/collectd-vs-telegraf-comparing-metric-collection-agents/

https://angristan.xyz/monitoring-telegraf-influxdb-grafana/
