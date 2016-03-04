---
layout: post
title: My experience with Terraform
description: ""
modified: 2014-12-11
category: posts
tags: 
- automation
blog: true
---

One of the projects recently tasked to me was cloud infrastructure automation. We are building out a presence in AWS and potentially GCE and need a way to manage the infrastructure services we are providing to our app teams. Since cloudformation is lock-in to AWS, we needed something else... Enter [Terraform](https://www.terraform.io/).

Using terraform we were able to build out a versioned configuration for the interdependencies in AWS and soon to be GCE. 

Also have been using it to build out my personal educational projects, example would be generating/destroying CoreOS clusters in DigitalOcean. 
