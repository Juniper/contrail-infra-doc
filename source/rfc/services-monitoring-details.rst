Services monitoring (going into details)
========================================

+---+
|WIP|
+---+

What?
-----

List of services we need to watch:

* Nodepool launchers (2) (importance HIGH, urgency HIGH)
* Zookeeper (1) (importance HIGH, urgency HIGH)
* Nodepool builder (1) (importance HIGH, urgency normal)
* Zuul scheduler (1) (importance HIGH, urgency normal)
    * Gearman (1) (importance HIGH, urgency normal)
* Zuul merger (1) (importance normal, urgency normal)
* Zull executors (4) (importance HIGH, urgency normal)
* Zuul web (1) (importance normal, urgency normal)
* Zuul proxy (?)
* Gerrit (1) (importance HIGH, urgency normal)
* Apache2 (Zuul WebUI proxy) (1) (importance HIGH, urgency normal)
* Stats (1) (importance normal, urgency normal)


Legend:

Importance - how important is given service
Urgency - how urgent is to monitor given service (high possibility of failing)


Where?
------

On which servers does those services run.

**Nodepool launchers**

* nl01-jnpr.opencontrail.org (10.84.35.178; behind VPN; talk to Zookeeper) - down
* nl02-jnpr.opencontrail.org (10.84.35.185; behind VPN; talk to Zookeeper)

**Nodepool builder**

* nb01-jnpr.opencontrail.org (10.84.35.154; behind VPN)

**Zuul scheduler, Zuul merger, Zuul web, Zookeeper, Apache2 (Zuul WebUI proxy)**

* zuulv3.opencontrail.org (148.251.110.23; public)

**Zuul executors**

* ze01-jnpr.opencontrail.org (10.84.56.49; behind VPN)
* ze02-jnpr.opencontrail.org (10.84.56.129; behind VPN)
* ze03-jnpr.opencontrail.org (10.87.72.42; behind VPN)
* ze04-jnpr.opencontrail.org (10.87.72.25; behind VPN)

**Zuul proxy**

**Gerrit**

* review.opencontrail.org (148.251.110.21; public)

**Stats**

* stats.opencontrail.org (148.251.110.24; public)


How?
----

How we can monitor them (some specifics about each service).

**Nodepool launchers**

* Long living connection (Nodepool launcher 2* <--> Zookeeper) could be
  monitored by listing systems open connections (uncertain if this
  connection disappears while not working).
* Nodepool or Zuul log watching (hard to do, as logs are uneasy to parse).
    * analyzing i.e. are there any machines created in given time period
      (i.e. last hour).

\* Nodepool launcher 1 is down and this is normal.

**Zookeeper**

* Monitor running process hosting ``/usr/share/java/zookeeper.jar`` (Java)
  (on host)
* Monitor open port ``3389`` (on host, remotely)
* zookeeper::client_port, 2181 (?)

**Nodepool builder**

?

**Zuul scheduler**

* Monitor running processes (always 2; second is Gearman)
  ``zuul-scheduler`` (on host)
* gearman_listen_port: 7999 (moved from default 4730)

**Zuul merger**

* Monitor running process ``zuul-merger`` (on host)

**Zuul executors**

* Monitor running process (?)
* Monitoring open ports:
    * ze01: ``0.0.0.0:7900`` (on host or remotely),
      ``127.0.0.1:10000`` (on host)
    * ze02: ``0.0.0.0:7901`` (on host or remotely),
      ``127.0.0.1:10010`` (on host)
    * ze03: ``0.0.0.0:7902`` (on host or remotely),
      ``127.0.0.1:10020`` (on host)
    * ze04: ``0.0.0.0:7903`` (on host or remotely),
      ``127.0.0.1:10030`` (on host)

**Zuul web**

* Monitor running process ``zuul-web`` (on host)
* Monitor open port ``0.0.0.0:9000`` (on host or remotely, but firewalled)

**Zuul proxy**

Is it an service/process?

**Gerrit**

* Monitor running process ``GerritCodeReview`` (Java) (on host)
* Monitor open ports: ``8181`` --> ``80``, ``29418`` (on host or remotely)

**Entire Zuul workflow (supplementary)**

* Auto-executing test Zuul job, which must to pass (as from definition).
  When it hung or fails, means something is not as it should be.

**Stats**

* Monitor running process mysqld (on host)
* Monitor open port ``3306`` (MySQL) (on host, remotely)

Why?
----

Why do we need to monitor those services (except of that this is rational
and definitely lays in good practices).

**Nodepool launchers**

