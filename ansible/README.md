# Intro

Ansible is an infastructure automation and configuration management tool. It will prevent typos, allow for code review, and configuration as code is generally a good idea.

Ansible also provides a little bit of built in auditing since the developers name is next to the change.

Ansible gives you idemponcy. Where if you run an update two, three, or our times it will only change what is different.

## 4 aspects of a playbook

Packages (linux packages)
Service Handler (upstart, systemd, etc)
System configuration (yaml, users, etc)
App configuration

# Environment Setup

Ansible commands are executed from a control machine. For this tutorial I am using docker-machine, but any VM will work.

- [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)

# Ansible configuration file

You can create the main `ansible.cfg` file to specify some defaults

### Set the default inventory file

```
[defaults]
inventory = ./dev
```

# Inventory

There is a default ansible hosts file, in the tutorial it was not commented out, but on my machine it was.

This is located at `/etc/ansible/hosts` on the control machine.

You DO NOT want this in a production env because it contains a bunch of wrong domains.

### Create the file

You can create as many inventory files as you want. You can seperate them with .conf style headers if needed

```
vim inventory/dev
```

```
[app]
host1
host2

[database]
db1
db2
db3
```

### Defining local host

Ansible by default will attempt to SSH to the machine. On your control machine, you can specify this is a local connection like so.

```
[control]
controlHost ansible_connection=local
```

### Showing the hosts

Using the `dev` file we created earlier, we can now show the hosts like so

```
ansible -i inventory/dev --list-hosts all
```

# Host Selection

- [Patterns documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html)

You can glob hosts, or use wildcards to list specific hosts. This will be using the inventory examples above

```
ansible --list-hosts "*"
```

```
ansible --list-lists "db01"
```

```
ansible --list-hosts database,control,loadbalancer
```

# Tasks

- [Module Index](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html)

In general, you should not run ad-hoc commands and use playbooks instead, however this works and is a good way to learn the fundamentals.

You can run simple tasks on your inventory like so. Here is an example using the ping module.

```
ansible -m ping all
```

We can also run shell commands on the servers
Note: `command` is the default module, the below commands are the same.

```
ansible -m command -a "hostname" all
ansible -a "hostname" all
```

### Failures

You may have noticed output was green if you did it right. Output is red when you made a mistake.

```
ansible -a "/bin/false" all
```

# Plays

A playbook is a yaml file with a set of instructions, or "plays" of commands that will be executed on a server.

It allows you to run multiple commands from different modules all at once.

```
hosts: all
  tasks:
     - command: hostname
```

## Executing a playbook

```
ansible-playbook playbooks/hostname.yml
```

### GATHERING FACTS

When this is executed, the first thing ansible does is begin `GATHERING FACTS`

Where it will log into each server and generate `FACTS` about the server. Things like current configuration.

### TASK

These are the tasks that are performed, it should dislay what is happening on each machine.

### PLAY RECAP

This will show a recap of everything, including any errors.

# Facts

Facts are retreived on the hosts you connect to.

You can view the facts by running

```
ansible -m setup <host>
```

# Advanced ansible execution

### Getting the time of playbooks

```
time ansible-playbook <your-playbook>
```

### Run playbook on specific host

Let's say you took a server down for maintenance, you can run the playbook on it like so

```
ansible-playbook site.yml --limit <specific host>
```

### List available tags

```
ansible-playbook site.yml --list-tags
```

### Run available tags

```
ansible-playbook site.yml --tags "packages"
```

### Skip tags

```
ansible-playbook site.yml --skip-tags "packages"
ansible-playbook site.yml --skip-tags "packages,system,configure"
```

### step by step

```
ansible-playbook site.yml --step
```

### List tasks/start at task

```
ansible-playbook site.yml --list-tasks
ansible-playbook site.yml --start-at-task "some task name"
```

### Syntax Check / Dry Run

```
ansible-playbook --syntax-check <playbook>
```
