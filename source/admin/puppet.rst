Puppet
======

Puppet is used in a client-server configuration for static node management in
the project. Both public infra nodes (that is, having public IP addresses, e.g. logs.opencontrail.org)
and private ones (inside Juniper network e.g. zuul executors) have puppet agents installed.
Puppetmaster is thus also a public node (ci-puppetmaster.opencontrail.org).

Setup
-----

Some general information about the setup:

* `production` is the main branch
* repository on the puppetmaster node is updated by manual pushes of the code to
  puppet@ci-puppetmaster.opencontrail.org:puppet.git
* the pushed branch's name maps to a name of an environment in puppet, which means that if you push
  'test' branch, the puppet master will create a new 'test' environment in /etc/puppet/environments
  (or whatever the `environmentpath` is set to in /etc/puppet/puppet.conf)
* each puppet agent has the `environment` set to `production`, so testing any changes done in any
  other branch is more complex - one would need to reconfigure the puppet agent on the specific
  node to a different `environment` setting

Development Workflow
--------------------

Use the workflow described below to make changes to puppet-related functionalities:

#. Make your changes locally on `production` branch.
#. Inform the team that you will push changes for testing. This is required so that no one will
   interrupt or overwrite your changes by pushing other ones.
#. Push your `production` branch changes to puppet.
   NOTE: if you're pushing changes which could impact multiple nodes then you will need another approach
   to not break anything. For instance: duplicate a module or manifest with changes applied and apply
   it only to a single node.
#. Test the changes and make sure everything is working as expected.
#. Go through the usual review process on Gerrit.
#. After the review is merged push the `production` branch forcefully to puppet to overwrite previous
   changes and keep the branch up-to-date on the master node.
