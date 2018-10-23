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


Pipeline: experimental-sanity
=============================

The manual work described in the `Sanity Job Development` section is available out of the box for every project
through the 'experimental-sanity' pipeline. The pipeline runs a relevant set of sanity jobs (might be different depending
on the project e.g. OpenShift sanity will be run only for changes to openshift-ansible project) using the latest images
from the nightly build.

The pipeline does not vote in any way in the review, it is only meant to be used to test changes which impact how deployment
or tests are run (e.g. changing how an ansible role behaves).

A full workflow using the pipeline would be as follows:

1. A user submits a new review or a patchset to an existing one. Zuul triggers the 'check' pipeline jobs for the project.

2. The user writes a comment on the review 'check experimental-sanity' to trigger the experimental-sanity pipeline
   jobs assigned for the project.

   NOTE: Both the check and experimental-sanity pipelines run have the same sanity jobs assigned, but are different
   since in the check pipeline the docker images used in sanity jobs come from the build-containers dependent jobs.
   The images used in experimental-sanity pipeline are the latest images from the most recent, successful nightly build.

3. The experimental-sanity pipeline jobs will finish sooner, faster providing feedback for the user whether their
   change is working. This makes it more convenient to test changes to deployment code as the user does not have to wait
   for the packages and containers to be built (which in sum could take even 2 hours).

NOTE: Keep in mind that running sanity jobs in the above described way for reviews which change the actual TungstenFabric
code does not make sense (since the jobs will not use containers which include the change). At the moment of this writing
running experimental-sanity pipeline makes sense for changes in the following projects:

  * contrail-zuul-jobs
  * contrail-ansible-deployer
  * contrail-kolla-ansible
  * contrail-helm-deployer
  * contrail-helm-infra
  * openstack-helm
  * openshift-ansible
