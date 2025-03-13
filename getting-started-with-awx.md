# Getting started with AWX
<img src="https://github.com/user-attachments/assets/51407902-6747-4e44-8f27-1f7428ee798f" align="right" width="300">

## What is AWX?
AWX is an open-source project that provides web-based user interface, REST API, and task enginer for Ansible. 
It is the upstread project from which the Red Hat Ansible Automation Platform is derived. AWX allows the user to manage and control Ansible Automation in a more efficient and user-friendly manner.

## Purpose of AWX
AWX is designed to help users and organizations manage the Ansible playbooks, inventories, credentials in a centralized and user-friendly manner. It provides a web-based interface that simplifies the Infrastructure Automation process and projects. AWX also made the collaboration for the teams easy through the use of structured approach to manage the Ansible Automation.

## Key Terminology
|         |                                                                 |
|---------|-----------------------------------------------------------------|
| Organizations    | An Organization is a logical collection of Users, Teams, Projects and Inventories. And it is the highest level in the AWX hierarchy. It also provides a structured way to manage access and permissions within AWX, making it easier to control `who can do what` within the system. |
| Playbooks | Playbooks are the YAML files that define a series of tasks to be executed by Ansible. They are used to automate complex workflows and configurations. |
| Inventories | Inventories are lists of hosts or groups of hosts that Ansible manages. They can be defined in static files or dynamically generated from various sources. |
| Credentials| Credentials in AWX are used to authenticate and authorize access to remote systems. AWX seemless integration with various cloud, vault or secret service providers allows the users/teams/organizations to securely store and manage credentials for different environments and use cases. |
| Projects | Projects in AWX are collections of Ansible playbooks and related files. They can be sourced from various version control systems like Git, allowing for versioned and collaborative development of automation scripts. |
| Jobs | Jobs are instances of AWX launching an Ansible playbook against an inventory of hosts. AWX allows the user to schedule, monitor, and manage jobs, providing detailed logs and status information. |
| Job Templates | A job template is a definition and set of parameters for running an Ansible job. They include information about the playbook to run, the inventory to use, and any extra variables or options required for the execution. Job templates are useful to execute the same job many times. |
| Workflows | Workflows are sequences of job templates that can be executed in a specific order, where the templates may or may not share inventory, playbooks, or permissions. They allow you to chain multiple automation tasks together, enabling more complex and conditional automation scenarios. |
| Execution Environments | Execution Environments in AWX are containerized environments that provide the necessary dependencies and runtime for executing Ansible playbooks. They ensure consistency and isolation of the execution context, allowing users to define and manage the specific versions of Ansible and other required tools. Execution Environments can be customized and shared across different teams and projects, facilitating a standardized and reproducible automation process. |

## Code example to create awx resources

### Directory tree

awx_deployment/
├── playbook.yml
├── templates/
│   ├── survey.json.j2
├── vars/
│   ├── organizations.yml
│   ├── machine_credentials.yml
│   ├── scm_credentials.yml
│   ├── custom_credential_types.yml
│   ├── custom_credentials.yml
│   ├── inventories.yml
│   ├── inventory_sources.yml
│   ├── hosts.yml
│   ├── projects.yml
│   ├── execution_environments.yml
│   ├── job_templates.yml
│   ├── workflow_job_templates.yml

### playbook example

```yaml
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

### values dictionary example

vars/

```yaml
organizations:
  org1:
    name: "Organization 1"
    description: "Description for Organization 1"
    state: "present"
```

### template example for surveys

templates/

```yaml
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