Sometimes nodepool disconnect from Zookeeper and test builds are not
executed until restart of nodepool service. In worst case Zuul tests
are stopped for several hours or even weekend).

Possible causes:

* As this is long living connection, the firewall could blocking it
  after some time.
* ...?


Found patterns
--------------

When Nodepool is disconnected from Zookeeper "Nodepool nodes requests"* rise
instantly and heavily. On Grafana dashboard they rise from few requests
per 10 min (most of the time not passing threshold of 20) to maximum 75
requests per 10 min (and staying at this). At the same time, Zuul queued
jobs\** and Zuul running jobs\*\** counters stay at 0.

| * ``ci.zuulv3.statsd.gauge.zuul.nodepool.current_requests``
| \** ``ci.zuulv3.statsd.gauge.zuul.executors.jobs_queued``
| \*\** ``ci.zuulv3.statsd.gauge.zuul.executors.jobs_running``

I.e. Grafana dashboard stats for last situation (26-27 Jan):

http://148.251.5.91/grafana/dashboard/db/zuul?orgId=1&from=1548413374000&to=1548931774000

**But we must have in mind, that those stats should be considered simultaneously.**

Nodepool nodes requests sometimes can be enormously high (i.e. over 200
per 10 min) while number of Zuul queued and running jobs are also high
(28 Feb - 1 Mar). And this is correct (?)

Notes
-----

``contrail-infra`` repo - check for info about port mappings

Ports to identify:

* ``0.0.0.0:34654`` (tcp)
* ``0.0.0.0:8001`` (tcp)
* ``0.0.0.0:35850`` (udp)
* ``0.0.0.0:44117`` (udp)

Collectd has implemented StatsD network protocol as a plugin since v5.4.

    The StatsD plugin implements the StatsD network protocol to allow
    clients to report "events", such as the serving of a web page. These
    events are aggregated by collectd and dispatched regularly.

https://collectd.org/wiki/index.php/Plugin:StatsD



**Apache2 DocumentRoot (on zuulv3):**

DocumentRoot /var/lib/zuul/www

Apache2 rewrites:

    | RewriteRule ^/keys/(.*) http://127.0.0.1:8001/opencontrail/keys/$1 [P]
    | RewriteRule ^/status.json$ http://127.0.0.1:8001/opencontrail/status.json [P]
    | RewriteRule ^/status/(.*) http://127.0.0.1:8001/opencontrail/status/$1 [P]
    | RewriteRule ^/connection/(.*) http://127.0.0.1:8001/connection/$1 [P]
    | RewriteRule ^/console-stream ws://127.0.0.1:9000/console-stream [P]
    | RewriteRule ^/static/(.*) http://127.0.0.1:9000/static/$1 [P]
    | RewriteRule ^/jobs/(.*) http://127.0.0.1:9000/jobs/$1 [P]


From logs
^^^^^^^^^


**Example of execution done (/var/log/zuul/zuul.log):**

    2019-03-25 12:00:19,892 INFO zuul.ExecutorClient: Build <gear.Job 0x7ff02cac7668 handle: b'H:148.251.110.23:15418' name: executor:execute unique: f3d0b252b19f4da782cd5fd394d44460> complete, result SUCCESS


**Example of correct talk to nodepool (/var/log/zuuul/zuul.log):**

    | 2019-03-25 12:01:54,623 INFO zuul.nodepool: Node request <NodeRequest 200-0000234731 <NodeSet ubuntu-xenial OrderedDict([('ubuntu-xenial', <Node 0000215981 ubuntu-xenial:ubuntu-xenial-small>)])OrderedDict()>> fulfilled
    | 2019-03-25 12:01:54,625 INFO zuul.nodepool: Accepting node request <NodeRequest 200-0000234731 <NodeSet ubuntu-xenial OrderedDict([('ubuntu-xenial', <Node 0000215981 ubuntu-xenial:ubuntu-xenial-small>)])OrderedDict()>>
    | 2019-03-25 12:01:54,651 INFO zuul.Pipeline.opencontrail.check: Completed node request <NodeRequest 200-0000234731 <NodeSet ubuntu-xenial OrderedDict([('ubuntu-xenial', <Node 0000215981 ubuntu-xenial:ubuntu-xenial-small>)])OrderedDict()>> for job contrail-build-win2016 of item <QueueItem 0x7ff034b44a58 for <Change 0x7ff0374121d0 50370,1> in check> with nodes <NodeSet ubuntu-xenial OrderedDict([('ubuntu-xenial', <Node 0000215981 ubuntu-xenial:ubuntu-xenial-small>)])OrderedDict()>
    | 2019-03-25 12:01:54,663 INFO zuul.nodepool: Setting nodeset <NodeSet ubuntu-xenial OrderedDict([('ubuntu-xenial', <Node 0000215981 ubuntu-xenial:ubuntu-xenial-small>)])OrderedDict()> in use


