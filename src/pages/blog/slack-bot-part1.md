---
layout: "../../layouts/BlogPost.astro"
title: "Trigger Azure Function Apps With Slack Slash Commands: Part 1"
description: "Azure Python SDK"
pubDate: "Dec 14 2022"
heroImage: "/azure-hero.avif"
previewText: "Slack is a popular instant messaging service. Developers using slack can create a bot that triggers an Azure Function App written in Python."
---

## Overview

Slack is a popular instant messaging service. Developers can use [Slack Apps](https://devopsprofessionalsco.slack.com/apps) to create integrations with the platform. 

In this article, I show how to create a basic Slack bot. The Slack bot accepts [Slash Commands](https://api.slack.com/interactivity/slash-commands) to make HTTP requests to an Azure Function App endpoint. The Function App will then run Azure comands using [Azure Python SDK](https://learn.microsoft.com/en-us/azure/developer/python/sdk/azure-sdk-overview).

<!-- The Azure Python SDK allows you to access Azure resources using python code. From the [Azure Python SDK documentation](https://learn.microsoft.com/en-us/azure/developer/python/sdk/azure-sdk-overview)

>Because the Azure CLI is written in Python using the management libraries, anything you can do with Azure CLI commands you can also do from a Python script.

This opens new opportunities for automation of Azure resources. For example:

* A Slack Bot that pulls information about running/stopped VMs
*  -->

## Setting Up a Local Dev Environment

First thing to do is to set up our local environment. We will need Azure CLI, a Python virtual environment, and [azure-functions-core-tools](https://www.npmjs.com/package/azure-functions-core-tools), which will require npm.
```bash
$ mkdir -p ~/code/slack-bot
$ cd ~/code/slack-bot
# Install Azure CLI
$ brew install azure-cli
# Authenticate to Azure
$ az login
# Create a python virtual environment (requires specific python version)
$ brew install python@3.9
$ python3.9 -m venv venv
# Activate the virtual environment
$ . ./venv/bin/activate
# Azure Functions Core Tools
$ brew install npm
$ npm install -g azure-functions-core-tools@4 --unsafe-perm true
```

Now we can create our function project using `func init`. You should ensure that the python version in your venv is the same as the Function App runtime in Azure. I am using Python 3.9.

```bash
(venv) tpulliam@lappy slack-bot % func init
Select a number for worker runtime:
1. dotnet
2. dotnet (isolated process)
3. node
4. python
5. powershell
6. custom
Choose option: 4
python
Found Python version 3.9.16 (python3).
Did you know? There is a new Python programming model in public preview. For fewer files and a decorator based approach, learn how you can try it out today at https://aka.ms/pythonprogrammingmodel
Writing requirements.txt
Writing .funcignore
Writing getting_started.md
Writing .gitignore
Writing host.json
Writing local.settings.json
/Users/tpulliam/code/slack-bot/.vscode/extensions.json already exists. Skipped!
```

This creates a `requirements.txt` file, which can be used to install necessary python dependencies using pip.

```bash
(venv) tpulliam@lappy slack-bot % pip install -r requirements.txt
```

Finally, create the actual python function

```bash
(venv) tpulliam@lappy slack-bot % func new --name hello --language python --template 'HTTP trigger'
Select a number for template:HTTP trigger
Function name: [HttpTrigger] Writing /Users/tpulliam/code/slack-bot/hello/__init__.py
Writing /Users/tpulliam/code/slack-bot/hello/function.json
The function "hello" was created successfully from the "HTTP trigger" template.
```

We can test the function locally using `func start`

Note: If you are using a MacBook with an M1 ARM processor, `func start` will produce an error. For a workaround, see this [Medium article](https://medium.com/@andreas.katzian/running-python-azure-function-locally-on-an-m1-18dae7128ac8).

You should have a similar project structure. The `__init__.py` file contains the actual function code. 

![project structure](/slack-bot/project-structure.png)

## Create the Function App

When creating a function app, you must create or link to a general-purpose Azure Storage account that supports Blobs, Queue, and Table storage.

You must also create a function app consumption plan. The follow plan skus can be used:
* B1(Basic Small) 
* B2(Basic Medium)
* B3(Basic Large)
* S1(Standard Small)
* P1V2(Premium V2 Small)
* I1(Isolated Small)
* I2 (Isolated Medium)
* I3 (Isolated Large)
* K1 (Kubernetes).

```bash
#!/bin/bash

RESOURCE_GROUP_NAME='rg-slackbot'
LOCATION='east us'
STORAGE_ACCOUNT_NAME='saslackbot'
FUNCTIONAPP_NAME='funcapp-slackbot'

# create resource group
az group create \
    --location $LOCATION \
    --resource-group $RESOURCE_GROUP_NAME

# create storage account 
# name must be globally unique
az storage account create \
    --name $STORAGE_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP_NAME \
    --location $LOCATION \
    --sku Standard_LRS

# create function app consumption plan
# Azure only charges for time the function app runs
az functionapp plan create \
    --name "${FUNCTIONAPP_NAME}-plan" \
    --resource-group $RESOURCE_GROUP_NAME \
    --location $LOCATION \
    --is-linux true \
    --sku B1 

# create function app
# name must be globally unique
az functionapp create \
    --name $FUNCTIONAPP_NAME \
    --resource-group $RESOURCE_GROUP_NAME \
    --os-type Linux \
    --runtime python \
    --runtime-version 3.9 \
    --storage-account $STORAGE_ACCOUNT_NAME \
    --plan "${FUNCTIONAPP_NAME}-plan" \
    --functions-version 4
```

`func azure functionapp publish` takes our local code and creates a tarball, which then is uploaded to the function app in Azure.

```bash
(venv) tpulliam@Timothys-MBP slack-bot % func azure functionapp publish slack-bot-demo                
Getting site publishing info...
Updating Application Settings for Remote build...
Creating archive for current directory...
Performing remote build for functions project.
...
Remote build succeeded!
```

In the azure portal, you should be able to see the deployed function (it can take a few minutes to appear)

![create function](/slack-bot/new-function.png)

You can get the HTTP URL for the function by clicking on the function and then clicking Get Function URL. We will need this later to point Slack to this URL.

![get function url](/slack-bot/get-url.png)

Any time you make changes to the `__init__.py` file, you can redeploy by issuing the `func azure functionapp publish slack-bot-demo` command.

## Create the Slack App

To create a slack bot, we must first create a Slack App. Documentation can be found [here](https://api.slack.com/authentication/basics).

You can see your apps by going to https://api.slack.com/apps

![Create a Slack App](/slack-bot/create-slack-app.png)

Choose the option to create from scratch.

![Specify name and workspace for your slack app](/slack-bot/slack-app-name.png)

### Set OAuth Permssion scope

Click Oauth and Permissions

![oauth scope](/slack-bot/oauth-scope.png)

Scroll down to the Scopes section and click to Add an OAuth Scope.

Click on OAuth and Permssions, and under the Scopes section, grant permssion for slash commands

![Grant app permissions for slash commands](/slack-bot/scopes.png)

Install the app to your workspace

![install app](/slack-bot/install-app.png)

Set bot to Always Show as Online

![always online](/slack-bot/always-online.png)

You should now be able to see the bot in your Slack workspace


![install app](/slack-bot/helper.png)

## Create Slash Command

Now that the bot has been created, we need to create a slash command to trigger the bot.

Navigate to Slash Commands and create a new slash command. Set the Request URL to the functionapp endpoint (which can be found in the Azure portal). For now, I am just going to create a slash command that says hello.

![create slash comand](/slack-bot/create-slash-command.png)

![new command](/slack-bot/new-command.png)

### Testing the Slash Command

In any channel of your Slack workspace, type `/hello`. You should see a response from the function app that says "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."

![testing the slash command](/slack-bot/testing-command.png)

## Part 2

You now have a working slack bot that triggers a function app using slash commands. In part 2, we will edit the function to do something meaningful using Azure Python SDK. 

