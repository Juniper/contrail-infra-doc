Services monitoring
===================

Problem
-------

Services in Zuul infrastructure are not monitored in sufficient manner.
So there is a need of some better monitoring of e.g. Gerrit service,
Zuul executors, Zuul web, Zuul merger, Zuul scheduler, Nodepool launchers.

**Monitoring solution should allow:**

* to gather and store node resource usage
* to health check services
* to alert about errors.

(Would be great if it store its configs as text files).

**We also need to have auto-healing of services. Some ideas on this:**

* simple bash scripts
   * run by cron in specified periods of time
   * run by trigger (remotely from alerting service?)
* simple application with REST API allowing remote restarting
  specified services
* manual intervention triggered by an Slack alert.

Questions to be asked (and hopefully answered)
----------------------------------------------

* Do we need continuous states (working/not working) history
  or just information about events (fail/recovery)?

   From discussion:
      - it could be last known status, which was transitioned to by an event.

* How long history do we want to have to search in it (data retention)?

   From discussion:
      - 3 months at least.


Solutions proposal (draft)
--------------------------

Staying with Grafana/Graphite
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
As already known and working solution.
But extending the monitoring to processes.
(Now monitored are just filesystems).

That was the first idea, as something most straightforward.

#. | For hosts with systemd i.e.
   | https://github.com/mbachry/collectd-systemd

