---
layout: "../../layouts/BlogPost.astro"
title: "Scripting a Terraform Backend"
description: "Create a Terraform Backend with Azure CLI"
pubDate: "Dec 13 2022"
heroImage: "/Timothy-Pulliam/terraform-hero.png"
---

A best practice when working with Terraform is to store the [state file](https://developer.hashicorp.com/terraform/language/state) in a remote location. This is accomplished by creating a [Terraform Backend](https://developer.hashicorp.com/terraform/language/settings/backends/configuration). For Azure, an [Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) can be used to store the state file in.

This is something you will probably do a lot. So why not script it? Here is the script I use when creating a new Terraform Backend. Feel free to use and modify it yourself.

```bash
#!/bin/bash

# This script creates an Azure Storage Account
# to store the Terraform backend. This is where
# Terraform stores its state file.

RESOURCE_GROUP_NAME="rg-tfm-backend"
# Note: Storage Account names must be globally unique to Azure
STORAGE_ACCOUNT_NAME="satfmbackend"

# Create Resource Group
az group create --location eastus --name $RESOURCE_GROUP_NAME --tags project=iac

# Create Storage Account
# Note: Storage Account names must be globally unique to Azure
az storage account create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $STORAGE_ACCOUNT_NAME \
    --access-tier hot \
    --allow-blob-public-access false \
    --encryption-services blob \
    --location eastus \
    --sku STANDARD_ZRS \
    --kind StorageV2 

# enable blob container versioning
az storage account blob-service-properties update \
    --enable-versioning true \
    --resource-group $RESOURCE_GROUP_NAME \
    --account-name $STORAGE_ACCOUNT_NAME

# enable container level delete retention (soft delete)
az storage account blob-service-properties update \
    --enable-container-delete-retention true \
    --container-delete-retention-days 7 \
    --account-name $STORAGE_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP_NAME

# create blob container
STORAGE_ACCOUNT_ACCESS_KEY=$(az storage account keys list --account-name $STORAGE_ACCOUNT_NAME --query "[0].value")
az storage container create \
    --name tfmstate \
    --account-name $STORAGE_ACCOUNT_NAME \
    --account-key $STORAGE_ACCOUNT_ACCESS_KEY
```

Then make sure to add the backend configuration to your main.tf file.

```hcl
terraform {
  # Terraform version
  required_version = ">= 1.2.7"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "3.27.0"
    }
  }

  # create backend using create_backend.sh
  backend "azurerm" {
    resource_group_name  = "rg-tfm-backend"
    storage_account_name = "satfmbackend"
    container_name       = "tfmstate"
    key                  = "terraform.tfstate"
    # Storage Account Access Key
    # Do not store in plain text!!! Export to env variable instead!!!
    # export ARM_ACCESS_KEY="my-storage-account-access-key"
    access_key = ""
  }
}
```

And that's all there is to it. After you set up your Terraform Backend, you might want to create a Terrafom Workspace for each of your environments (dev, qa, uat, prod). I cover that in [this blog post](tf-workspaces). Thanks for reading.