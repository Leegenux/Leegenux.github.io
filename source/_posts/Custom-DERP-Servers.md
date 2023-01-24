---
title: Custom DERP Servers
date: 2023-01-24 21:58:34
tags: tailscale, derp
---

Tailscale is a useful utility to connect all your consoles together with a virtual LAN. 
It's based on Wireguard and supports Windows, Linux, macOS, Android and iOS operating system.

DERP servers are relay nodes in the tailscale network. Since all the official tailscale DERP servers are deployed outside of China, a private DERP server is necessary for fast connection.
Deployment is explained in offcial documentation - [Custom DERP Servers - Tailscale](https://tailscale.com/kb/1118/custom-derp-servers/)

Here I make a brief record of my deployment and note some points that's not illustrated in the official documentation.

## Installation

DERP server is written in Go. To avoid any strange problems that may occur, I recommend using the [official build of Go release](https://go.dev/dl/) and follow the [instructions](https://go.dev/doc/install) to install it. 

The installation is straightforward,
```
go install tailscale.com/cmd/derper@main
```

## Deployment

As a reference, I provide a command for starting up the DERP server that meets most use cases
```
sudo ~/go/bin/derper --hostname='your_domain_name' --verify-clients --stun -a :444 -certdir path_certs -certmode "manual" -http-port 81
```
With this command, the derper serves on port `444` with TLS encryption and STUN protocal requests are served on port `3478` by default. 

Note that the derper requires the certificate and key files named following the pattern.
```
your_domain_name.crt
your_domain_name.key
```
and placed in `path_certs`.

NOTE: You can obtain the SSL certification with the help of `certbot`, a helper program that supports the automatic certification APIs provided by [letsencrypt](https://letsencrypt.org/) organization. Instructions are given in [one](2023/01/24/Deployment-of-Static-Sites-with-Nginx-and-Certbot/index.html) of my posts.