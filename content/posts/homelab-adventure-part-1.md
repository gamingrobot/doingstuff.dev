---
title: "Homelab Adventure - Part 1: The Adventure Begins"
date: "2019-11-09T12:00:00-05:00"
tags: ["homelab"]
---

Welcome to my journey in building my own homelab. This will hopefully be an ongoing series of blog posts of my adventures in building my own personal infrastructure.

<!--more-->

**Part 1: The Adventure Begins (You are here!)**  
**[Part 2: Lets Get Salty](/posts/homelab-adventure-part-2/)**

## What is a Homelab

The primary idea behind a homelab is a place to learn about infrastructure, servers and development.

/r/homelab defines a homelab as:

> Homelab \[hom-læb\](n): a laboratory of (usually slightly outdated) awesome in the domicile

Now by this definition a homelab should reside in a home. In my case part of my homelab will exist outside of my home, due to space and noise constraints. So my homelab is somewhere between a homelab and a "cloudlab".

The [/r/homelab wiki](https://www.reddit.com/r/homelab/wiki/introduction) has a more information on how to get started with a homelab.

## Inventory

At the time of writing here is my current inventory of servers. It will most likely grow and evolve as time goes on.

* 1 VPS
  * CPU: 2 Cores
  * RAM: 2GB
  * Disk: 40GB SSD
* 1 NAS
  * CPU: Intel i5-6600K
  * RAM: 8GB
  * Disk: 2x 250GB SSD, 4x 3TB WD Red
* 2 Dedicated Servers
  * Dedicated Server 1
    * CPU: 2x Intel Xeon L5420
    * RAM: 32GB
    * Disk: 2x 1TB HDD
  * Dedicated Server 2
    * CPU: Intel Xeon E5 1410 v2
    * RAM: 64GB
    * Disk: 3x 6TB HDD

## Goals

* Ability to host applications easily, there is [a lot of good self hosted software](https://github.com/awesome-selfhosted/awesome-selfhosted) out there.
* Keep ongoing maintenance to a minimum.
* Automatic encrypted backups, *because RAID is not a backup*
* Ability to easily add and remove servers.
* An internal network between servers, some of my servers exist outside of my home network.
* Alerting and Monitoring, so I know when servers go down, failing drives, backups didn't run, etc.

## The Plan

These are the tools/software I decided on to satisfy each of the above goals.

### Hosting Applications Easily

* [Docker](https://www.docker.com/) for hosting most applications and allows me to move services around easily.

* [Portainer](https://www.portainer.io/) for managing the docker containers on specific servers.

* [Salt](https://github.com/saltstack/salt) for everything else that doesn't fit into a container.

* [Traefik](https://traefik.io/) provides routing and certificates for services. 

### Minimizing Ongoing Maintenance

* [Automatic security updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates) will reduce some of the maintenance involved in updating.
* [Watchtower](https://github.com/containrrr/watchtower) will keep selected containers up to date.

### Automatic Backups

* [restic](https://restic.net/) for backups, including deduplication and encryption.
* [Amazon S3](https://aws.amazon.com/s3/)/[Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html) for offsite storage of backups.

### Adding and Removing Servers

* [Salt](https://github.com/saltstack/salt) allows me to have a declarative configuration for servers.
* [netboot.xyz](https://netboot.xyz/) simplifies installs with PXE booting.
* DNS for servers and services to facilitate container movement. This will be configured automatically or through [dnscontrol](https://github.com/StackExchange/dnscontrol).

### Internal Network

* [Zerotier](https://www.zerotier.com/) is for my internal network. It's not a fully decentralized like [tinc](https://www.tinc-vpn.org/) but has some trade offs that make it easier to use, you can read more about it [here](http://adamierymenko.com/decentralization.html).

### Alerting and Monitoring

* [netdata](https://github.com/netdata/netdata) will be used for metrics and basic alerting.
* [alerta](https://alerta.io/) will be used for custom alerting.

## Here we go!

I hope you will join me in this adventure into homelabbing. In the next post in the series we will cover configuration management.

**Part 1: The Adventure Begins (You are here!)**  
**[Part 2: Lets Get Salty](/posts/homelab-adventure-part-2/)**