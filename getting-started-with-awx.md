# Getting started with AWX
![awx_logo](https://github.com/user-attachments/assets/51407902-6747-4e44-8f27-1f7428ee798f)


## What is AWX?
AWX is an open-source project that provides web-based user interface, REST API, and task enginer for Ansible. 
It is the upstread project from which the Red Hat Ansible Automation Platform is derived. AWX allows the user to manage and control Ansible Automation in a more efficient and user-friendly manner.

## Purpose of AWX
AWX is designed to help users and organizations manage the Ansible playbooks, inventories, credentials in a centralized and user-friendly manner. It provides a web-based interface that simplifies the Infrastructure Automation process and projects. AWX also made the collaboration for the teams easy through the use of structured approach to manage the Ansible Automation.

## Key Terminology
### Organizations
An Organization is a logical collection of Users, Teams, Projects and Inventories. And it is the highest level in the AWX hierarchy. It also provides a structured way to manage access and permissions within AWX, making it easier to control 'who can do what' within the system. 
### Playbooks
Playbooks are the YAML files that define a series of tasks to be executed by Ansible. They are used to automate complex workflows and configurations.
### Inventories
Inventories are lists of hosts or groups of hosts that Ansible manages. They can be defined in static files or dynamically generated from various sources.
### Credentials
Credentials in AWX are used to authenticate and authorize access to remote systems, for example authenticate the sign-in to the VM or a source control system. AWX seemless integration with various cloud, vault or secret service providers allows the users/teams/organizations to securely store and manage credentials for different environments and use cases. 
### Projects
Projects in AWX are collections of Ansible playbooks and related files. They can be sourced from various version control systems like Git, allowing for versioned and collaborative development of automation scripts.
### Jobs
Jobs are instances of AWX launching an Ansible playbook against an inventory of hosts. AWX allows the user to schedule, monitor, and manage jobs, providing detailed logs and status information.
### Job Templates
A job template is a definition and set of parameters for running an Ansible job. They include information about the playbook to run, the inventory to use, and any extra variables or options required for the execution.
Job templates are useful to execute the same job many times. 
### Workflows
Workflows are sequences of job templates that can be executed in a specific order, where the templates may or may not share inventory, playbooks, or permissions. They allow you to chain multiple automation tasks together, enabling more complex and conditional automation scenarios.


