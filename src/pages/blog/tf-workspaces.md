---
layout: "../../layouts/BlogPost.astro"
title: "Terraform Workspaces"
description: "Tutorial for Terraform Workspaces"
pubDate: "Dec 13 2022"
heroImage: "/terraform-hero.png"
previewText: "If you are working with Infrastructure as Code, you probably don't want to deploy new changes directly to your production environment. It's better to deploy into a dev environment that won't affect your end users' experience. Enter Terraform Workspaces."
---

If you are working with Infrastructure as Code, you probably don't want to deploy new changes directly to your production environment. It's better to deploy into a dev environment that won't affect your end users' experience. Enter [Terraform Workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces).

> A common use for multiple workspaces is to create a parallel, distinct copy of a set of infrastructure to test a set of changes before modifying production infrastructure.

Terraform always creates a default workspace

```bash
$ terraform workspace list
* default
```

Creating and using a workspace is simple.

```bash
$ terraform workspace new dev
Created and switched to workspace "dev"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
$ terraform workspace list
  default
* dev
```

Switch to a different workspace

```bash
$ terraform workspace select prod
$ terraform workspace show
prod
```

Each workspace has it's own state file

```bash
$ ls terraform.tfstate.d
dev	prod
```

If using [Azure Blob Storage for Terraform Backend](/scripts/tf-backend), you can see the different state files.

![terraform state files in Azure Blob Storage](/tfstate_files.png "Terraform state files in Azure Blob Storage")

```bash
$ terraform workspace list
  default
  dev
  prod
  qa
* uat
```

You can then create a .tfvars file for each workspace.

```bash
vars
├── dev.tfvars
├── prod.tfvars
├── qa.tfvars
└── uat.tfvars
```

Then use the var file corresponding to the current workspace

```bash
$ terraform plan -var-file=./vars/$(terraform workspace show).tfvars

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and
found no differences, so no changes are needed.
```

Terraform workspaces can help you organize your Terraform project. That's all there is to it.