**Example of connection problem (nodepool logs; logs can be found also: nl01.contrail.juniper.net, project (private repo): contrail-zuul):**

    | 2019-01-26 07:03:47,861 INFO nodepool.CleanupWorker: ZooKeeper suspended. Waiting
    | 2019-01-26 07:04:05,469 INFO nodepool.DeletedNodeWorker: ZooKeeper suspended.Waiting
    | 2019-01-26 07:04:12,006 INFO nodepool.NodePool: ZooKeeper suspended. Waiting


**Try of correlate nodepool errors to Zookeeper logs (but these seems to be common errors - possibly not helping):**

    | 2019-01-25 13:38:10,941 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d7130090b, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-25 13:38:10,991 [myid:] - ERROR [SyncThread:0:NIOServerCnxn@178] - Unexpected Exception:
    | java.nio.channels.CancelledKeyException
    | at sun.nio.ch.SelectionKeyImpl.ensureValid(SelectionKeyImpl.java:73)
    | at sun.nio.ch.SelectionKeyImpl.interestOps(SelectionKeyImpl.java:77)
    | at org.apache.zookeeper.server.NIOServerCnxn.sendBuffer(NIOServerCnxn.java:151)
    | at org.apache.zookeeper.server.NIOServerCnxn.sendResponse(NIOServerCnxn.java:1082)
    | at org.apache.zookeeper.server.FinalRequestProcessor.processRequest(FinalRequestProcessor.java:170)
    | at org.apache.zookeeper.server.SyncRequestProcessor.flush(SyncRequestProcessor.java:200)
    | at org.apache.zookeeper.server.SyncRequestProcessor.run(SyncRequestProcessor.java:131)

[...]

    | 2019-01-25 13:57:06,404 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 3539ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide
    | 2019-01-25 13:57:06,731 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@362] - Exception causing close of session 0x160d92d7130090b due to java.io.IOException: Connection reset by peer
    | 2019-01-25 13:57:45,251 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 2601ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide

[...]

    | 2019-01-26 00:41:55,919 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 1187ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide
    | 2019-01-27 17:12:24,798 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d713009a8, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 17:12:37,463 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d713009a9, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 17:15:27,537 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d713009aa, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 17:15:52,147 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d713009ab, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 17:16:34,594 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d713009ac, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 17:17:02,337 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d713009ad, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 18:19:40,176 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 1082ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide

[...]

    | 2019-01-27 19:37:12,966 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 1103ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide
    | 2019-01-27 19:59:24,524 [myid:] - WARN [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:3389:NIOServerCnxn@357] - caught end of stream exception
    | EndOfStreamException: Unable to read additional data from client sessionid 0x160d92d7130090b, likely client has closed socket
    | at org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:230)
    | at org.apache.zookeeper.server.NIOServerCnxnFactory.run(NIOServerCnxnFactory.java:203)
    | at java.lang.Thread.run(Thread.java:748)
    | 2019-01-27 19:59:24,553 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 3218ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide
    | 2019-01-27 19:59:24,553 [myid:] - ERROR [SyncThread:0:NIOServerCnxn@178] - Unexpected Exception:
    | java.nio.channels.CancelledKeyException
    | at sun.nio.ch.SelectionKeyImpl.ensureValid(SelectionKeyImpl.java:73)
    | at sun.nio.ch.SelectionKeyImpl.interestOps(SelectionKeyImpl.java:77)
    | at org.apache.zookeeper.server.NIOServerCnxn.sendBuffer(NIOServerCnxn.java:151)
    | at org.apache.zookeeper.server.NIOServerCnxn.sendResponse(NIOServerCnxn.java:1082)
    | at org.apache.zookeeper.server.FinalRequestProcessor.processRequest(FinalRequestProcessor.java:170)
    | at org.apache.zookeeper.server.SyncRequestProcessor.run(SyncRequestProcessor.java:169)
    | 2019-01-27 20:11:24,337 [myid:] - WARN [SyncThread:0:FileTxnLog@334] - fsync-ing the write ahead log in SyncThread:0 took 1151ms which will adversely effect operation latency. See the ZooKeeper troubleshooting guide

[...]
