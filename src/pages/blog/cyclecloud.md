---
layout: "../../layouts/BlogPost.astro"
title: "High Performance Computing with Azure Cycle Cloud"
description: "Build an HPC cluster in Azure with Azure Cycle Cloud"
pubDate: "Oct 15 2023"
heroImage: "/azure-hero.avif"
previewText: "High Performance Computing clusters are commonly used to run massively parallel scientific computations. They work by using a scheduler (such as SLURM) which allocates compute resources (CPU, memory, GPU) to running processes to optimize resource utilization."
---

## High Performance Computing Clusters

High Performance Computing clusters are commonly used to run massively parallel scientific computations.

HPC clusters work by using

- operating system provisioning platform such as [Warewulf](https://warewulf.org/docs/development/contents/introduction.html)
- job scheduler such as SLURM, PBS Pro, LSF, Grid Engine, etc.

### Warewulf Compute Cluster

![Warewulf Compute cluster Architecture](/cyclecloud/beowulf_architecture.png)
_[Warewulf Compute cluster](https://warewulf.org/docs/development/contents/background.html)_

Warewulf is responsible for adding new nodes to the cluster and provisioning the OS via PXE. The compute/worker nodes connect via a high speed, private, Infiniband network to facilitate intra-cluster communication.

### Compute Cluster with Scheduler

![HPC Cluster Architecture](/cyclecloud/hpc_architecture.webp)
_[HPC Cluster with Scheduler](https://docs.hpc.cofc.edu/)_

The job scheduler runs on a Head/Control node, which is also the login node. A user connects to a Login Node, typically via SSH, to submit Batch scripts to the scheduler.

The scheduler monitors resources (CPU, memory, GPU) on Compute/Worker nodes and allocates new jobs to available worker nodes with the goal of optimizing resource utilization.

The scheduler knows which compute nodes are in use and which are free to run the Batch job. For long term storage, NFS or a parallel filesystem (such as BeeGFS) can be used. Local SSDs can be used for temporary scratch storage.

There are many job schedulers to choose from. I will be using [SLURM](https://slurm.schedmd.com/documentation.html).

## VNet and Storage

First let's create a resource group and VNet where our clusters will be created.

```
az group create --location eastus --name rg-cyclecloud
az network vnet create --location eastus --name vnet-cyclecloud --resource-group rg-cyclecloud --address-prefixes '10.0.0.0/24'
az network vnet subnet create --vnet-name vnet-cyclecloud --name snet-cyclecloud --resource-group rg-cyclecloud --address-prefixes '10.0.0.0/24'
```

We will need to create a blob storage account for Cycle Cloud to use for storing cluster configuration scripts.

```
az storage account create \
    --location eastus \
    --name sacyclecloud \
    --resource-group rg-cyclecloud \
    --sku Standard_LRS
```

If you plan on using NFS, you will also need to create a Premium file share storage account. Premium file shares are billed based on provisioned size, regardless of used capacity. I use the minimum file share size for demonstration purposes.

```
az storage account create \
    --location eastus \
    --name saclusternfs \
    --resource-group rg-cyclecloud \
    --sku Premium_LRS \
    --kind FileStorage
az storage share-rm create \
   --resource-group rg-cyclecloud \
   --storage-account saclusternfs \
   --name cluster-nfs \
   --access-tier Premium \
   --quota 100 \
   --enabled-protocols NFS
```

You will need to disable Secure Transfer setting for the NFS file share and create a private endpoint.

![Azure Marketplace](/cyclecloud/cc10.png)

![Azure Marketplace](/cyclecloud/cc11.png)

To create the private endpoint

```
storageId=$(az storage account show --name saclusternfs --resource-group rg-cyclecloud --query id --output tsv)
az network private-endpoint create --name pe-saclusternfs --resource-group rg-cyclecloud --vnet-name vnet-cyclecloud --subnet snet-cyclecloud --private-connection-resource-id $storageId --connection-name pe-connection-saclusternfs --group-ids file
```

## Cycle Cloud

[Azure Cycle Cloud](https://learn.microsoft.com/en-us/azure/cyclecloud/overview) allows us to create HPC clusters using a variety of schedulers and backend storage options. Cycle Cloud also allows you to template clusters so you can automate the creation of HPC clusters.

Next, let's create a Cycle Cloud VM which will be used to create and manage our HPC clusters. Here, I use version 8.4. Deployment instructions can be found here https://learn.microsoft.com/en-us/azure/cyclecloud/qs-install-marketplace.

![Azure Marketplace](/cyclecloud/cc1.png)

You can accept most of the defaults, however it is recommended to enable a Managed System Identity.

![Azure Marketplace](/cyclecloud/cc2.png)

Once the Cycle Cloud managment VM is deployed, get the public IP address or hostname and navigate to it in your browser. You will get an insecure self-signed certificate warning. You can [use your own TLS certificate](https://learn.microsoft.com/en-us/azure/cyclecloud/how-to/ssl-configuration?view=cyclecloud-8).

![Azure Marketplace](/cyclecloud/cc3.png)

You should now see the Cycle Cloud web console. Set a site name for your clusters.

![Azure Marketplace](/cyclecloud/cc4.png)

You will be prompted to create an Azure CycleCloud administrator account. This is a local user which allows you to sign in to the Cycle Cloud management VM.

![Azure Marketplace](/cyclecloud/cc5.png)

Your Cycle Cloud management VM will need contributor access at the subscription level to create resource groups and cluster resources (VMs, Disks, etc.).
https://learn.microsoft.com/en-us/azure/cyclecloud/how-to/managed-identities

```
SUBSCRIPTION_ID=$(az account show --query id | tr -d '"')
MANAGED_IDENTITY=$(az vm identity show --name vm-cyclecloud --resource-group rg-cyclecloud --query principalId | tr -d '"')
az role assignment create --assignee $MANAGED_IDENTITY --role Contributor --scope "/subscriptions/${SUBSCRIPTION_ID}"
```

Azure is very slow. You may need to wait some time before the role assigment completes.

![Azure Marketplace](/cyclecloud/cc6.png)

Next, click on "Back to Clusters" in the upper left corner.

We are ready to create our first cluster.

## Create a SLURM cluster

Let's create a SLURM cluster. Give your cluster a name. You will need to specify the VNet where the cluster will be reside in. You can also set the OS for the cluster (default is Alma Linux) as well as quotas for CPU utilization.

![Azure Marketplace](/cyclecloud/cc7.png)

Point your cluster to the NFS file share using the IP address of the private endpoint

![Azure Marketplace](/cyclecloud/cc12.png)

![Azure Marketplace](/cyclecloud/cc13.png)

In the "Advanced Settings" I recommend disabling internet access to the cluster.

![Azure Marketplace](/cyclecloud/cc8.png)

## Cycle Cloud CLI

We can use the [Cycle Cloud CLI](https://learn.microsoft.com/en-us/azure/cyclecloud/cli?view=cyclecloud-8) to start clusters, connect to running clusters, and even create clusters using templates. Let's use it to start our new cluster.

SSH to the cycle cloud management VM

```
chmod 0600 ~/Downloads/vm-cyclecloud_key.pem
ssh -i ~/Downloads/vm-cyclecloud_key.pem azureuser@ccdemo.eastus.cloudapp.azure.com
```

```
[azureuser@vm-cyclecloud ~]$ cyclecloud --version
CycleCloud 8.4.2-3186
```

First, initialize the cyclecloud settings. Make sure to use the Cycle Cloud admin user account we created earlier.

```
[azureuser@vm-cyclecloud ~]$ cyclecloud initialize
CycleServer URL: [http://localhost]
Detected untrusted certificate.  Allow?: [yes] yes
Using https://localhost
CycleServer username: [azureuser] ccadmin
CycleServer password:

Generating CycleServer key...
Initial account already exists, skipping initial account creation.
CycleCloud configuration stored in /home/azureuser/.cycle/config.ini
```

Let's see available clusters

```
[azureuser@vm-cyclecloud ~]$ cyclecloud show_cluster
----------------
slurm-test : off
----------------
Resource group:
Cluster nodes:
    scheduler: Off -- --
Total nodes: 1
```

Let's start this cluster

```
[azureuser@vm-cyclecloud ~]$ cyclecloud start_cluster slurm-test
Starting cluster slurm-test....
----------------------------------
slurm-test : allocation -> started
----------------------------------
Resource group:
Cluster nodes:
    scheduler: Off -- --
Total nodes: 1
```

## Running a test job

To run a test job, we must first connect to the new cluster's scheduler node.



An example SLURM batch script which runs a Single-Threaded Job

```bash
#!/bin/bash
#SBATCH --job-name=serial_job_test    # Job name
#SBATCH --mail-type=END,FAIL          # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=email@ufl.edu     # Where to send mail
#SBATCH --ntasks=1                    # Run on a single CPU
#SBATCH --mem=1gb                     # Job memory request
#SBATCH --time=00:05:00               # Time limit hrs:min:sec
#SBATCH --output=serial_test_%j.log   # Standard output and error log
pwd; hostname; date

module load python

echo "Running plot script on a single CPU core"

python /data/training/SLURM/plot_template.py

date
```

## Finishing Up

When you are finished, make sure you delete the Cycle Cloud resource group you created, so you aren't accidentally charged.

```
az group delete --name rg-cyclecloud
```
