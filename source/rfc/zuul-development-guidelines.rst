Zuul job configuration development guidelines
=============================================

Zuul YAMLs - general

#. Playbook naming: `playbooks/<job_name>/{run,pre,post}.yaml`
#. Zuul-jobs vs project-config: which jobs/roles where
#. Site-variables
#. Nodes(ets) naming
#. job variants - where to override settings

Variables

#. common libs usage
#. Zuul return
#. Docker tags
#. Docker registries
#. defaults and vars
#. looking up project source paths

Ansible code

#. All the Ansible code should follow the style guide
#. No tasks should be present in the playbooks (extract everything to roles)