#. | But Gerrit host is Ubuntu 14.04 and does not have systemd.
   | Then (some loose propositions):

   * Curling some URL (if Gerrit has URL that can be tested)
     (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_curl).
   * Executing some test command on system i.e. `pidoff GerritCodeReview`
     (need installation of sysvinit-utils) or `netstat -tulpn | grep 29418`
     (works OOTB) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_exec).
   * Checking existing TCP connections on specific port (i.e. 29418)
     (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_tcpconns).
   * | Processes plugin
     | "If processes are selected (the collectd.conf(5) manpage can tell
        you how) the following information is gathered.
        All this information is aggregated by the process name.
     | [...]
     | * The number of processes by that name"
     | (https://collectd.org/wiki/index.php/Plugin:Processes).
#. Auto-healing can be accomplished by independent simple bash scripts
   run by cron.

From Graphite online documentation:

   **Graphite Events**

   Graphite is well known for storing simple key/value metrics using the
   Whisper time-series database on-disk format. What is not well known about
   Graphite is that it also ships with a feature known as Events that supports
   a richer form of metrics storage suitable for irregular events often
   associated with metadata.

   Examples of data appropriate for this storage format include releases,
   commits, application exceptions, and state changes where you may wish
   to track or correlate the event with traditional time-series activity.

https://graphite.readthedocs.io/en/latest/events.html


**Pros**

* Using existing infrastructure.
* Continuous history of changes.
* Graphite can log events instead of continuously registering non-changing
  states.

**Cons**

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

Place of storing configuration:
   * Zabbix: in database
   * Icinga: in text files

Icinga has a great library of monitoring plugins.

Monitoring with Prometheus and InfluxDB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---+
|WIP|
+---+

Prometheus
""""""""""

Prometheus is kind of monitoring system,
that could be used as a data source for Grafana visualizations
(https://prometheus.io/docs/introduction/overview/; https://prometheus.io/docs/visualization/grafana/)

It's dedicated to dynamic infrastructure, but could be used to static also.

| It has alerting functionality with ``alerter``.
| Prometheus alerting can execute also webhooks:

.. code-block:: yaml

   receivers:
      - name: email_and_webhook
        email_configs:
      - to: "MyGMailAddress"
        from: "MyGMailAddress"
        smarthost: smtp.gmail.com:587
      webhook_configs:
      - send_resolved: false
        url: http://my.firewall_IP:3667/v1/alert


https://medium.com/@josebiro/building-a-simple-command-and-control-system-with-prometheus-6ce110b81e41


As for now it's very promising. And in future its utilization could be extended
to any new dynamic infrastructure also.
Could be used as alerter and data source for Grafana.

   pedoh
   25 points · 1 year ago
   I spent years in Nagios-land, and now I'm in deep with Prometheus,
   which I view as a combination of Nagios and Graphite. I think
   Prometheus is really solid, and am particularly excited about
   the integrations with Kubernetes (kube-prometheus, prometheus-operator),
   so if monitoring Kubernetes is a need for you, Prometheus is a strong
   option.

https://www.reddit.com/r/devops/comments/6hg4n6/best_monitoring_solutions/


InfluxDB
""""""""

InfluxDB is open-source time series database (TSDB).
Could be an alternative to Graphite. Also, it can receive data as Graphite
(`Graphite input <https://docs.influxdata.com/influxdb/v1.7/supported_protocols/graphite/>`_;
in case of migrating).

Simple **Graphite vs InfluxDB** comparison: https://db-engines.com/en/system/Graphite%3BInfluxDB .

But, as I read, there isn't clear winner. And this solutions are exchangeable.

Sensu
^^^^^
+---+
|WIP|
+---+

   Sensu is a comprehensive monitoring solution that is powerful enough
   to solve complex monitoring problems at scale, yet simple enough
   to use for traditional monitoring scenarios and small environments.
   It achieves this broad appeal via building upon two simple, yet powerful
   monitoring primitives: Service Checks and Event Processing. These building
   blocks also provide infinitely extensible pipelines for composing monitoring
   solutions.

https://docs.sensu.io/sensu-core/1.6/overview/what-is-sensu/

Work a bit like Zabbix or Nagios: Server-Client architecture
(https://docs.sensu.io/sensu-core/1.6/overview/how-sensu-works/#publishing-subscription-check-requests)

As a great advantage of Sensu over other mentioned monitoring solutions it that
**Sensu Client allow auto-healing (remedation)** of monitoring services.
(http://dev.nuclearrooster.com/2013/07/27/remediation-with-sensu/;
https://stackoverflow.com/questions/43251860/sensu-remediation-restart-failing-monitored-process;
https://medium.com/cloudgeek/how-to-streamline-sensu-email-alerts-using-sensu-remidiation-2e8da8dd9fe7;
https://blog.sensu.io/alert-fatigue-part-3-automating-triage-remediation-with-checks-hooks-handlers;
https://blog.sensu.io/using-check-hooks-a739a362961f;
https://gist.github.com/nstielau/6096392)

Other ideas (list to check)
---------------------------

* `Statsd <https://github.com/statsd/statsd>`_

   Zuul comes with support for the statsd protocol, when enabled and
   configured (see below), the Zuul scheduler will emit raw metrics
   to a statsd receiver which let you in turn generate nice graphics.

Promising:
^^^^^^^^^^

**zuul.nodepool.requests**

   Holds metrics related to Zuul requests and responses from Nodepool.
   States are one of:

   requested
      Node request submitted by Zuul to Nodepool

   canceled
      Node request was canceled by Zuul

   failed
      Nodepool failed to fulfill a node request

   fulfilled
      Nodes were assigned by Nodepool

**zuul.nodepool.requests.<state> (timer)**

   Records the elapsed time from request to completion for states failed
   and fulfilled. For example, zuul.nodepool.request.fulfilled.mean will
   give the average time for all fulfilled requests within each statsd
   flush interval.
   A lower value for fulfilled requests is better. Ideally, there will
   be no failed requests.

zuul.tenant.<tenant>.pipeline.<pipeline name>.resident_time (timer)

   A timer metric reporting how long each item has been in the pipeline.

https://zuul-ci.org/docs/zuul/admin/monitoring.html


Currently considering
---------------------

* **Prometheus** as monitoring system (gathering metrics from exporters).
* **Grafana** as dashboards, visualizations and simple alerting.
* Metrics exporters:

  - **node_exporter** - exporting OS and machine related metrics,
    as well as results of running predefined scripts.
  - **process-exporter** - exporting running processes metrics,
  - **mysqld_exporter** - exporting MySQL DB metrics,
  - **sql_exporter** - exporting results of predefined SQL queries.
  - (optional) **blackbox-exporter** - for checking status of open
    ports remotely.
  - (optional) **conntrack_exporter** - exporting information about
    existing network connections on host.

* (optional) **Alertmanager** for sending alerts.
* (optional) **InfluxDB** for longer term database.
* (optional) **Stunnel** (or Ghostunnel) for securing access
  to exporters' metrics and  communication between Prometheus and exporters.

Prometheus exporters are rather light and work OOTB, but have
no security implemented at all.

In untrusted environments exporters eventually need to be put behind
some reverse proxy or other tool adding encrypting of communication
and authentication/limitation on access.

Then the lightest and simplest solution seems to be
application just adding encryption and certificate
authentication (2in1).

Something like **stunnel** or **ghostunnel** (newer, slightly different approach, inspired by stunnel).

| https://www.stunnel.org/
| https://github.com/square/ghostunnel

Another try of securing metrics without use of www server as proxy
is **exporter_exporter**.

It exports all metrics on one port and behaves like a simple reverse proxy
with optional TLS encryption and authentication.

https://github.com/QubitProducts/exporter_exporter

Or with SSH tunneling (rather not considered):

https://miek.nl/2016/february/24/monitoring-with-ssh-and-prometheus/


Implementation
^^^^^^^^^^^^^^

Installation and configuration would be done with use of Ansible roles.

All project files would be stored in GIT.


Some links
----------

https://prometheus.io/docs/introduction/comparison/ (Prometheus to other one-by-one)

https://www.loomsystems.com/blog/single-post/2017/06/07/prometheus-vs-grafana-vs-graphite-a-feature-comparison

http://dieter.plaetinck.be/post/on-graphite-whisper-and-influxdb/

https://www.wavefront.com/collectd-vs-telegraf-comparing-metric-collection-agents/

https://angristan.xyz/monitoring-telegraf-influxdb-grafana/

https://www.reddit.com/r/devops/comments/5wzhzl/time_series_databases/

https://www.reddit.com/r/sysadmin/comments/3mihlx/anyone_using_influxdb_grafana/

https://graphite.readthedocs.io/en/latest/tools.html (other apps working with Graphite)

https://collectd.org/

https://github.com/sensu/sensu

https://blog.sensu.io/alert-fatigue-part-1-avoidance-and-course-correction

https://zuul-ci.org/docs/zuul/admin/monitoring.html

https://icinga.com/docs/icinga2/latest/doc/04-configuring-icinga-2/

https://www.monitoring-plugins.org/doc/man/index.html

https://github.com/imgix/prometheus-am-executor

https://medium.com/@josebiro/building-a-simple-command-and-control-system-with-prometheus-6ce110b81e41

https://miek.nl/2016/february/24/monitoring-with-ssh-and-prometheus/

https://github.com/ncabatoff/process-exporter

https://grafana.com/dashboards/249 (Named processes - Grafana Dashboard)

https://github.com/hiveco/conntrack_exporter

https://github.com/justwatchcom/sql_exporter

https://github.com/bakins/gearman-exporter

https://github.com/prometheus/blackbox_exporter (Remotely testing open ports)

https://github.com/savvydatainsights/monitoring (Prometheus - monitoring setup example)

https://github.com/gree-gorey/bash-exporter (exporter for Prometheus which allow to run custom scripts as tests)

https://github.com/kevincoakley/ansible-role-prometheus-node-exporter

https://github.com/mrlesmithjr/ansible-prometheus-node-exporter (another approach)

https://github.com/ocadotechnology/nexus-exporter

https://github.com/QubitProducts/exporter_exporter

https://github.com/prometheus/jmx_exporter (For Gerrit?)

https://www.stunnel.org/

https://github.com/square/ghostunnel
