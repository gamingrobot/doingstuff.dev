---
title: "Homelab Adventure - Part 1: The Adventure Begins"
description: Homelab overview
date: 2019-11-09
slug: homelab-adventure-part-1
taxonomies: 
  tags: ["Linux", "Homelab"]
---

Welcome to my journey in building my Homelab. This will be an ongoing series of blog posts of my adventures in building my personal infrastructure.

<!-- more -->

**Part 1: The Adventure Begins (You are here!)**  
[**Part 2: Configuration Management**](@/posts/homelab-adventure-part-2.md)  
[**Part 3: Internal Network**](@/posts/homelab-adventure-part-3.md)  
[**Sidequest: Switching from Salt to Ansible**](@/posts/homelab-switching-salt-to-ansible.md)   
[**Part 4: Application Hosting and Monitoring**](@/posts/homelab-adventure-part-4.md)  

## What is a Homelab

The primary idea behind a homelab is a place to learn about infrastructure, servers and development.

/r/homelab defines a homelab as:

> Homelab \[hom-læb\](n): a laboratory of (usually slightly outdated) awesome in the domicile

Now by this definition a Homelab should reside in a home. In my case part of my Homelab will exist outside of my home, due to space and noise constraints. So my Homelab is somewhere between a Homelab and a "cloudlab".

The [/r/homelab wiki](https://www.reddit.com/r/homelab/wiki/introduction) has a more information on how to get started with a Homelab.

## Inventory

At the time of writing here is my current inventory of servers. It will most likely grow and evolve as time goes on.

- 1 NAS
  - CPU: Intel i5-6600K
  - RAM: 32GB
  - Disk: 2x 250GB SSD, 4x 12TB HDD
- 2 Dedicated Servers
  - Dedicated Server 1
    - CPU: Intel
    - RAM: 64GB
    - Disk: 2x 512GB SSD
  - Dedicated Server 2
    - CPU: Intel
    - RAM: 32GB
    - Disk: 2x 3TB HDD

## Goals

- Ability to host applications easily, there is [a lot of good self hosted software](https://github.com/awesome-selfhosted/awesome-selfhosted) out there.
- Keep ongoing maintenance to a minimum.
- Automatic encrypted backups, _because RAID is not a backup_
- Ability to easily add and remove servers.
- An internal network between servers, some of my servers exist outside of my home network.
- Alerting and Monitoring, so I know when servers go down, failing drives, backups didn't run, etc.

## The Plan

These are the tools/software I decided on to meet each of the above goals.

{%info()%}The crossed out tools are things that have changed or been replaced since I wrote this in 2019.{%end%}

### Hosting Applications

- [Docker](https://www.docker.com/) for hosting most applications and allows me to move services around.
- [Ansible](https://www.ansible.com/) for managing containers and configuration.
- [Traefik](https://traefik.io/) provides routing and certificates for services.
- ~~[Portainer](https://www.portainer.io/) for managing the docker containers on specific servers.~~
- ~~[Salt](https://github.com/saltstack/salt) for everything else that doesn't fit into a container.~~

### Minimizing Ongoing Maintenance

- [Automatic security updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates) will reduce some of the maintenance involved in updating.
- [Watchtower](https://github.com/containrrr/watchtower) will keep selected containers up to date.

### Automatic Backups

- [restic](https://restic.net/) for backups, including deduplication and encryption.
- [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html) for offsite storage of backups.

### Adding and Removing Servers

- ~~[Salt](https://github.com/saltstack/salt)~~ [Ansible](https://www.ansible.com/) allows me to have a declarative configuration for servers.
- [netboot.xyz](https://netboot.xyz/) simplifies installs with PXE booting.
- [dnscontrol](https://github.com/StackExchange/dnscontrol) for DNS management.

### Internal Network

- [Zerotier](https://www.zerotier.com/) is for my internal network. It's not decentralized like [tinc](https://www.tinc-vpn.org/) but has some trade offs that make it easier to use, you can read more about it [here](http://adamierymenko.com/decentralization.html).

### Alerting and Monitoring

- [Netdata](https://github.com/netdata/netdata) will be used for metrics and basic alerting.
- [Uptime Kuma](https://github.com/louislam/uptime-kuma) will be used for custom monitoring/alerting.
- ~~[Icinga2](https://icinga.com/docs/icinga2/latest/) will be used for alerting.~~
- ~~[InfluxDB](https://www.influxdata.com/time-series-platform/influxdb/) is a time series database used for storage of metrics.~~
- ~~[Grafana](https://grafana.com/) is used for making graphs and dashboards from the data stored in influxdb.~~

## Here we go!

I hope you will join me in this adventure into homelabbing. In the next post in the series we will cover configuration management.

**Part 1: The Adventure Begins (You are here!)**  
[**Part 2: Configuration Management**](@/posts/homelab-adventure-part-2.md)  
[**Part 3: Internal Network**](@/posts/homelab-adventure-part-3.md)  
[**Sidequest: Switching from Salt to Ansible**](@/posts/homelab-switching-salt-to-ansible.md)   
[**Part 4: Application Hosting and Monitoring**](@/posts/homelab-adventure-part-4.md)  
