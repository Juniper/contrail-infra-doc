Sanity Job Development
======================

All sanity jobs are dependent on both packaging and docker image building jobs which makes developing
them time-consuming. However, it is possible to use existing images of contrail components and run
only the sanity job by adding the `contrail_docker_registry` and `contrail_version` vars to the
job vars definition. The changes suggested below should only be done for testing changes in the sanity
job steps and should be reverted before performing the merge. Let's assume changes are needed for
`contrail-sanity-centos7-kolla-queens` job. Introduce the following changes:

1. Decide on the docker registry to be used e.g. opencontrailnightly.
2. Decide on the contrail version to be used. It is best to use the newest available tag e.g. latest or master-latest.

3. Adjust the definition of the job using the chosen values (contrail-zuul-jobs project).

  ::

    vars:
      contrail_docker_registry: opencontrailnightly
      contrail_version: master-latest

 A full definition would be:

  ::

    - job:
        name: contrail-sanity-centos7-kolla-queens
        parent: contrail-sanity-kolla-base
        required-projects:
          - name: Juniper/contrail-kolla-ansible
            override-checkout: contrail/queens
        vars:
          openstack_version: queens
          kolla_version: queens
          contrail_docker_registry: opencontrailnightly
          contrail_version: master-latest
        nodeset:
          nodes:
            - name: kolla-aio
              label: c7-systest-c

4. Add the job to contrail-zuul-jobs check pipeline (or experimental pipeline). The changes below are for
   contrail-zuul-jobs zuul.d/project.yaml file:

  ::

    - project:
        name: Juniper/contrail-zuul-jobs
        check:
          jobs:
            - contrail-sanity-centos7-kolla-queens
            - noop:
                irrelevant-files:
                  - ^roles/.*
                  - ^playbooks/.*
                  - ^tests/.*

5. Introduce the changes you want to test for the jobs (or it's parents') playbooks/roles (this does not circumvent
   trusted / untrusted context in any way so you will not be able to test the changes made in contrail-project-config).

6. Push the changeset to gerrit.

7. When the changes are tested revert the config definition changes and the contrail-zuul-jobs check pipeline changes.
