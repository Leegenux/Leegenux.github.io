---
title: Introduction to Hexo Blog Framework
date: 2023-01-24 10:10:47
tags: hexo, blog framework
---

Before writing your posts, first create a file system of hexo framework by executing
```
hexo init
```
in a empty directory.

The static site is generated with
```
hexo generate
```
and stored in `public` directory.

If you want to create a new post, use 
```
hexo new
```
command.

Before deploying, preview the site by serving at localhost with
```
hexo server
```
After everything settled, use 
```
hexo deploy
```
to deploy all the documents in `public`. The deployer follows the configurations in the section `deploy` of the configuration file `_config.yml`.