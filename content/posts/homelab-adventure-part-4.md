+++
title = "Homelab Adventure - Part 4: Application Hosting and Monitoring"
description = "Homelab application hosting and monitoring"
date = 2025-07-13
slug = "homelab-adventure-part-4"
[taxonomies]
tags = ["Linux", "Homelab"]
+++

Welcome to my journey in building my Homelab. This is part of a multipart series; in the last part I showed how to set up an internal network across multiple hosts. This "final" post will go over application hosting and monitoring.

<!-- more -->

[**Part 1: The Adventure Begins**](@/posts/homelab-adventure-part-1.md)  
[**Part 2: Configuration Management**](@/posts/homelab-adventure-part-2.md)  
[**Part 3: Internal Network**](@/posts/homelab-adventure-part-3.md)  
[**Sidequest: Switching from Salt to Ansible**](@/posts/homelab-switching-salt-to-ansible.md)  
**Part 4: Application Hosting and Monitoring (You are here!)**

## Application Hosting

All applications are hosted in [Docker](https://docs.docker.com/) and configured by [Ansible](https://docs.ansible.com/ansible/latest/index.html) and kept updated via [Watchtower](https://github.com/containrrr/watchtower). I decided to go with this route instead of [Kubernetes](https://kubernetes.io/) because:

- I don't have shared storage across all servers
- I don't need multiple instances of the applications
- Most applications are on specific servers

Each container gets a separate role in Ansible that creates a storage folder, a user, and the container. Then in the main playbook I can specify which roles apply to which hosts or the whole group. 

```yaml
# roles/container-readeck/tasks/main.yml
- name: readeck-user-present
  ansible.builtin.user:
    name: readeck
    shell: /usr/sbin/nologin
    uid: 2002

- name: readeck-config-dir
  ansible.builtin.file:
    path: /data/readeck
    state: directory
    owner: readeck
    group: readeck
    mode: '0755'

- name: readeck-container
  community.docker.docker_container:
    name: readeck
    image: codeberg.org/readeck/readeck:latest
    restart_policy: unless-stopped
    user: '2002:2002' # has to be the id not name
    volumes:
      - "/data/readeck:/readeck"
    labels:
      com.centurylinklabs.watchtower.enable: 'true'
      traefik.enable: 'true'
```

### Exposing applications

I use [Traefik](https://doc.traefik.io/traefik/) to expose each application based on labels. I use a wildcard certificate to enable HTTPS, but Let's Encrypt could also be used. If you are worried about mounting the `docker.sock` directly to Traefik, you can set up [socket-proxy](https://github.com/wollomatic/socket-proxy).

```yaml
# roles/container-traefik/tasks/main.yml
- name: traefik-user-present
  ansible.builtin.user:
    name: traefik
    shell: /usr/sbin/nologin
    uid: 2001

- name: traefik-config-dir
  ansible.builtin.file:
    path: /data/traefik
    state: directory
    owner: traefik
    group: traefik
    mode: '0755'

- name: traefik-cert-dir
  ansible.builtin.file:
    path: /data/traefik/config/certs
    owner: traefik
    group: traefik
    state: directory
    mode: '0755'

- name: traefik-dynamic-config
  ansible.builtin.template:
    src: dynamic.yaml.j2
    dest: /data/traefik/config/dynamic.yaml
    owner: traefik
    group: traefik
    mode: '0644'
  notify: traefik-container-restart

- name: traefik-config
  ansible.builtin.template:
    src: traefik.yaml.j2
    dest: /data/traefik/config/traefik.yaml
    owner: traefik
    group: traefik
    mode: '0644'
  notify: traefik-container-restart

- name: traefik-cert
  ansible.builtin.copy:
    src: "{{ item.key }}.cert"
    dest: "/data/traefik/config/certs/{{ item.key }}.cert"
    owner: traefik
    group: traefik
    mode: '0644'
  loop: "{{ traefik_certs | dict2items }}"
  loop_control:
    label: "{{ item.key }}"
  diff: false
  notify: traefik-container-restart
  when: traefik_certs is defined

- name: traefik-certkey
  ansible.builtin.copy:
    dest: /data/traefik/config/certs/{{ item.key }}.key
    content: "{{ item.value.key }}"
    owner: traefik
    group: traefik
    mode: '0644'
  loop: "{{ traefik_certs | dict2items }}"
  loop_control:
    label: "{{ item.key }}"
  diff: false
  notify: traefik-container-restart
  when: traefik_certs is defined

- name: traefik-container
  community.docker.docker_container:
    name: traefik
    image: traefik:3.2
    restart_policy: unless-stopped
    command: "--configFile=/config/traefik.yaml"
    user: "2001:2001" # has to be the id not name
    published_ports:
      - 80:80
  	  - 443:443
  	  # Use these if you want traefik to only listen on the internal network (define 'traefik_internal_interface' in host_vars or group_vars)
      # - "{{ vars['ansible_'~traefik_internal_interface].ipv4.address }}:80:8080"
      # - "{{ vars['ansible_'~traefik_internal_interface].ipv4.address }}:443:8443"
    volumes:
      - /data/traefik/config:/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    labels:
      com.centurylinklabs.watchtower.enable: 'true'
```

```yaml
# roles/container-traefik/handlers/main.yml
- name: traefik-container-restart
  community.docker.docker_container:
    name: traefik
    restart: true
```

This config sets up the certificates and a middleware for HTTPS redirection and an `ipAllowList`. This middleware can then be specified on a container via the `traefik.http.routers.<app_name>.middlewares: 'internal-network@file'` label (replace `<app_name>` with the application name).

```yaml
# roles/container-traefik/templates/dynamic.yaml.j2
{% if traefik_certs %}
tls:
  certificates:
{% for name in traefik_certs.keys() %}
    - certFile: /config/certs/{{ name }}.cert
      keyFile: /config/certs/{{ name }}.key
{% endfor %}
{% endif %}

http:
  middlewares:
    internal-network:
      chain:
          middlewares:
            - internal-allowlist
            - https-only
            
    https-only:
      redirectScheme:
        scheme: https

    internal-allowlist: 
      ipAllowList:
        sourceRange:
          - "{{ traefik_internal_iprange }}" # internal network
          - "172.17.0.1/16" # docker range
```

This config sets up the entrypoints, HTTPS redirection and the Docker provider.

```yaml
# roles/container-traefik/templates/traefik.yaml.j2
log:
  level: INFO

entryPoints:
  web:
    address: ":8080" # will be routed to port 80 by docker
    http:
      redirections:
        entryPoint:
          to: ":443"
          scheme: https
  websecure:
    address: ":8443" # will be routed to port 443 by docker
    http:
      middlewares: 
        - https-only@file # default to https everything
      tls: {}  

providers:
  docker:
    exposedByDefault: false # require `traefik.enable: 'true'` label
    defaultRule: "Host(`{{ '{{' }} normalize .Name {{ '}}' }}.{{ traefik_domain }}`)"
    network: bridge # default network
  file:
    filename: "/config/dynamic.yaml"
    watch: true

serversTransport:
  insecureSkipVerify: true # enable self signed certs on containers
```

Now any container with the `traefik.enable: 'true'` will be automatically exposed on `<container_name>.<traefik_domain>`. If `traefik.http.routers.<app_name>.middlewares: 'internal-network@file'` is set as a label, the application will only be accessible from IPs on the `ipAllowList`.

For managing DNS, I use [DNSControl](https://dnscontrol.org/) so I can version control my DNS configuration. Ansible could also be used, but it's more verbose.

## Monitoring

For general server metrics and monitoring, I use [Netdata](https://www.netdata.cloud/). There are some additional monitoring modules I have set up: 

- [SMART monitoring](https://learn.netdata.cloud/docs/collecting-metrics/hardware-devices-and-sensors/s.m.a.r.t.)
- [UPS monitoring](https://learn.netdata.cloud/docs/collecting-metrics/ups/apc-ups)
- [Container metrics](https://learn.netdata.cloud/docs/collecting-metrics/containers-and-vms/docker)

This gives alerts for things like high disk usage, high memory usage, UPS failed over to battery, and SMART failures.

For custom monitoring, I use [Uptime Kuma](https://github.com/louislam/uptime-kuma). It works well for application health monitoring as well as custom webhook monitors. I have set up custom webhook monitors for:

- BTRFS Health
- BTRFS Scrubbing
- Backups

I have alerts from both Netdata and Uptime Kuma going to [Pushover](https://pushover.net/) and email.

[**Part 1: The Adventure Begins**](@/posts/homelab-adventure-part-1.md)  
[**Part 2: Configuration Management**](@/posts/homelab-adventure-part-2.md)  
[**Part 3: Internal Network**](@/posts/homelab-adventure-part-3.md)  
[**Sidequest: Switching from Salt to Ansible**](@/posts/homelab-switching-salt-to-ansible.md)  
**Part 4: Application Hosting and Monitoring (You are here!)**
