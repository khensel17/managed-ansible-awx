# managed-ansible-awx

This repository contains Ansible playbooks and configurations for Ansible AWX.

## Playbooks

| Name | Description | Documentation |
| ---- | ----------- | ------------- |
| awx-configure | Configures Ansible AWX based on configuration given via the inventory. | [Link](#awx-configure) |
| awx-deploy-inventory | Deploys the inventory in AWX based on configuration files in the ``.awx_config`` directory. | [Link](#awx-deploy-inventory) |
| awx-deploy-playbook | Deploys playbooks in AWX based on configuration files in the ``.awx_config`` directory. | [Link](#awx-deploy-playbook) |
| awx-sync-inventory | Playbook that syncs the inventory with AWX. | [Link](#awx-sync-inventory) |
| awx-sync-playbook | Playbook that makes sure the playbooks and configurations are synced with AWX. | [Link](#awx-sync-playbook) |

## Roles and Collections

This project depends on other Ansible Roles and Collections. These roles and collections are gathered by ansible-galaxy with the ``requirements.yml`` file.
In this file you can find the dependencies and versions.

### Local update Ansible collections

To install the Ansible Collections from the ``requirements.yml`` file locally run:

```bash
ansible-galaxy collection install -r ./requirements.yml -p ./playbooks/collections --force
ansible-galaxy role install -r ./requirements.yml -p ./playbooks/roles --force
```

## awx-configure

The playbook ``awx-configure`` configures several components in Ansible AWX, like:

- Credentials
- Execution Environments
- Organizations
- Teams

At the moment only the creation and updating these components are supported due to the shared Ansible AWX instance. This might change in the future where also the old components are deleted.

The configurations of these AWX components should be placed in the ``host_vars/awx.yaml`` of the inventory. The inventory ``test`` for the Ansible AWX test instance and the inventory ``production`` for the Ansible AWX prod instance.

Example contents ``host_vars/awx.yaml``:

```yaml
# Organizations
awx_config:
  controller_host: "awx-tst2.apps.okdtst.domain.local"
  controller_username: "appl_ansible_awx"
  controller_password: ENCRYPTED VAULT STRING HERE
  controller_oauth_token: ENCRYPTED VAULT STRING HERE

  organizations: []

  extra_execution_environments: []

  credentials: []
```

### Organizations

For the Hypervisor team the organization ``organization1`` is created and being managed via the ``awx-configure`` playbook.

**For now no multiple organizations are supported**

#### organizations:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | The name of the organization. |
| description | string | _optional_ | The description of the organization. |
| execution_environment | string | _optional_ | The name of the default execution_environment that is used by all underlying jobs. |
| teams | list [teams](#teams) | _optional_ | A list of teams under the organization. |

#### teams:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | Name of the team. |
| description | string | _optional_ | Description of the team. |

#### Example

```yaml
- name: organization1
  description: Hypervisor Team
  execution_environment: "awx-custom-ee"
  teams:
    - name: Operations
      description: "Operations Team"
    - name: Development
      description: "Development Team"
```

### Credentials

Also credentials can be configured that can be used.

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | Name of the credential. |
| description | string | _optional_ | Description of the credential. |
| credential_type | string | _required_ | The type of the credentials. For supported types: [documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/credential_module.html#parameter-credential_type) |
| owner.organization | string | _required_ | Set the owner of the credentials to the organization ``organization1``. |
| parameters | object [parameters](#parameters) | _required_ | Set the parameters configuration. |

#### parameters:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| username | string | _optional_ | The username of the credential. |
| password | Encrypted Ansible Vault string | _optional_ | The password of the credential encrypted by Ansible Vault. |
| ssh_key_path | string | _optional_ | The path of the SSH private key to import. |
| ssh_key_unlock | Encrypted Ansible Vault string | _optional_ | The passphrase of the SSH private key encrypted by Ansible Vault. |

#### Example

```yaml
credentials:
  - name: git-read-credential
    description: "Git Read Credential"
    credential_type: Source Control
    owner:
      organization: organization1
    parameters:
      username: USERNAME
      password: ENCRYPTED VAULT STRING HERE
```

## awx-deploy-inventory

To deploy an inventory in Ansible AWX you can add a configuration file under the ``.awx_config`` directory in the inventory Git project.

> **Warning**
> Inventory sources are provisioned in Ansible AWX according to the configuration present in the inventory Git repository under the directory ``.awx_config``. Any manual changes in Ansible AWX will be undone.

For now two files are supported under this directory:

- ``test.yaml``
  - Configuration of the Ansible AWX test instance.
- ``prod.yaml``
  - Configuration of the Ansible AWX production instance (TO BE DONE).

#### Example ``test.yaml``

```yaml
project_name: organization1-project-inventory-test
project_description: Project Inventory test
project_organization_name: organization1
project_scm_url: https://github.com/khensel17/inventories/ansible-inv-test.git
inventories:
  - name: organization1-inventory-test
    description: Inventory test
    sources:
      - name: inventory
        path: inventory.ini

```

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| project_name | string | _required_ | Name of the project in Ansible AWX. Follow the naming convention as example. |
| project_description | string | _optional_ | Description of the inventory project. |
| project_organization_name | string | _required_ | The name of the organization under where the project is created. Must be ``organization1``. |
| project_scm_url | string | _required_ | The ``https://`` clone url of the Git project. Make sure ``gitlab`` is replaced with ``git``. |
| project_scm_credential | string | _optional_ | Override the name of the credential used to clone Git projects. Default: ``git-read-credential``. |
| inventories | list [inventories](#inventories) | _required_ | A list object containing the Ansible AWX inventories. |

#### inventories:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | Name of the Ansible AWX inventory. Follow the naming convention as example. |
| description | string | _optional_ | Description of the Ansible AWX inventory. |
| sources | list [sources](#sources) | _required_ | A list object of sources. |

#### sources:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | Name of the inventory source. |
| path | string | _required_ | Filepath of the inventory file based on the Git project file structure. |

## awx-deploy-playbook

To deploy a playbook in Ansible AWX you can add a configuration file under the ``.awx_config`` directory in the inventory Git project.

> **Warning**
> Job templates are provisioned in Ansible AWX according the configuration present in the playbooks Git repository. Any manual changes in Ansible AWX will be undone.

For now two files are supported under this directory:

- ``test.yaml``
  - Configuration of the Ansible AWX test instance.
- ``prod.yaml``
  - Configuration of the Ansible AWX production instance (TO BE DONE).

#### Example ``test.yaml``

```yaml
project_name: organization1-project-playbooks-gitlab-management
project_description: Project Playbooks Gitlab Management
project_organization_name: organization1
project_scm_url: https://github.com/khensel17/playbooks/gitlab-management.git
project_scm_branch: development
job_templates:
  - name: create-gitlab-playbooks-project
    description: Create Gitlab project Playbooks template
    job_type: run
    inventory: organization1-inventory-test
    credentials:
      - test-vault-development
    playbook: playbooks/gitlab-create-playbooks-project.yaml
    allow_simultaneous: true

```

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| project_name | string | _required_ | Name of the project in Ansible AWX. Follow the naming convention as example. |
| project_description | string | _optional_ | Description of the inventory project. |
| project_organization_name | string | _required_ | The name of the organization under where the project is created. Must be ``organization1``. |
| project_scm_url | string | _required_ | The ``https://`` clone url of the Git project. Make sure ``gitlab`` is replaced with ``git``. |
| project_scm_branch | string | _required_ | The name of the branch that needs to be used. |
| project_scm_credential | string | _optional_ | Override the name of the credential used to clone Git projects. Default: ``git-read-credential``. |
| job_templates | list [job_templates](#job_templates) | _required_ | A list object containing the Ansible AWX job_templates. |

#### job_templates:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | Name of the Ansible AWX job template. Follow the naming convention as example. |
| description | string | _optional_ | Description of the Ansible AWX job template. |
| job_type | string | _required_ | The type of the job. Allowed values: ``run`` or ``check``. |
| playbook | string | _required_ | Path to the playbook file in the current Git repository. |
| inventory| string | _required_ | The name of the inventory that needs to be used for the job template. |
| credentials | list strings | _optional_ | A list of strings with the names of the credentials that needs to be used for the job. |
| forks | integer | _optional_ | The number of parallel or simultaneous processes to use while executing the playbook. Default: ``5`` |
| limit | integer | _optional_ | A host pattern to further constrain the list of hosts managed or affected by the playbook. |
| verbosity | integer | _optional_ | Control the output level Ansible produces as the playbook runs. 0 - Normal, 1 - Verbose, 2 - More Verbose, 3 - Debug, 4 - Connection Debug. Default: ``0`` |
| job_slicing | integer | _optional_ | The number of jobs to slice into at runtime. Will cause the Job Template to launch a workflow if value is greater than 1. Default: ``1`` |
| timeout | integer | _optional_ | Maximum time in seconds to wait for a job to finish (server-side). Default: ``0`` |
| diff_mode | boolean | _optional_ | Enable diff mode for the job template. Default: ``false`` |
| become_enabled | boolean | _optional_ | Activate privilege escalation. Default: ``false`` |
| ask_credential_on_launch | boolean | _optional_ | Prompt user for credential on launch. Default: ``false`` |
| ask_diff_mode_on_launch | boolean | _optional_ | Prompt user to enable diff mode (show changes) to files when supported by modules. Default: ``false`` |
| ask_execution_environment_on_launch | boolean | _optional_ | Prompt user for execution environment on launch. Default: ``false`` |
| ask_forks_on_launch | boolean | _optional_ | Prompt user for forks on launch. Default: ``false`` |
| ask_instance_groups_on_launch | boolean | _optional_ | Prompt user for instance groups on launch. Default: ``false`` |
| ask_inventory_on_launch | boolean | _optional_ | Prompt user for inventory on launch. Default: ``false`` |
| ask_job_slice_count_on_launch | boolean | _optional_ | Prompt user for job slice count on launch. Default: ``false`` |
| ask_job_type_on_launch | boolean | _optional_ | Prompt user for job type on launch. Default: ``false`` |
| ask_labels_on_launch | boolean | _optional_ | Prompt user for labels on launch. Default: ``false`` |
| ask_limit_on_launch | boolean | _optional_ | Prompt user for a limit on launch. Default: ``false`` |
| ask_scm_branch_on_launch | boolean | _optional_ | Prompt user for (scm branch) on launch. Default: ``false`` |
| ask_skip_tags_on_launch | boolean | _optional_ | Prompt user for job tags to skip on launch. Default: ``false`` |
| ask_tags_on_launch | boolean | _optional_ | Prompt user for job tags on launch. Default: ``false`` |
| ask_timeout_on_launch | boolean | _optional_ | Prompt user for timeout on launch. Default: ``false`` |
| ask_variables_on_launch | boolean | _optional_ | Prompt user for (extra_vars) on launch. Default: ``false`` |
| ask_verbosity_on_launch | boolean | _optional_ | Prompt user to choose a verbosity level on launch. Default: ``false`` |
| allow_simultaneous | boolean | _optional_ | Allows the job to run simultaneously. |
| extra_vars | dict | _optional_ | Extra variables can will be used in the job. For example ``test_variable: TEST_VALUE``. |
| survey | list [survey](#survey) | _optional_ | A list object of survey. |
| schedules | list [schedules](#schedules) | _optional_ | A list object of schedules. |

#### survey:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| question_name | string | _required_ | The name of the question / survey item. |
| question_description | string | _optional_ | The description of the question / survey item. |
| variable | string | _required_ | The key name of the variable how this is used in the Ansible Playbook. |
| type | string | _required_ | The type of the survey item. Allowed values: ``text``, ``textarea``, ``password``, ``multiplechoice``, ``multiselect``, ``integer``, ``float``. |
| default | string | _optional_ | The default value of the survey item. |
| required | boolean | _optional_ | Makes the survey item required of not. |
| min | integer | _required_ **see description** | The minimal length of the value. If ``type`` is one of: ``text``, ``textarea``, ``password``. |
| max | integer | _required_ **see description** | The maximum length of the value. If ``type`` is one of: ``text``, ``textarea``, ``password``. |
| choices | list string | _required_ **see description** | A list of strings with possible answers to select. If ``type`` is ``multiplechoice`` or ``multiselect``. |

**Example:**

```yaml
# text input
- question_name: Project name
  question_description: The name of the Gitlab project
  variable: gitlab_project_gitlab_project_name
  type: text
  default: ""
  required: true
  min: 3
  max: 50

# multiple choice
- question_name: Cluster
  question_description: Select the cluster to deploy the VM
  variable: cluster
  type: multiplechoice
  required: true
  choices:
    - cluster1
    - cluster2
    - cluster3
    - cluster4
```

#### schedules:

| parameter | type |   | description |
| --------- | ---- | - | ----------- |
| name | string | _required_ | The name of the schedule. |
| description | string | _optional_ | The description of the schedule. |
| enabled | boolean | _required_ | Enable the schedule. |
| start_time | string | _required_ | The start time following the following format: ``2021-06-01 00:00:00``. |
| extra_data | dictionary | _optional_ | A dictionary with variables that will be set in the job template. |
| rules | list [rules](#rules) | _required_ | A list object of the schedule's rules. [See documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/schedule_rruleset_lookup.html) |

#### rules:

Follows the same parameters as the rules parameters in the [documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/schedule_rruleset_lookup.html#parameter-rules).

**Example:**

```yaml
- name: TestSchedule
  description: This is a test schedule
  enabled: true
  start_time: "2021-06-01 00:00:00"
  # https://docs.ansible.com/ansible/latest/collections/awx/awx/schedule_rruleset_lookup.html
  rules:
    - frequency: hour
      interval: 6
      include: true
      # end_on: "2024-09-30 00:00:00"
    - frequency: hour
      interval: 2
      byweekday: "tuesday,wednesday,friday"
```

## awx-sync-inventory

This playbook will synchronize the AWX Inventory project and underlying sources based on the ``inventories`` variable defined in the configuration under ``.awx_config`` directory.

## awx-sync-playbook

This playbook will synchronize the AWX project based on the configuration under ``awx_config`` directory.
