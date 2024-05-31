+++
title = "Homelab Sidequest - Switching from Salt to Ansible"
description = "Switching my Homelab from Salt to Ansible"
date = 2024-03-20
slug = "homelab-switching-salt-to-ansible"
[taxonomies]
tags = ["Homelab"]
+++

About a month ago I thought it would be nice to be able to configure VM's and Droplets with Salt by using my existing configuration I had setup in [Part 2: Configuration Management](@/posts/homelab-adventure-part-2.md).

<!-- more -->

I could have gone the normal route of installing the Salt minion and attaching it to the Salt master then running the highstate. But since these would be resource constrained machines I had concerns about the resource usage of the minion.

So I decided to try Salt SSH, which uses ssh to transfer a lightweight minion that executes the states and then exits. It would also allow me to to use Salt from my local machine instead of remoting into the server hosting the Salt master.

This worked great until I attempted to run it and hit this [lovely bug](https://github.com/saltstack/salt/issues/60002) where if you are using gpg-encrypted pillars the minion will attempt to decrypt them rather than decrypting locally and when rendering the state.

This wasn't the first time I had run into a nasty Salt bug, [this bug](https://github.com/saltstack/salt/issues/47325) caused watchtower to break and I had to rollback to a previous version.

## Giving Ansible a try

I read Jeff Geerling's [Ansible for DevOps](https://www.jeffgeerling.com/project/ansible-devops) book, which gave a really good overview of Ansible and how I could map what I knew from Salt to Ansible. I also looked for blog posts of people migrating from Salt to Ansible, but almost all of them were company blogs talking about how they outgrew Ansible and migrated to Salt.

I ported some of my simpler states to Ansible to try and get a feel for how my Salt traits and states mapped to Ansible playbooks and roles.

These are the Ansible concepts explained in terms of Salt:
* Playbook
  * Ansible playbooks are the mapping between host and roles/tasks.
  * Salt's top file is also a mapping between hosts to states.
* Role
  * Ansible roles contain tasks, files, variables and handlers.
  * Salt formulas contain states, files and variables.
* Variables
  * Ansible has two types of vars `group_vars`and `host_vars`. `group_vars` are scoped to the group in the inventory while `host_vars` are scoped to a specific host.
  * Salt pillars contain the variables specific to a host. Pillars can contain gpg encrypted values for Ansible it encrypts the whole yaml via `ansible-vault`.
* Handlers
  * Ansible handlers are tasks but are triggered via `notify`, these most closely map to Salt Reactors. But are primarily used to achieve the same result that Salt's `on_change` does to have states that trigger off other states changing.
* Inventory
  * Ansible needs to know what hosts its going to be connecting to.
  * Salt's master keeps track of the connected minions and in Salt SSH it uses a roster.
* Galaxy
  * Ansible Galaxy is a repository of roles created by the community.
  * Salt's version of this is the [saltstack-formulas](https://github.com/saltstack-formulas) repos.

## After porting 1000 lines of yaml

After porting all my Salt states to Ansible there were a few things that tripped me up.

Salt allows for jinja templating inside state files, instead Ansible only allows jinja templating inside yaml values and template files. This affects how loops are done, in Salt they are handled with a jinja for loop, but in Ansible you use a `loop` and `loop_control` in the yaml.

#### Salt

```jinja
{% for name in salt['pillar.get']('certs', {}).keys() %}
cert-{{ name }}:
  file.managed:
    - name: /opt/certs/{{ name }}.cert
    - contents_pillar: certs:{{ name }}:cert
{% endfor %}
```

#### Ansible

```yaml
- name: cert
  ansible.builtin.copy:
    dest: /opt/certs/{{ item.key }}.cert
    content: "{{ item.value.cert }}"
  loop: "{{ certs | dict2items }}"
  loop_control:
    label: "{{ item.key }}"
```

With Salt you can specify the source of a local file via `salt://{{ tpldir }}`. In Ansible when specifying a local file it will first look in a role's `files` folder.

#### Salt

```yaml
dynamic-config:
  file.managed:
    - name: /opt/config/dynamic.yaml
    - source: salt://{{ tpldir }}/config/dynamic.yaml
```

#### Ansible

```yaml
- name: dynamic-config
  ansible.builtin.template:
    src: dynamic.yaml
    dest: /opt/config/dynamic.yaml
```

Salt has targeting as part of the `state.apply` command, in Ansible you can limit the scope of the playbook by using `--limit`

```console
$ salt 'testserver' state.apply
$ ansible-playbook servers.yml --limit testserver
```

## Living with Ansible for a few months

### Things I liked

I really like that when you hit `Ctrl+C` during execution Ansible stops the playbook execution. This is nice when I have forgotten to make a change or something failed on one host but not another. With Salt you get a job id that you have to cancel manually or let it finish.

Because I am running Ansible locally, iteration time is quicker. In Salt I am committing then pushing my changes, pulling them down on the master and running highstate. 

Documentation is considerably better for Ansible. The documentation pages include multiple examples for a module. Since many other homelabbers run Ansible there are many blog posts and examples on how to do something.

Ansible Vault is much easier to use than gpg encrypting pillar data. By default it encrypts entire files but you can encrypt specific values like Salt's gpg pillars.

### Things I disliked

The roles folder structure is verbose but only requires the folders you use.

The largest complaint I saw about Ansible was how slow it was compared to Salt. I saw speeds similar to Salt with this additional configuration:

```toml
[defaults]
strategy = free # don't block on each host per task
[ssh_connection]
pipelining = True # reduce the number of network operations
```
