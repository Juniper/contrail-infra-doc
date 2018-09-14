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


Looking up project source paths
-------------------------------

Some of the jobs should be decoupled from their main project. When you
write a job that builds some particular project, you assume that the
``zuul.project`` variable is set to your primary project. Sometimes, however,
the job has to run properly also on other projects, for example when built in
the periodic pipeline or unit-tested in the contrail-zuul-jobs project.
For this to work, the `src_dir` should be resolved from the ``zuul.projects``
dictionary instead of the ``zuul.project`` variable. The Jinja incantation to
look up a project by its short name is:

``{{ (zuul._projects.values() | selectattr('short_name', 'equalto', 'contrail-infra-doc') | first).src_dir }}``

Warning: in Zuul before commit 4223442 you should use the ``_projects`` dict,
in newer Zuul versions it is renamed to just ``projects``.
