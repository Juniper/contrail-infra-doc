Services monitoring
===================

Problem
-------

Services in Zuul infrastructure are not monitored in sufficient manner.
So there is a need of some better monitoring of e.g. Gerrit service,
Zuul executors, Zuul web, Zuul merger, Zuul scheduler, Nodepool launchers.


Solution proposal (draft)
-------------------------

Staying with Grafana/Graphite (as already known and working solution).

#. | For hosts with systemd i.e.
   | https://github.com/mbachry/collectd-systemd

#. | But Gerrit host is Ubuntu 14.04 and does not have systemd.
   | Then (some loose propositions):

   * Curling some URL (if Gerrit has URL that can be tested) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_curl)
   * Executing some test command on system i.e. `pidoff GerritCodeReview` (need installation of sysvinit-utils) or `netstat -tulpn | grep 29418` (works OOTB) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_exec)
   * Checking existing TCP connections on specific port (i.e. 29418) (https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_tcpconns)
   * | Processes plugin
     | "If processes are selected (the collectd.conf(5) manpage can tell
        you how) the following information is gathered.
        All this information is aggregated by the process name.
     | [...]
     | * The number of processes by that name"
     | (https://collectd.org/wiki/index.php/Plugin:Processes)

