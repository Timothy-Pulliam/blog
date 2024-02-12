---
layout: "../../layouts/BlogPost.astro"
title: "Estimating Cloud Costs with Infracost/Terraform"
description: "Estimating Cloud Costs with Infracost/Terraform"
pubDate: "Feb 11 2024"
heroImage: "/terraform-hero.png"
previewText: "Infracost uses your Terraform code to tell you how much your resources will cost _before_ deploying them. This allows you to include budgeting in your code pipelines!"
---

# The Problem

Before, if you wanted to know how expensive your cloud infrastructure would be, you would have to use a tool like [Azure's Pricing Calculator](https://azure.microsoft.com/en-in/pricing/calculator/?cdn=disable). It's a nice tool that Azure provides, but it can involve a lot of tedious manual calcuations. Wouldn't it be nice if you could _automate_ price estimates?

# Enter Infracost

[Infracost](https://www.infracost.io/docs/) uses your Terraform code to tell you how much your resources will cost _before_ deploying them. This allows you to skip tedious calculations, avoid surprise billing, as well as include budgeting in your Terraform code pipelines!

Infracost is

- Incredibly easy to use
- built to work with AWS, Azure, and Google Cloud
- Runs on Mac OS, Linux, and Windows
- [It's Free!](https://www.infracost.io/pricing/)

# Installation

## Install for Mac OS

```
brew install infracost

infracost --version # Should show 0.10.33
```

## Install for Linux

```
# Downloads the CLI based on your OS/arch and puts it in /usr/local/bin
curl -fsSL https://raw.githubusercontent.com/infracost/infracost/master/scripts/install.sh | sh
```

## Install with Chocolatey

```
choco install infracost

infracost --version # Should show 0.10.33
```

# Get API key

Next, you have to register for a free API key.

```
infracost auth login
```

You will be prompted to register for a free API key. Once registered, click on "Org Settings" to obtain your API key. Then set it on the CLI

![Infracost API key](/infracost/infracost.png)

```
infracost configure set api_key MY_API_KEY
```

You can then get your api key with the following command.

```
infracost configure get api_key
```

# Running a Cost Breakdown

```
cd ~/code/terraform-code/
infracost breakdown --path .
```

You will get an estimated breakdown of costs based on your defined Terraform resources.

```
Evaluating Terraform directory at .
  ✔ Downloading Terraform modules
  ✔ Evaluating Terraform directory
  ✔ Retrieving cloud prices to calculate costs

Project: Timothy-Pulliam/aks-terraform-dev

 Name                                            Monthly Qty  Unit                      Monthly Cost

 azurerm_application_gateway.this
 ├─ Gateway usage (basic v2)                             730  hours                          $146.00
 └─ V2 capacity units (basic)                            730  CU                               $5.84

 azurerm_key_vault_certificate.this
 ├─ Certificate renewals                  Monthly cost depends on usage: $3.00 per requests
 └─ Certificate operations                Monthly cost depends on usage: $0.03 per 10K transactions

 azurerm_postgresql_flexible_server.this
 ├─ Compute (GP_Standard_D8ds_v4)                        730  hours                          $519.76
 ├─ Storage                                              512  GB                              $58.88
 └─ Additional backup storage             Monthly cost depends on usage: $0.095 per GB

 azurerm_private_dns_zone.db
 └─ Hosted zone                                            1  months                           $0.50

 azurerm_public_ip.appgw_pip
 └─ IP address (static)                                  730  hours                            $3.65

 OVERALL TOTAL                                                                               $734.63
──────────────────────────────────
18 cloud resources were detected:
∙ 5 were estimated, 3 of which include usage-based costs, see https://infracost.io/usage-file
∙ 13 were free, rerun with --show-skipped to see details

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┓
┃ Project                                            ┃ Monthly cost ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━┫
┃ Timothy-Pulliam/aks-terraform-dev                  ┃ $735         ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━┛
```

# More Features

Infracost can do so much more, including

- Save cost breakdowns as JSON files
- Diff your current breakdown to previous saved JSON file
- Integrate billing into your CI/CD pipelines
- Display your breakdown in a web-based graphical dashboard by uploading a JSON file to Infracost Cloud

```
infracost upload --path infracost-base.json
```

![infracost dashboard](/infracost/dashboard-chart.png)

And so much more.

Check out the Docs for more information

[https://www.infracost.io/docs/
](https://www.infracost.io/docs/)

or ask questions on the Infracost Community Slack channel

[https://www.infracost.io/community-chat](https://www.infracost.io/community-chat)
