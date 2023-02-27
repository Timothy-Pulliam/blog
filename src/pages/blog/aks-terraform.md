---
layout: "../../layouts/BlogPost.astro"
title: "Terraforming an AKS Cluster"
description: "Create an AKS cluster using Terraform"
pubDate: "Dec 30 2022"
heroImage: "/azure-hero.avif"
previewText: "TODO"
---

If you are creating Azure Kuberenetes clusters, it's a good idea to automate it using Terraform. This allows you to quickly recreate the cluster, reducing human error, and to track the code in version control.

To do this, I use the Azure [AKS Terraform module](https://registry.terraform.io/modules/Azure/aks/azurerm/latest), maintained by Microsoft.