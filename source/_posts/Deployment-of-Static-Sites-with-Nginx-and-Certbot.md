---
title: Deployment of Static Sites with Nginx and Certbot
date: 2023-01-24 19:17:10
tags: nginx, ssl, https, static website, letsencrypt, certbot
---

This note introduces how to deploy a https-enabled static website with `nginx` and `certbot`.

## Nginx Configuration
Suppose you already have the `nginx` package installed in your Linux system.
For a simple static site located at `/home/evan/blog/public`, we can have it up runing with the a simple configuration located at `/etc/nginx/conf.d/your_domain.conf`, which writes as
```shell
server {
  listen 443 ssl;
  server_name your_domain;
  
  ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;

  root /home/lighthouse/blog/public;
  index index.html; # name of index page
  location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
    root /home/lighthouse/blog/public;
  } # access to static files
}
```
After your are done editing, don't forget to run `nginx -s reload` to apply the new configuration.

If you aim to implement more advanced features, you should add more details or even a couple of more configuration files, please refer to the official [nginx documentation](https://nginx.org/en/docs/).

I believe that now you wonder **how to get those SSL certificate and key**.

## Get the SSL Certificate

You can require a paid one from some [SSL certificate organization](https://www.google.com.hk/search?q=ssl+certificate&oq=SSL+certificate). Nevertheless for us non-commercial users, we can simply generate a free certificate by ourselves, with the help of `certbot`, a certification helper that supports the automatic certification APIs provided by [letsencrypt](https://letsencrypt.org/) organization.

To generate one, all you need are the `certbot` installed and a legally owned domain name. The command is as follows
```bash
sudo certbot certonly --standalone -d your_domain_name
```
There are some points to be aware of:
- You should have ports `443` and `80` unoccupied before running the command.
- Your domain name should be well resolved by DNS service providers and pointed to your current machine.
- In China, in order to build a healthy Internet environment, the authorities require us to file our website information with the relevant department and obtain confirmation, otherwise the domain name won't resolve properly and no SSL certificate can be obtained.