---
layout: "../../layouts/BlogPost.astro"
title: "Terraform Virtual Environments with tfenv"
description: "creating a virtual environment with tfenv"
pubDate: "Dec 13 2022"
heroImage: "/terraform-hero.png"
previewText: "If you are writing many terraform configs, or your team is large, it’s really important to make sure everyone is using the same Terraform version. This is where Terraform virtual environments come in."
---

When writing a Terraform config, you should [specify the version of Terraform](https://developer.hashicorp.com/terraform/language/settings#specifying-a-required-terraform-version) you are using. This ensures that anyone using your terraform config is using the same version of the CLI tool. This is set in the terraform block, usually in main.tf, like so

```hcl
terraform {
    required_version = "1.1.5"
}
```

If you are writing many terraform configs, or your team is large, it’s really important to make sure everyone is using the same Terraform version. This is where [tfenv](https://github.com/tfutils/tfenv) comes in.

### Installation and Usage

It’s really easy to get it installed and running. On MacOS it’s as simple as

```bash
$ brew install tfenv
```

You can then list out your terraform versions

```bash
$ tfenv list
* 1.0.10
```

The version with the asterisk is the version of terraform that is currently activated. You can then install and use a new version like so

```bash
$ tfenv install 1.1.5
$ tfenv use 1.1.5
```

The same version can be uninstalled simply with

```bash
$ tfenv uninstall 1.1.5
```

Of course you can install a version which matches certain criteria. From the README

```plaintext
Install a specific version of Terraform.

If no parameter is passed, the version to use is resolved automatically via TFENV_TERRAFORM_VERSION environment variable or .terraform-version files, in that order of precedence, i.e. TFENV_TERRAFORM_VERSION, then .terraform-version. The default is 'latest' if none are found.

If a parameter is passed, available options:

| <div style="width:150px">Syntax</div> | Description |
| :------------- | :----------- |
| x.y.z  | Semver 2.0.0 string specifying the exact version to install |
| latest | is a syntax to install latest version |
| latest:&lt;regex&gt; | is a syntax to install latest version matching regex (used by grep -e) |
| latest-allowed | is a syntax to scan your Terraform files to detect which version is maximally allowed. |
| min-required | is a syntax to scan your Terraform files to detect which version is minimally required. |
```

```bash
$ tfenv install
$ tfenv install 0.7.0
$ tfenv install latest
$ tfenv install latest:^0.8
$ tfenv install min-required
```

### Conclusion

It’s just that easy. You should use tfenv whenever you start a Terraform project of any size. Let me know if you have used tfenv before and if you like it. And remember, automate everything!
