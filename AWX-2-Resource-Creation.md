# Resource Creation

#########################################################################
#########################################################################
#########################################################################

# PART 2 Vorarbeit (Bitte vorerst ignorieren)

#########################################################################
#########################################################################
#########################################################################

<details><summary>Terminology</summary>

Understanding the key terminology in AWX is crucial for effectively using the platform:

|         |                                                                 |
|---------|-----------------------------------------------------------------|
| Organizations    | An Organization is a logical collection of Users, Teams, Projects and Inventories. And it is the highest level in the AWX hierarchy. It also provides a structured way to manage access and permissions within AWX. |
| Playbooks | Playbooks are the YAML files that define a series of tasks to be executed by Ansible. They are used to automate complex workflows and configurations. |
| Inventories | Inventories are lists of hosts or groups of hosts that Ansible manages. They can be defined in static files or dynamically generated from various sources. |
| Credentials| Credentials in AWX are used to authenticate and authorize access to remote systems. AWX seemless integration with various cloud, vault or secret service providers allows the users/teams/organizations to securely store and manage credentials for different environments and use cases. |
| Projects | Projects in AWX are collections of Ansible playbooks and related files. They can be sourced from various version control systems like Git, allowing for versioned and collaborative development of automation scripts. |
| Jobs | Jobs are instances of AWX launching an Ansible playbook against an inventory of hosts. AWX allows the user to schedule, monitor, and manage jobs, providing detailed logs and status information. |
| Job Templates | A job template is a definition and set of parameters for running an Ansible job. They include information about the playbook to run, the inventory to use, and any extra variables or options required for the execution. Job templates are useful to execute the same job many times. |
| Workflows | Workflows are sequences of job templates that can be executed in a specific order, where the templates may or may not share inventory, playbooks, or permissions. They allow you to chain multiple automation tasks together, enabling more complex and conditional automation scenarios. |
| Execution Environments | Execution Environments in AWX are containerized environments that provide the necessary dependencies and runtime for executing Ansible playbooks. They ensure consistency and isolation of the execution context, allowing users to define and manage the specific versions of Ansible and other required tools. Execution Environments can be customized and shared across different teams and projects, facilitating a standardized and reproducible automation process. |

</details>


### Directory-Tree example of AWX collection

When you install the collections and Roles the files get created similar to the following tree structure.

```
awx_collection/
├── playbooks/
│   ├── playbook1.yaml
│   ├── playbook2.yaml
│   ├── [...]
│   ├── vars/
│   │   ├── organizations.yml
│   │   ├── machine_credentials.yml
│   │   ├── scm_credentials.yml
│   │   ├── custom_credential_types.yml
│   │   ├── custom_credentials.yml
│   │   ├── inventories.yml
│   │   ├── inventory_sources.yml
│   │   ├── hosts.yml
│   │   ├── projects.yml
│   │   ├── execution_environments.yml
│   │   ├── job_templates.yml
│   │   ├── workflow_job_templates.yml
│   │   ├── [...]
│   │   
│   ├── templates/
│   │   ├── survey.json.j2
│   │   ├── [...]

```

### playbook.yaml

When working with Ansible and AWX, organizing your playbooks efficiently is crucial for maintainability and scalability. Let's break down the structure of the provided playbook and understand the technicalities and advantages of this approach.

```yaml
---
- hosts: "{{ target_host | default('localhost') }}"

  vars_files:
    - "{{ path | default('.') }}/awx-local-env.yaml"
    - "{{ path | default('.') }}/organization-sthings.yaml"
    - "{{ path | default('.') }}/scm-creds-sthings.yaml"
    - "{{ path | default('.') }}/projects-sthings.yaml"
    - "{{ path | default('.') }}/machine-creds-sthings.yaml"
    - "{{ path | default('.') }}/dynamic-inventory-vsphere.yaml"
    - "{{ path | default('.') }}/inventory-source.yaml"
    - "{{ path | default('.') }}/ee-sthings.yaml"
    - "{{ path | default('.') }}/schedule-baseos.yaml"
    - "{{ path | default('.') }}/job-baseos.yaml"
    - "{{ path | default('.') }}/survey-baseos.yaml"

  tasks:
    - ansible.builtin.import_tasks: check_connection.yaml
    - ansible.builtin.import_tasks: render_survey.yaml
    - ansible.builtin.import_tasks: creation_tasks.yaml
```

