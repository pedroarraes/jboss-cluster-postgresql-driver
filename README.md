# Install Postgresql drivers on JBoss EAP cluster using ansible
This Ansible script is designed to efficiently download and install both XA and non-XA PostgreSQL drivers on a JBoss EAP cluster.

## Requeriments

* Bastion Server.
* Host 1 - A Virtual Machine RHEL 9.2 for Domain Controller Server.
* Hosts 2, 3, 4 - Virtual Machines RHEL 9.2 for Work Nodes.
* Ansible [core 2.14.8].
* JBoss EAP 7.4.

## Summary
* [Downloading JBoss EAP](#downloading-jboss-eap)
  * [Installer](#installer)
  * [Patches](#patches)
* [Ansible Script](#ansible-script)
  * [Folder group_vars](#folder-group_vars)
  * [Folder packages](#folder-packages)
  * [Folder roles](#folder-roles)
  * [Files install.yml and host](#files-installyml-and-host)
* [Cluster EAP](#cluster-eap)
* [Load Balancer](#load-balancer)
* [Executing script](#executing-script)

This section outlines the steps to download JBoss EAP and apply patches for updates.

## Ansible Script

The Ansible script is structured into three primary folders, group_vars, packages, roles:
.</br>
├── ansible.cfg</br>
├── files</br>
│   └── module.xml</br>
├── group_vars</br>
│   └── all.yml</br>
├── hosts</br>
├── install.yml</br>
├── LICENSE</br>
├── README.md</br>
└── roles</br>
    ├── domain</br>
    │   └── tasks</br>
    │       └── main.yml</br>
    └── workers</br>
        └── tasks</br>
            └── main.yml</br>
</br>
8 directories, 9 files</br>


### Folder group_vars

In the "group_vars" folder, we house global environment variables. These variables are defined in the "all.yml" file, allowing you to modify default values as per your specific requirements.

```yaml
# This file contains the variables that are common to all the hosts
POSTGRESQL_DRIVER_VERSION: 42.2.5
POSTGRESQL_DRIVER_URL: https://jdbc.postgresql.org/download/postgresql-{{ POSTGRESQL_DRIVER_VERSION }}.jar
JBOSS_HOME: /opt/jboss-eap-7.4
JBOSS_USER: jboss-eap
```

### Folder roles

The Ansible scripts are structured into roles, including:

* domain: This role facilitates the creation of PostgreSQL drivers on a domain controller.
* workers: This role manages the download and installation of PostgreSQL modules on JBoss EAP worker nodes.

All roles can be adapted as needed, and new custom roles can be created and utilized in the install.yml file located in the project's root folder. Let's examine the contents of the install.yml file:

### Files install.yml and host


In the **install.yml** file, you have the ability to declare additional roles to enhance the script's functionality:

```yaml
- hosts: all
  remote_user: root
  become: yes
  become_method: sudo
  roles:
    - workers

- hosts: domaincontroller
  remote_user: root
  become: yes
  become_method: sudo
  roles:
    - domain
```

In the **hosts** file, you can declare additional virtual machines to scale and expand your cluster:

```yaml
[domaincontroller]
domaincontroller ansible_ssh_pass=1234 ansible_ssh_user=root

[all]
domaincontroller ansible_ssh_pass=1234 ansible_ssh_user=root
host01 ansible_ssh_pass=1234 ansible_ssh_user=root
host02 ansible_ssh_pass=1234 ansible_ssh_user=root
host03 ansible_ssh_pass=1234 ansible_ssh_user=root
```

## Executing script

To execute the playbook, follow these steps:
1. Download the repository on the bastion server.
2. Access the main project folder within the downloaded repository.
3. Execute the Ansible playbook using the ansible-playbook command.

Make sure to have the necessary permissions and prerequisites installed on the bastion server before proceeding.

Example:

```bash
ansible-playbook -i hosts install.yml --ask-become-pass
```
```console
BECOME password: 
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [all] ***********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************
ok: [domaincontroller]
ok: [host02]
ok: [host03]
ok: [host01]

TASK [workers : Create directory for Postgresql driver] **************************************************************************************************************************************************************************************
ok: [host03]
ok: [domaincontroller]
ok: [host02]
ok: [host01]

TASK [workers : Download Postgresql driver to host] ******************************************************************************************************************************************************************************************
ok: [host03]
ok: [host02]
ok: [domaincontroller]
ok: [host01]

TASK [workers : Create module.xml for Postgresql driver] *************************************************************************************************************************************************************************************
ok: [domaincontroller]
ok: [host03]
ok: [host02]
ok: [host01]

PLAY [domaincontroller] **********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************
ok: [domaincontroller]

TASK [domain : Create Postgresql Non-XA Driver in profile full-ha] ***************************************************************************************************************************************************************************
changed: [domaincontroller]
Omitted
```

