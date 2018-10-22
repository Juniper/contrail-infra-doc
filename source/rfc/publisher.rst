Tungsten Fabric Publisher Specification
=======================================

The aim of this spec is to define the functional requirements the publisher software should fulfill and to provide user
stories.

Problem Description
===================

In Tungsten Fabric there are dozens of container images built in every periodic nightly build. Initially in the
image building jobs the images are pushed to an internal, per-build registry. Only after the jobs are finished
are they published to (at the moment of this writing) dockerhub. Therefore we need a _functional_ piece of software
which will provide this functionality.

Current State of Art
====================

The current `publisher.py` script has a few confusing functionalities, which make it harder to use and understand
on how it works:
* `image_overrides` - it _seems_ only the first image override from the list, which will match the source image
  will be used. It is more intuitive that all of the configurations should be applied if they match the source image.

* `openstack_version` is both a CLI argument and a configuration parameter for the configuration template (it is used
  to render the configuration file fields during execution). This may lead to unexpected behavior since a user might
  pass the parameter, while the configuration file has the openstack version hardcoded in some of the overrides. This is
  because the Openstack versions are not treated equally (ocata is tagged {{ release }}-latest and latest for all images,
   queens is {{ release }}-latest and latest for RHEL distributions).

* `build-registry` is both a CLI argument and a configuration parameter- introduces a confusion, should be provided only
  through the configuration file

* `filter` is a CLI argument - provides yet another layer for filtering images but is not actually used or needed.

Functional Requirements
=======================

The publisher software should provide the functionalities listed below. All of the actions should take an
input in the form of a configuration file (specification of which is provided in `Configuration Specification`
subsection).

F1. List images in a given registry matching a search criteria.
F2. Provide a dictionary where the keys are the source images matching a specific search criteria and the values are
    lists of target registries/repositories/tags the source image should be retagged to.
F3. Pull images from a given repository .
F4. Retag images based on the dictionary provided by F2.
    * Pull the source image if not present on the machine.
F5. Push images based on target lists in the dictionary provided by F2.
F6. Verify source images and all of their targets have the same checksums based on the dictionary provided by F2.
F7. Should handle connections to both trusted and untrusted registries.
F8. Should handle connections to registries with authentication enabled (e.g. dockerhub).
F9. Should support retries in both pull / push functionalities.

CLI Arguments
-------------

The software should have the following CLI inputs:
* config - path to the configuration file
* release - the release string (at the time of this writing this could be either 5.0 or master)
* build_no - build number for the release

Configuration Specification
---------------------------

Considering the functional and CLI requirements, the configuration file, to support `build_no` and `release` parameters,
needs to be a template. Below is an example configuration file for the `publisher` software.

:: bash

  source_registry:
    host: 1.1.1.1
    port: 1337
    untrusted: true

  target_registries:
    - host: 2.2.2.2
      port: 7331
      untrusted: true
    - host: 3.3.3.3
      port: 3317
      untrusted: false

  image_overrides:
    - image_matcher: "^.*$"
      tag_matcher: "^ocata-{{ release }}-{{ build_no }}$"
      release_tags:
        - "ocata-{{ release }}-{{ build_no }}"
        - "ocata-{{ release }}-latest"
        - "{{ release }}-{{ build_no }}"
        - "{{ release }}-latest"

    - image_matcher: "^.*$"
      tag_matcher: "^ocata-master-{{ build_no }}$"
      release_tags:
        - "latest"

    - image_matcher: "^.*$"
      tag_matcher: "^queens-.*$"
      release_tags:
        - "queens-{{ release }}-{{ build_no }}"
        - "queens-{{ release }}-latest"

    - image_matcher: "^.$"
      tag_matcher: "^rhel-queens-{{ release }}-{{ build_no }}$"
      release_tags:
        - "rhel-queens-{{ release }}-{{ build_no }}"
        - "rhel-queens-{{ release }}-latest"
        - "rhel-{{ release }}-{{ build_no }}"
        - "rhel-{{ release }}-latest"

Fields descriptions for the above configuration template:

* source_registry - the registry to search images in
* target_registries - the registries which images should be pushed to
* image_overrides - a list of configurations describing how images from the source registry should be retagged for
  the target registries

  - image_matcher - regex which will be matched with repository names available in the source registry
  - tag_matcher - regex which will be matched with tags of images available in the source registry
  - release_tags - a list of tags with which the images from the source registry should be pushed to the target registries




User Stories
============

As a `publisher` software user I want to be able to :

1. Compile a list of all images present in a given repository which have a specific tag e.g. ocata-master-252.
2. Compile a list of all images present in a given repository whose tags match a specific pattern (regex-wise) e.g.
   rhel-.*-master-259.
3.