Hosts:
The first line specifies the target host for the playbook. Using a variable (target_host) with a default value (localhost) makes the playbook flexible and reusable across different environments. Define the target_hosts variable in your environment and change it to the host you want to run against.

Variable files:
Including multiple variable files allows you to manage configurations and credentials separately. This modular approach enhances security and makes it easier to update specific settings without modifying the entire playbook. The Path to the files to include is a variable (path) for a more flexible approach.

Tasks:
Using import_tasks to include other task files promotes reusability and keeps the main playbook clean and readable. Each imported file can focus on a specific set of tasks, making it easier to debug and maintain.

### Task example

Included tasks are referenced playbooks looking similar to this example of the playbook creation_tasks:

```yaml
playbooks:
  - name: Deploy AWX Resources
    hosts: localhost
    tasks:
      - name: Create organizations
        awx.awx.organization:
          name: "{{ item.value.name }}"
          description: "{{ item.value.description }}"
          state: "{{ item.value.state }}"
          galaxy_credentials:
            - Ansible Galaxy
          validate_certs: false
        loop: "{{ lookup('dict', organizations, wantlist=True) }}"
        when: organizations is defined
[...]
```

These playbooks consist of multiple tasks to create awx resources or job templates. To be able to change specific values more easily most of the key-values are defined with a variable which itself is defined in a variable file section inside a dictionary block.

### values dictionary example

 The following snippet is an example of a variable file.

```yaml
vars:
  - name: organization-sthings
    file: |
      ---
      organizations:
        org1:
          name: "Organization 1"
          description: "Description for Organization 1"
          state: "present"
```

### template example for surveys

An AWX survey is a feature used to collect user input before running jobs in AWX. Surveys allow you to prompt users for information that can be used to customize the execution of AWX jobs. A survey consists of a series of questions. Each question can have different types, such as text, multiple choice, or password. Questions can be marked as required or optional. The answers provided in a survey are passed as variables to the Ansible playbook being executed.

```yaml
templates:
  - name: survey.json.j2
    file: |
      {
          "name": "",
          "description": "",
          "spec": [
          {% for key, question in questions.items() %}
          {
              "question_name": "{{ question.name }}",
              "question_description": "{{ question.description }}",
              "required": {{ question.required | lower }},
              "type": "{{ question.type }}",
              "variable": "{{ question.variable }}",
              "min": {{ question.min }},
              "max": {{ question.max }},
              {% if question.choices is defined %}
              "choices": {{ question.choices }},
              {% endif %}
              "default": "{{ question.default }}"
          }{% if not loop.last %},{% endif %}{% endfor %}
          ]
      }
```

### deployment order of AWX resources

When setting up AWX, it's crucial to follow a specific order of tasks to ensure that all dependencies are correctly established.

Start by defining your **organizations**. These are the top-level containers for all other resources in AWX, providing a structure for managing users and permissions.

Next, set up **machine credentials**. These are necessary for authenticating and accessing the machines where your playbooks will run.

After machine credentials, configure your **source control management (SCM) credentials**. These are used to access your playbook repositories.

Define any **custom credential types** that your organization requires. This allows you to create credentials tailored to specific needs.

With the custom types in place, you can now create the actual **custom credentials**.

Set up your **inventories**, which are collections of hosts against which jobs will be executed.

Define **inventory sources** to dynamically populate your inventories from external sources like cloud providers.

Add **hosts** to your inventories. These are the actual machines where your playbooks will run.

Create **projects** to organize and manage your playbooks. Projects link to your SCM repositories.

Define **execution environments**, which specify the runtime environment for your jobs, including the necessary dependencies and configurations.

Set up **job templates** to define the parameters for running your playbooks, including the inventory, project, and credentials to use.

Finally, create **workflow job templates** to chain multiple job templates together, allowing for complex automation workflows.

This order ensures that each resource is available and properly configured before it's needed by subsequent tasks. Following this sequence helps avoid errors and ensures a smooth setup process.

The AWX-Collection is structured in a way that these resources get created in exactly this order.

