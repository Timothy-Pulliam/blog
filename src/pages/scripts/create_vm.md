---
layout: "../../layouts/BlogPost.astro"
title: "Azure CLI VM creation script"
description: "Create an Azure VM with Azure CL"
pubDate: "Dec 4 2023"
heroImage: "/azure-hero.avif"
previewText: "Here is a script I use when deploying Linux VMs to Azure."
---

Here is a script I use when deploying Linux VMs to Azure. Feel free to modify it to suit your needs.

##### create_vm.sh

```bash
#!/bin/bash

options=("Ubuntu 22.04 LTS" "RHEL9" "Quit")
select opt in "${options[@]}"
do
    case $opt in
        "Ubuntu 22.04 LTS")
	    # https://ubuntu.com/server/docs/find-ubuntu-images-on-azure
            IMAGE="Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest"
            break
            ;;
        "RHEL9")
            IMAGE="RedHat:RHEL:9-LVM:latest"
            break
            ;;
        "Quit")
            exit 0
            ;;
        *) echo "Invalid option $REPLY"
            exit 1
            ;;
    esac
done

echo -n "VM Name: "
read VM_NAME

# image: [UbuntuLTS|RHEL]

echo -n "Location [eastus]: "
read LOCATION
if [ -z $LOCATION ]; then
    LOCATION="eastus";
fi

echo -n "Subscription [My_Subscription]: "
read SUBSCRIPTION
if [ -z $SUBSCRIPTION ]; then
    SUBSCRIPTION="My_Subscription"
fi

echo -n "Resource Group [My_RG]: "
read RESOURCE_GROUP_NAME
if [ -z $RESOURCE_GROUP_NAME ]; then
    RESOURCE_GROUP_NAME="My_RG"
fi

echo -n "VNet [My_VNET]: "
read VNET
if [ -z $VNET ]; then
    VNET="My_VNET"
fi

echo -n "Subnet [My_Subnet]: "
read SUBNET
if [ -z $SUBNET ]; then
    SUBNET="My_Subnet"
fi

echo ""
echo "location:         ${LOCATION}"
echo "subscription:     ${SUBSCRIPTION}"
echo "resource group:   ${RESOURCE_GROUP_NAME}"
echo "VNet:             ${VNET}"
echo "Subnet:           ${SUBNET}"
echo "VM Name:          ${VM_NAME}"
echo "Image:            ${IMAGE}"
echo ""
echo -n "Create VM with these values? (y|n): "

create_vm(){
    PASSWORD=$(python3 -c 'import secrets; password = secrets.token_urlsafe(20); print(password)')

    az vm create \
        --location $LOCATION \
        --subscription $SUBSCRIPTION \
        --resource-group $RESOURCE_GROUP_NAME \
        --name $VM_NAME \
        --image $IMAGE \
        --size Standard_D4s_v3 \
        --admin-username azureuser \
        --admin-password $PASSWORD \
        --generate-ssh-keys \
        --assign-identity "[system]" \
        --vnet-name $VNET \
        --subnet $SUBNET \
        --public-ip-address "" \
        --os-disk-size-gb 500 \
        --tags "source=azure-cli" \
        --accept-term
}

read choice
case "$choice" in
  y|Y ) create_vm;;
  n|N ) echo "exiting"; exit 0;;
  * ) echo "invalid";;
esac

echo ""
echo "New VM Credentials"
echo "username: azureuser"
echo "password: ${PASSWORD}"

echo ""
echo "Copying Ansible SSH key to new VM"
echo ""
# get IP address of new VM
IP=$(az vm show --subscription $SUBSCRIPTION --resource-group $RESOURCE_GROUP_NAME --name $VM_NAME -d --query privateIps | tr -d '"')
ssh-copy-id -f -i ./ansible_rsa.pub azureuser@$IP
```
