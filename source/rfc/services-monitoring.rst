Services monitoring
===================

Problem
-------

Services in Zuul infrastructure are not monitored in sufficient manner.
So there is a need of some better monitoring of e.g. Gerrit service,
Zuul executors, Zuul web, Zuul merger, Zuul scheduler, Nodepool launchers.


Solutions proposal (draft)
--------------------------

Staying with Grafana/Graphite (as already known and working solution)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
But extending the monitoring to processes.
(Now monitored are just filesystems).

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
   ran by cron.

**Pros**

* Using existing infrastructure.
* Continuous history of changes.

**Cons**

* Continuous flow of unneeded data being submitted to Grafana.
  (We need only info about downtimes).
* No integrated auto-healing.


Old-school solution (simple bash scripts)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#. Simple bash scripts ran by cron in given time

   * Checking if service is working by listing running processes
     or opened ports.
   * Restarting service when it is not working.
   * Sending alerts by mail or/and submitting them to Slack.

**Pros**

* Easy to implement and maintain.
* Integrated auto-healing.

**Cons**

* No continuous state monitoring

  * ran by cron in given time periods
  * just information about changes - no change tracking (i.e. in Grafana).


Full blown monitoring solution (i.e. Icinga, Zabbix, Nagios)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pros and cons just like staying with Grafana/Graphite, but it also need to
implement a new system besides existing one (maybe replacing it?).

Worth noticing: no auto-healing also.

Monitoring with Prometheus and Influx (new idea from email about Tech-Talk)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Something completely new to me.
But for now I suppose it could be something similar (in functionality)
to Grafana/Graphite.

