+++
title = "Homelab Adventure - Part 3: Internal Network"
date = 2023-03-29
slug = "homelab-adventure-part-3"
[taxonomies]
tags = ["Homelab"]
+++

Welcome to my journey in building my homelab. This is part of a multipart series, in the last part I gave an overview of how to do configuration management. This one will cover how I set up my internal network.

<!-- more -->

[**Part 1: The Adventure Begins**](/posts/homelab-adventure-part-1/)  
[**Part 2: Configuration Management**](/posts/homelab-adventure-part-2/)  
**Part 3: Internal Network (You are here!)**

## Why?

It is helpful to have an internal and external network for your homelab. This allows hosting of internal services without exposing them to the internet.

Since my homelab lives in multiple datacenters (and at home), it's less convenient to use a site-to-site VPN. So a peer-to-peer VPN between all my servers is ideal. This also makes it easy for me to add non-servers to the network like my phone or desktop to access those internal services.

## What is ZeroTier

> It’s a distributed network hypervisor built atop a cryptographically
> secure global peer-to-peer network. It provides advanced network
> virtualization and management capabilities on par with an enterprise SDN
> switch, but across both local and wide area networks and connecting
> almost any kind of app or device.

This means ZeroTier is "mostly" a peer-to-peer VPN, what I mean by "mostly" is that it still uses a "centralized" network controller (you can host your own if you want). The bulk of the traffic is peer-to-peer, and the network controller is only for the initial routing. [This has some trade-offs with something that is entirely peer-to-peer](http://adamierymenko.com/decentralization.html).

## Alternatives

These are some other options for peer-to-peer VPNs:

[tinc](https://www.tinc-vpn.org/) is one of the oldest peer-to-peer VPN's, but to set it up you have to share the generated public keys for each node with all the other nodes you want to connect.

[Nebula](https://github.com/slackhq/nebula) (inspired by tinc) has similar issues to tinc with certificate distribution. One advantage over tinc is that a discovery server is present in the Nebula network to ease node discovery.

[Tailscale](https://tailscale.com/) while a great alternative to ZeroTier, it didn’t exist at the time I set up ZeroTier. [This is a great comparison between Tailscale and ZeroTier](https://tailscale.com/compare/zerotier/).

## ZeroTier and Salt

I have written a [salt formula](https://github.com/gamingrobot/salt-formula-zerotier) for managing ZeroTier with salt. It allows interacting with the local agent and with the network controller's API.

[**Part 1: The Adventure Begins**](/posts/homelab-adventure-part-1/)  
[**Part 2: Configuration Management**](/posts/homelab-adventure-part-2/)  
**Part 3: Internal Network (You are here!)**
