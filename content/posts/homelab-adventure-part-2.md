+++
title = "Homelab Adventure - Part 2: Configuration Management"
date = 2020-02-06
slug = "homelab-adventure-part-2"
[taxonomies]
tags = ["Homelab"]
+++

Welcome to my journey in building my homelab. This is part of a multipart series, in the last part I gave an overview of the homelab plan. This one will cover how I handle configuration management.

<!-- more -->

[**Part 1: The Adventure Begins**](/posts/homelab-adventure-part-1/)  
**Part 2: Configuration Management (You are here!)**  
[**Part 3: Internal Network**](/posts/homelab-adventure-part-3/)  

## What is Salt

> [Salt](https://github.com/saltstack/salt) is a configuration management and orchestration tool.

What this means is it allows me to declare a "state" that the infrastructure should be in. It also allows me to run commands/modules across the infrastructure. Some alternatives to Salt are [Ansible](https://www.ansible.com/), [Chef](https://www.chef.io/configuration-management/), [Puppet](https://puppet.com/).

## How I am using Salt

Salt is the base of my infrastructure, once a server is installed Salt then provisions the rest it.

The idea is anything not in docker is configured by Salt. That way when a server needs to be reinstalled or added, I know configuration isn't missing due to manual changes.

### Folder Structure

This is a view of my Salt configuration git repo, some files/directories are missing from this view for the sake of readability. It is useful to see an example of how other people have laid out their Salt configuration when setting up your own.

```
salt
├── _modules
│   └── zerotier.py
├── _states
│   └── zerotier.py
├── app
│   ├── at
│   ├── btrfs
│   ├── docker
│   ├── fail2ban
│   ├── ntp
│   ├── nullmailer
│   ├── powerpanel
│   ├── restic
│   ├── smart
│   └── zerotier
├── config
│   ├── apt
│   ├── salt-minion
│   ├── ssh
│   ├── timezone
│   └── users
├── container
│   ├── portainer
│   ├── portainer-agent
│   ├── salt-master
│   ├── traefik
│   └── watchtower
├── trait
│   ├── backup.sls
│   ├── battery-backup.sls
│   ├── btrfs.sls
│   ├── docker-extra.sls
│   ├── docker-master.sls
│   ├── docker-proxy.sls
│   ├── docker.sls
│   ├── internal-network.sls
│   ├── mailer.sls
│   └── salt-master.sls
├── default-packages.sls
└── top.sls
pillar
├── app
│   ├── docker
│   ├── restic
│   ├── nullmailer.sls
│   ├── portainer.sls
│   ├── traefik.sls
│   └── zerotier.sls
├── config
│   ├── network.sls
│   └── users.sls
├── default-schedule.sls
└── top.sls
```

#### Salt

##### _module/_states

These are custom modules and states that are synced to minions. I will cover the custom zerotier module/state in another blog post.

##### App

These are states that require the installation of a package. The configuration of the application also lives here.

##### Config

These are states that are pure configuration and nothing new is installed.

##### Container

These are docker containers that are either going to be on every server (watchtower) or are required to bootstrap other containers (portainer).

##### Trait

Salt has many ways for targeting states to a machine in the `top.sls` file. The `top.sls` file is what Salt looks at when you run highstate for what states should be applied to a minion.

The Salt community has this idea of using grains to apply roles to a server called [Role-based infrastructure](http://www.saltstat.es/posts/role-infrastructure.html). This works if the server has a specific role like `database` or `webserver`.

Traits are a similar idea but are more targeted at what you need on the server and are more fine-grained than `database` or `webserver`.

This setup allows me to have a very small initial highstate and then apply traits as I need them for specific servers.

##### default-packages.sls

This is a list of helpful packages that are installed by default on all minions.

```yaml
install-default-packages:
  pkg.installed:
    - pkgs:
      - rsync
      - curl
      - vim
      - htop
      - iotop
      - git
      - ncdu
      - openssh-client
      - ncurses-term # for putty profiles
      - traceroute
      - jq
      - virt-what # for salt virtual grains
      - net-tools # ifconfig/route
```

##### top.sls

This is the file looked at when highstate is run. As you can see I have a very simple base that gets installed by default and then everything else is defined by traits.

```yaml
base:
  '*':
    - default-packages
    - config.users
    - config.timezone
    - config.salt-minion
    - config.ssh
    - config.apt
    - app.at
    - app.ntp
    - app.fail2ban

{% set traits = salt['grains.get']('traits', []) %}
{% for trait in traits %}
  'traits:{{ trait }}':
    - match: grain
    - trait.{{ trait }}
{% endfor %}
```

#### Pillar

This is where the any machine specific configuration and sensitive data lives.

I store both my Salt states and pillar data in the same git repository so I can test and update configurations. Any secrets in the pillars are PGP encrypted using Salt's [gpg renderer](https://docs.saltstack.com/en/latest/ref/renderers/all/salt.renderers.gpg.html). Salt supports [many pillar sources](https://docs.saltstack.com/en/master/ref/pillar/all/index.html) and they can even be used in combination.

##### top.sls

This is the pillar top file which defines what minions have access to specific pillar data.

```yaml
base:
  '*':
    - default-schedule
    - config.users
    - config.network

  'traits:internal-network':
    - match: grain
    - app.zerotier

  'traits:mailer':
    - match: grain
    - app.nullmailer

  'traits:backup':
    - match: grain
    - app.restic.base
    - app.restic.{{ grains.id }}

  'traits:docker-proxy':
    - match: grain
    - app.traefik

  'traits:docker-extra':
    - match: grain
    - app.portainer

  'traits:docker-master':
    - match: grain
    - app.portainer
```

{% warning() %}
Grains are controlled by the minion, so a server can access pillar data for other traits due to matching on grains.
{% end %}

## Maximum Saltiness

I hope this gave you a good view of how I use Salt to configure my servers. In the next part we will cover how to have an internal network over the internet.

[**Part 1: The Adventure Begins**](/posts/homelab-adventure-part-1/)  
**Part 2: Configuration Management (You are here!)**  
[**Part 3: Internal Network**](/posts/homelab-adventure-part-3/)  
