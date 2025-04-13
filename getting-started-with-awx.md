# Getting started with AWX in KIND
<img src="https://github.com/user-attachments/assets/51407902-6747-4e44-8f27-1f7428ee798f" align="right" width="300">

## What is AWX?
AWX is an open-source project that provides web-based user interface, REST API, and task enginer for Ansible. 
It is the upstread project from which the Red Hat Ansible Automation Platform is derived. AWX allows the user to manage and control Ansible Automation in a more efficient and user-friendly manner.

## Purpose of AWX
AWX is designed to help users and organizations manage the Ansible playbooks, inventories, credentials in a centralized and user-friendly manner. It provides a web-based interface that simplifies the Infrastructure Automation process and projects. AWX also made the collaboration for the teams easy through the use of structured approach to manage the Ansible Automation.

## Key Terminology

Understanding the key terminology in AWX is crucial for effectively using the platform:

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

## Why AWX in KIND?

AWX is generally used to manage and automate IT infrastructure, providing a centralized platform for running Ansible playbooks, managing inventories, and handling credentials. However, this blog focuses on using AWX as a developer setup to test and learn. By leveraging KIND (Kubernetes IN Docker), developers can create a local, isolated environment to experiment with AWX without the need for extensive infrastructure. Here are the pros:<br>

**Simplicity and Ease of Setup:**<br>
Kind is straightforward to install and configure, making it accessible for beginners. You can quickly spin up a Kubernetes cluster without needing extensive infrastructure or complex setup procedures.<br>

**Local Development:**<br>
Since kind runs Kubernetes clusters in Docker containers, you can run it locally on your development machine. This eliminates the need for cloud resources or remote servers, making it cost-effective and convenient.<br>

**Isolation and Safety:**<br>
Kind allows you to create isolated Kubernetes clusters. This means you can experiment and test AWX without affecting other environments or production systems. It's a safe playground for learning and testing.<br>

**Consistency with Production:**<br>
Kind provides a Kubernetes environment that is consistent with production clusters. This helps new users understand how AWX will behave in a real Kubernetes environment, ensuring that what they learn and test locally will be applicable in production.<br>

**Resource Efficiency:**<br>
Running Kubernetes clusters in Docker containers is resource-efficient. Kind clusters consume fewer resources compared to running full-fledged virtual machines, making it suitable for machines with limited resources.<br>

**Community and Support:**<br>
Kind is widely used and supported by the Kubernetes community. There are plenty of resources, tutorials, and community support available to help new users get started and troubleshoot any issues they encounter.<br>

## Install requirements

This file specifies the roles and collections that Ansible will use to perform various tasks.

```yaml
cat <<EOF > requirements.yaml
---
roles:
  - src: https://github.com/stuttgart-things/deploy-configure-rke.git
    scm: git
  - src: https://github.com/stuttgart-things/install-configure-powerdns.git
    scm: git
  - src: https://github.com/stuttgart-things/configure-rke-node.git
    scm: git
  - src: https://github.com/stuttgart-things/install-requirements.git
    scm: git
  - src: https://github.com/stuttgart-things/create-send-webhook.git
    version: 2024.06.06
    scm: git
  - src: https://github.com/stuttgart-things/install-configure-vault.git
    scm: git
  - src: https://github.com/stuttgart-things/install-configure-nfs.git
    scm: git
  - src: https://github.com/stuttgart-things/manage-filesystem.git
    version: 2024.06.07
    scm: git
  - src: https://github.com/stuttgart-things/download-install-binary.git
    scm: git
    version: 2025.03.27
  - src: https://github.com/stuttgart-things/create-os-user.git
    scm: git
    version: 2024.05.03
  - src: https://github.com/stuttgart-things/install-configure-docker.git
    scm: git
    version: 2025.10.04
  - src: https://github.com/stuttgart-things/install-configure-podman.git
    scm: git
    version: 2024.05.08
  - src: https://github.com/stuttgart-things/manage-proxmox-resources.git
    scm: git
    version: 2024.05.27

collections:
  - name: community.crypto
    version: 2.26.0
  - name: community.general
    version: 10.5.0
  - name: ansible.posix
    version: 2.0.0
  - name: kubernetes.core
    version: 5.2.0
  - name: community.docker
    version: 4.5.2
  - name: community.vmware
    version: 5.5.0
  - name: awx.awx
    version: 24.6.1
  - name: community.hashi_vault
    version: 6.2.0
  - name: ansible.netcommon
    version: 7.2.0
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-container-25.6.528.tar.gz/sthings-container-25.6.528.tar.gz
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-25.6.530.tar.gz/sthings-baseos-25.6.530.tar.gz
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-awx-25.4.1190/sthings-awx-25.4.1190.tar.gz
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-rke-25.1.568.tar.gz/sthings-rke-25.1.568.tar.gz
EOF
```

To install the specified collections and roles, use the following command:

```bash
ansible-galaxy collection install -r ./requirements.yaml -f
```

This command will download and install all the necessary collections and roles defined in the requirements.yaml file, ensuring that your Ansible environment is ready for the subsequent tasks to install KIND and AWX.

## Create KIND and AWX

### Inventory file

This inventory file defines the hosts and groups of hosts that Ansible will manage. The [group1] section lists the IP addresses or hostnames of the remote machines, along with the Ansible user and password for authentication (definable after each host entry or in the defaults section). Alternatively you can only specify one host. The [defaults] section sets global variables, such as disabling host key checking and specifying the Ansible user and password.

Example Inventory file:

