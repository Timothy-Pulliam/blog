---
layout: "../../layouts/BlogPost.astro"
title: "Terraform Workspaces"
description: "Tutorial for Terraform Workspaces"
pubDate: "Dec 13 2022"
heroImage: "/Timothy-Pulliam/terraform-hero.png"
---

If you are working with Infrastructure as Code, you probably don't want to deploy new changes directly to your production environment. It's better to deploy into a dev environment that won't affect your end users experience. Enter [Terraform Workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces).

>A common use for multiple workspaces is to create a parallel, distinct copy of a set of infrastructure to test a set of changes before modifying production infrastructure.

Creating and using a workspace is simple.

```bash
