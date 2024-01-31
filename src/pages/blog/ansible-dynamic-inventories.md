---
layout: "../../layouts/BlogPost.astro"
title: "Dynamic Ansible Inventories with Azure"
description: "Create an Azure VM with Azure CL"
pubDate: "Dec 7 2023"
heroImage: "/Ansible_logo.png"
previewText: "Ansible can automatically pull VMs from Azure to populate an inventory file. This is done using the azure_rm ansible plugin for dynamic inventory."
---

Ansible can automatically pull VMs from Azure to populate an inventory file. This is done using the [azure_rm ansible plugin](https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_inventory.html) for dynamic inventory.

It is really simple to set up.

```bash
$ python3.11 -m venv venv
$ . venv/bin/activate
$ pip install -U pip ansible azure-cli
$ ansible --version
ansible [core 2.16.1]
  config file = /Users/tpulliam/code/Ansible/ansible.cfg
  configured module search path = ['/Users/tpulliam/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/tpulliam/code/Ansible/venv/lib/python3.11/site-packages/ansible
  ansible collection location = /Users/tpulliam/.ansible/collections:/usr/share/ansible/collections
  executable location = /Users/tpulliam/code/Ansible/venv/bin/ansible
  python version = 3.11.1 (v3.11.1:a7a450f84a, Dec  6 2022, 15:24:06) [Clang 13.0.0 (clang-1300.0.29.30)] (/Users/tpulliam/code/Ansible/venv/bin/python3.11)
  jinja version = 3.1.2
  libyaml = True
$ ansible-galaxy collection list azure.azcollection

# /Users/tpulliam/code/Ansible/venv/lib/python3.11/site-packages/ansible_collections
Collection         Version
------------------ -------
azure.azcollection 1.19.0
```

In your `ansible.cfg` make sure to enable the azure_rm plugin.

```
[inventory]
enable_plugins = yaml, ini, azure_rm
```

You will then need to run `az login` to authenticate to Azure.

Your dynamic inventory filename must end with `azure_rm.yml`. Here is a simple dynamic inventory that groups VMs by Linux distribution.

###### inventory_azure_rm.yml

```yaml
plugin: azure.azcollection.azure_rm
auth_source: cli
include_vm_resource_groups:
  - "*"
conditional_groups:
  linux: os_profile.system == "linux"
  ubuntu: '"ubuntu" in image.offer'
  redhat: '"RHEL" in image.offer'
exclude_host_filters:
  # excludes hosts that are powered off
  - powerstate != 'running'
# change how inventory_hostname is generated. Each item is a jinja2 expression similar to hostvar_expressions.
hostnames:
  - name
```

```bash
(venv) tpulliam@lappy Ansible % ansible-inventory -i inventory.azure_rm.yml --graph
@all:
  |--@ungrouped:
  |--@linux:
  |  |--vm-rhel9-test
  |  |--vm-ubuntu-test
  |--@redhat:
  |  |--vm-rhel9-test
  |--@ubuntu:
  |  |--vm-ubuntu-test
```

If your VMs have a `vm_name` tag, you can use that as name of the VM in the inventory. That's pretty much it. See the documentation for more information.