```yaml
cat <<EOF > inventory
# Host group
[group1]
<xxx.xxx.x.xx> ansible_user=<user> ansible_password=<password>
<xxx.xxx.x.xx>

# Only one host
<xxx.xxx.x.xx> # Enter IP of the remote machine you want to use

# Variables
[defaults]
host_key_checking = False
ansible_user = <name> # Enter Ansible User
ansible_password = <password> # Enter Ansible User Password
EOF
```

### Install KIND and AWX playbook example

Now that you have your inventory file ready, you can proceed with the playbook to install KIND and AWX. This playbook includes various tasks, each broken down into separate playbooks for better manageability and clarity.

```yaml
- name: Base setup
  ansible.builtin.import_playbook: sthings.baseos.setup
  # vars:
  #   key: value

- name: Install binaries
  ansible.builtin.import_playbook: sthings.baseos.binaries
  # vars:
  #   key: value

- name: Install ansible
  ansible.builtin.import_playbook: sthings.baseos.ansible
  # vars:
  #   key: value

- name: Install golang
  ansible.builtin.import_playbook: sthings.baseos.golang
  # vars:
  #   key: value

- name: Install pre_commit
  ansible.builtin.import_playbook: sthings.baseos.pre_commit
  # vars:
  #   key: value

- name: Install semantic_release
  ansible.builtin.import_playbook: sthings.baseos.semantic_release
  # vars:
  #   key: value

- name: Install docker
  import_playbook: sthings.container.docker
  # vars:
  #   key: value

- name: Install container tools
  import_playbook: sthings.container.tools
  # vars:
  #   key: value

- name: Install nerdctl
  import_playbook: sthings.container.nerdctl
  # vars:
  #   key: value

- name: Install podman
  import_playbook: sthings.container.podman
  # vars:
  #   key: value

- name: Reboot
  import_playbook: sthings.container.reboot_vm
  # vars:
  #   key: value

- name: Execute dev config
  import_playbook: sthings.baseos.dev_config
  # vars:
  #   key: value

- name: Create kind dev cluster
  import_playbook: sthings.container.kind
  vars:
    kind_cluster_name: dev
    ansible_user: sthings
    count_worker_nodes: 3
    count_controlplane_nodes: 1
    kubectl_version: 1.32.3

- name: Deploy cilium on dev kind cluster
  import_playbook: sthings.container.deploy_to_k8s
  vars:
    profile: cilium-kind
    state: present
    path_to_kubeconfig: /home/{{ ansible_user }}/.kube/{{ kind_cluster_name }}
    target_host: all
    cluster_name: "{{ kind_cluster_name }}"

- name: Deploy cert-manager on dev kind cluster
  import_playbook: sthings.container.deploy_to_k8s
  vars:
    profile: cert-manager-kind
    state: present
    path_to_kubeconfig: /home/{{ ansible_user }}/.kube/{{ kind_cluster_name }}
    target_host: all
    cluster_name: "{{ kind_cluster_name }}"

- name: Deploy ingress-nginx on dev kind cluster
  import_playbook: sthings.container.deploy_to_k8s
  vars:
    profile: ingress-nginx-kind
    state: present
    path_to_kubeconfig: /home/{{ ansible_user }}/.kube/{{ kind_cluster_name }}
    target_host: all
    cluster_name: "{{ kind_cluster_name }}"

- name: Set fact Controlplane IP
  import_playbook: sthings.container.get_controlplane_ip
  vars:
    kind_cluster_name: dev
    ansible_user: <user>

- name: Deploy awx-operator and awx-instance on kind cluster
  import_playbook: sthings.container.deploy_to_k8s
  vars:
    profile: awx-operator-kind
    state: present
    path_to_kubeconfig: /home/{{ ansible_user }}/.kube/{{ kind_cluster_name }}
    target_host: all
    cluster_name: "{{ kind_cluster_name }}"
    control_plane_ip: "{{ control_plane_ip }}"

```

In this setup, each task is broken down into separate playbooks, making it easier to manage and understand. The playbooks are named clearly, indicating their purpose, which helps users follow the sequence of tasks effortlessly. By importing playbooks, the reusability is significantly increased, allowing them to be used for multiple purposes. Additionally, the use of variables enables easy customization of the playbooks for different environments or requirements.

From a technical perspective, each playbook handles a specific aspect of the setup, adhering to the principle of separation of concerns. This approach enhances maintainability and simplifies debugging. The structure is designed for scalability; adding more tools or configurations can be achieved by simply incorporating additional playbooks or expanding the variables sections. Furthermore, the setup automates the installation and configuration of various tools and services, minimizing the potential for human error and ensuring a consistent environment.

### Execute Collection

```bash
ansible-playbook sthings.baseos.dev -i inventory -vv
```

This command runs the playbook sthings.baseos.dev against the hosts defined in your inventory file, with verbose output enabled (-vv). This will provide detailed logs and status information, helping you monitor the execution process and troubleshoot any issues that arise.

By following these steps, you can set up a local development environment with AWX and KIND, allowing you to test and learn Ansible automation in a safe and isolated environment. This setup is ideal for developers who want to experiment with AWX without the need for extensive infrastructure, making it a cost-effective and convenient solution.

In Blog Part 2 we will explain how to setup AWX Resources and how to create the first jobs and workflows.


#########################################################################
#########################################################################
#########################################################################

# PART 2 Vorarbeit (Bitte vorerst ignorieren)

#########################################################################
#########################################################################
#########################################################################

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

