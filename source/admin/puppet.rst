Puppet
======

Puppet is used in a client-server configuration for static node management in
the project. Both public infra nodes (that is, having public IP addresses, e.g. logs.opencontrail.org)
and private ones (inside Juniper network e.g. zuul executors) have puppet agents installed.
Puppetmaster is thus also a public node (ci-puppetmaster.opencontrail.org).

Setup
-----

Applying the changes to the production systems:

* 'production' is the main branch, so ``git checkout production`` (instead of the usual 'master')

* The changes go live when you manually ``git push puppet@ci-puppetmaster.opencontrail.org:puppet.git``

  * If it denies authentication, add your SSH key to `ci-puppetmaster:~puppet/.ssh/authorized_keys`

* Observe the agents' behavior: ``systemctl status puppet``

  * It should be 'active' and periodically log 'Finished catalog run' (every half an hour)
  * Your new changes will be visible in the same log.
  * If you'd like to get the changes immediately: ``systemctl restart puppet``

Development Workflow
--------------------

Use the workflow described below to make changes to puppet-related functionalities:

#. Make your changes locally on `production` branch
#. Inform the team that you will push changes for testing. This is required so that no one will
   interrupt or overwrite your changes by pushing other ones.
#. Push your `production` branch changes to puppet.

   NOTE: if you're pushing changes which could impact multiple nodes then you will need another approach
   to not break anything. So either:

   * Do ``git push`` to ci-puppetmaster2, which is consumed only by test puppet agents.
   * Alternatively: duplicate a module or manifest, change this duplicate, apply it only to a single node.
   * Another possibility: the branch that you `git push` maps to a name of an environment in puppet. It means that
     if you push to branch named 'mytest', the puppet master will create a new 'mytest' environment in /etc/puppet/environments
     (or whatever the `environmentpath` is set to in /etc/puppet/puppet.conf). Each puppet agent has the `environment` set
     to `production`, so you need to reconfigure the chosen puppet agent(s) to use yours.

#. Test the changes and make sure everything is working as expected.
#. Go through the usual review process on Gerrit.
#. After the review is merged push the `production` branch forcefully to puppet to overwrite previous
   changes and keep the branch up-to-date on the master node.
