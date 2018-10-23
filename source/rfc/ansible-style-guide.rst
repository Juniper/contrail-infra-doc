Ansible Style Guide
===================

Purpose
-------

The purpose of this document is to define and describe the style guide to be used for Ansible
in contrail-project-config and contrail-zuul-jobs projects. Topics include but are not limited
to: role tasks separation between task files, task definition, interpolation syntax etc.

This guide is inspired by `whitecloud's ansible style guide <https://github.com/whitecloud/ansible-styleguide>`_.

Rules
-----

ANSIBLE-J0001 - YAML File Format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

YAML files should begin with the YAML marker `---` marking the start of the document and end with a blank line.

.. code-block:: yaml

  ---

  - name: restart apache
    service:
      name: 'apache2'
      state: 'restarted'

ANSIBLE-J0002 - YAML File Extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Always use the '.yaml' file extension for YAML files.

ANSIBLE-J0003 - Hosts Declaration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the following order for hosts declaration:
  * `hosts:` declaration
  * host options in alphabetical order
  * pre_tasks
  * roles
  * tasks

.. code-block:: yaml

  - hosts: webservers
    remote_user: 'centos'
    vars:
      tomcat_state: 'started'
    pre_tasks:
      - name: set the timezone to America/Boise
        lineinfile:
          dest: '/etc/environment'
          line: 'TZ=America/Boise'
          state: 'present'
        become: true
    roles:
      - { role: 'tomcat', tags: 'tomcat' }
    tasks:
      - name: start the tomcat service
        service:
          name: 'tomcat'
        state: '{{ tomcat_state }}'

ANSIBLE-J0004 - Task Declaration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Lowercase the first letter of a task name.
* Use the map syntax. Use only one space after the colon.
* Use the following order for task declaration:

  * task name
  * task map declaration
  * loop operators
  * task options in alphabetical order

.. code-block:: yaml

  - name: template sensu configuration
    template:
      src: '{{ item }}.conf.j2'
      dest: '/etc/sensu/{{ item }}.conf'
      mode: '0644'
      owner: 'sensu'
      group: 'sensu'
    with_items:
      - sensu-api
      - sensu-client
      - sensu-server
    become: true

ANSIBLE-J0005 - Role File Structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The number and size of task definitions in a task file should be kept in reasonable
bounds. Should a single task file get too big, it should be separated into multiple
task files, with only `include_tasks:` statements present in the main.yaml file of the role.

.. code-block:: bash

  roles/
    fetch-logs-sanity-tests/
      tasks/
        container-logs.yaml
        contrail-logs.yaml
        main.yaml
        kolla-logs.yaml
        sanity-logs.yaml

For the above structure, the main.yaml file would hold:

.. code-block:: yaml

  - include_tasks: container-logs.yaml
  - include_tasks: contrail-logs.yaml
  - include_tasks: kolla-logs.yaml
  - include_tasks: sanity-logs.yaml

ANSIBLE-J0006 - Quotes
^^^^^^^^^^^^^^^^^^^^^^

Always quote strings (defaulting to single quotes). Do NOT quote:

  * `hosts:` targets (e.g. `hosts: databases` rather than `hosts: 'databases'`)
  * `include_tasks:` and `include_roles:` target file names
  * task names
  * registered variables
  * number values
  * boolean values
  * conditional logic (`when:` task options)

Use double quotes only where single quotes can not be used to write syntactically or semantically correct code:

.. code-block:: yaml

  - hosts: all
    vars:
      my_string: "this is a \nmultiline string" # 'this is a\nmultiline' would not produce the same result
      my_num: 42
      my_boolean: false
    tasks:
      - name: this task doesn't make much sense
        copy:
          dest: '/tmp/file'
          content: '{{ my_string }}:{{ my_num}}'
        register: registered_var
        when: not my_boolean

ANSIBLE-J0007 - Booleans
^^^^^^^^^^^^^^^^^^^^^^^^

Use true/false for boolean values.

.. code-block:: yaml

  - hosts: all
    become: true
    roles:
      - add-ssh-keys

ANSIBLE-J0008 - Variables
^^^^^^^^^^^^^^^^^^^^^^^^^

Use `snake_case` for variable names:

.. code-block:: yaml

  my_string: 'hello thar'
  my_number: '42'
  my_boolean: true

ANSIBLE-J0009 - Module Usage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the following modules by default:
  * `package` instead of `yum` and `apt`
  * `command` instead of `shell` when actual shell functionality is not needed


Linter
------

`ansible-lint <https://github.com/willthames/ansible-lint>`_ allows for linting Ansible playbooks
and roles. It has a set of predefined rules and in this context is extensible. Adding custom rules
is supported by runtime flags. Defining a rule is a matter of inheriting from the `AnsibleLintRule`
class and providing an implementation of a specific method.

skip_ansible_lint - Tag Usage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use skip_ansible_lint tag with any task that you want to skip running linter for.
Remember to add comment with information about reason why are you skipping lint check 
and name of skipped ansible-lint rule.

.. code-block:: yaml

  - name: install dependencies from Puppetfile
    shell: PATH={{ ruby_path }}:$PATH r10k puppetfile install Puppetfile
    args:
      chdir: '{{ workdir }}'
    #ANSIBLE0013 ignore since we can't use shell with rbenv
    tags:
    - skip_ansible_lint
