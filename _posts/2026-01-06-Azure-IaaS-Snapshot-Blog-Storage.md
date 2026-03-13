---
layout: post
title: Save on Azure IaaS cost by keeping ad-hoc Virtual Machines in blob storage
date: 2026-01-06
categories: Cloud
---

This post is going to walk you through how to create a **snapshot** of an Azure VM disk for **short term** storage, and exporting the snapshot to **VHD** for **long term** storage in Azure blob container. Then the process to restore the VM from snapshot or VHD. 

> **NOTE**
> This post will outline a CLI method. For information on how to do this in the GUI I suggest these resources:
> - [Microsoft Learn - Snapshot Copy Managed Disk](https://learn.microsoft.com/en-us/azure/virtual-machines/snapshot-copy-managed-disk?tabs=portal)
> - [Microsoft Learn - Attach OS Disk](https://learn.microsoft.com/en-us/azure/virtual-machines/attach-os-disk?tabs=portal)
> - [Microsoft Learn - Download VHD](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/download-vhd?tabs=azure-portal)
>
> Follow the related links in these resources for the complete GUI solution.


Before we start, we have this simple setup in Azure. This is basically just the setup you would have in Azure after deploying a VM, plus a storage account.

- Resource group: `rgSimLab`
    - `SimLab-vnet`: vnet with default subnet only
    - `SimLab-nsg`: nsg associated to the default subnet inside the vnet
    - `simlabcoolstorage`: Cool storage account with a containter named "vhds"
    - `SimLab-VM`: the VM, with delete NIC, IP and Disk option checked at creation
    - `SimLab-VM_OsDisk`: the OS disk
    - `SimLab-VM-pubip`: the public IP
    - `simlab-vm882`: the NIC
 
![Azure Screenshot Resources](/images/2026-01-06-Azure-IaaS-Snapshot-Blog-Storage/image1.png)

---

## Short term storage method (snapshot only)
### Step 1: Take a snapshot

```
# VARIABLES
VM_NAME="SimLab-VM"
RG_NAME="rgSimLab"


# Get the OS disk name
OS_DISK_ID=$(az vm show --name "$VM_NAME" --resource-group "$RG_NAME" --query "storageProfile.osDisk.managedDisk.id" -o tsv)
OS_DISK_NAME=$(basename "$OS_DISK_ID")

# Create snapshot
SNAPSHOT_NAME="${VM_NAME}-snapshot"
az snapshot create \
 --resource-group "$RG_NAME" \
 --source "$OS_DISK_ID" \
 --name "$SNAPSHOT_NAME" \
 --output none

```
The snapshot will now exist a resource inside the resource group.

### Step 2: Delete VM
From here, you could simply delete the VM and associated Disk, IP and NIC for short term storage. We keep the NSG and VNET because they're free and will be used for re-provision:
```
az vm delete --resource-group $RG_NAME --name $VM_NAME --yes --no-wait
```

### Step 3: Restoring the snapshot to use the VM again
Create a managed disk from the snapshot. Ensure to update the `--location` parameter here.
```
# VARIABLES
RG_NAME="rgSimLab"
DISK_NAME="SimLab-VM_OsDisk"
SNAPSHOT_NAME="SimLab-VM-snapshot"
DISK_LOCATION="<specify same as VM location>"

  
az disk create \
 --resource-group $RG_NAME \
 --name $DISK_NAME \
 --source $SNAPSHOT_NAME \
 --location $DISK_LOCATION 
```
Create a VM from the Managed Disk
```
# VARIABLES
VM_NAME="SimLab-VM"
VM_SIZE="Standard_D2s_v3"
SUB_ID=$(az account show --query id -o tsv)
OS_DISK_ID="/subscriptions/$SUB_ID/resourceGroups/$RG_NAME/providers/Microsoft.Compute/disks/$DISK_NAME"
OS_TYPE="linux"

az vm create \
 --resource-group $RG_NAME \
 --name $VM_NAME \
 --size $VM_SIZE \
 --attach-os-disk $OS_DISK_ID \
 --os-type $OS_TYPE \
 --nsg ""
```

>**NOTE**
>We specify no NSG because we already have one assigned to the default subnet inside the vnet, which is where the VM will be deployed.

Our VM is now ready to use and in the same state as it was when we took a snapshot. Now that the VM is restored, you can delete the snapshot.

---

## Long term storage method (VHD in blob container)
### Step 1: Capture `plan-name`, `plan-publisher` and `plan-product`
You will need this information later if using a Marketplace image.
```
# VARIABLES
VM_NAME="SimLab-VM"
RG_NAME="rgSimLab"

az vm show \
 --resource-group $RG_NAME \
 --name $VM_NAME \
 --query "plan"
```
The output will look something like this (JSON):
```
{
  "name": "cis-rhel8-l1",
  "product": "cis-rhel8",
  "publisher": "center-for-internet-security-inc"
}
```

### Step 2: Take a snapshot (same as previous step 1). And setup variables for next steps.
```
#Variables
RG_NAME="rgSimLab"
VM_NAME="SimLab-VM"
SNAPSHOT_NAME="SimLab-VM-snapshot"
STORAGE_ACCOUNT="simlabcoolstorage"
CONTAINER_NAME="vhds"
BLOB_NAME="SimLab-VM-snapshot.vhd"


# Get the OS disk name
OS_DISK_ID=$(az vm show --name "$VM_NAME" --resource-group "$RG_NAME" --query "storageProfile.osDisk.managedDisk.id" -o tsv)
OS_DISK_NAME=$(basename "$OS_DISK_ID")

# Create snapshot
SNAPSHOT_NAME="${VM_NAME}-snapshot"
az snapshot create \
 --resource-group "$RG_NAME" \
 --source "$OS_DISK_ID" \
 --name "$SNAPSHOT_NAME" \
 --output none

```

### Step 3: Create a SAS URI from the snapshot and copy to storage
>**NOTE** The copy command will execute and put you back into interactive CLI, but the copy will still be active. Since it was cool storage, the state of the copy for me was "pending" for about 45 minutes. To check copy status, use command below.

```
#Generate SAS URI
SAS_URI=$(az snapshot grant-access \
 --resource-group "$RG_NAME" \
 --name "$SNAPSHOT_NAME" \
 --access Read \
 --duration-in-seconds 3600 \
 --query accessSAS \
 -o tsv)

#Copy to storage
az storage blob copy start \
 --destination-blob $BLOB_NAME \
 --destination-container $CONTAINER_NAME \
 --account-name $STORAGE_ACCOUNT \
 --source-uri "$SAS_URI"

#Check copy status - "pending" is in progress, "success" is complete
az storage blob show --container-name $CONTAINER_NAME --name $BLOB_NAME --account-name $STORAGE_ACCOUNT --query "properties.copy.status" -o tsv
```

If you're combining these commands into a script, you can use this codeblock to wait and check the status of the copy:

```
while true; do
    status=$(az storage blob show --container-name $CONTAINER_NAME --name $BLOB_NAME --account-name $STORAGE_ACCOUNT --query "properties.copy.status" -o tsv)
    if [ "$status" == "success" ]; then
        echo "Copy successful."
        break
    elif [ "$status" == "failed" ]; then
        echo "Copy failed, revoking access and stopping."
        az snapshot revoke-access --resource-group $RG_NAME --name $SNAPSHOT_NAME
        exit 1
    fi
    echo "Copying... ($status)"
    sleep 30
done
```
![Azure Screenshot vhd in storage](/images/2026-01-06-Azure-IaaS-Snapshot-Blog-Storage/image2.png)

### Step 4: Once copy is complete (`success` status), revoke access and clean up
```
#revoke snapshot access
az snapshot revoke-access --resource-group $RG_NAME --name $SNAPSHOT_NAME

#clean up
az vm delete --resource-group $RG_NAME --name $VM_NAME --yes --no-wait
az snapshot delete --resource-group $RG_NAME --name $SNAPSHOT_NAME
```
>**NOTE** I suggest you ensure the VHD exists in the storage account before deleting the snapshot. As long you have either a VHD or Snapshot resource in Azure, you can restore the VM.

### Step 5: Restoring the VHD to disk to recreate the VM
Create a managed disk from VHD. OS type either `windows` or `linux`. Ensure to set `--location`.
```
# VARIABLES
VM_NAME="SimLab-VM"
RG_NAME="rgSimLab"
VM_SIZE="Standard_D2s_v3"
SUB_ID=$(az account show --query id -o tsv)
OS_TYPE="linux"
DISK_NAME="disk-SimLab-VM"
STORAGE_ACCOUNT="simlabcoolstorage"
CONTAINER_NAME="vhds"
BLOB_NAME="SimLab-VM-snapshot.vhd"

az disk create \
 --resource-group $RG_NAME \
 --name $DISK_NAME \
 --source https://$STORAGE_ACCOUNT.blob.core.windows.net/$CONTAINER_NAME/$BLOB_NAME \
 --location <Same as desired VM location> \
 --os-type $OS_TYPE
```

  ### Step 6: Create a VM from disk, note that NSG is associated to existing subnet already
Here we are going to need the `plan-name`, `plan-publisher` and `plan-product` info we captured in step one. Because when we exported the snapshot to VHD, Azure lost this metadata associated with the snapshot, now it must be explicitly specified to create the VM.
```
az vm create \
 --resource-group $RG_NAME \
 --name $VM_NAME \
 --attach-os-disk /subscriptions/$SUB_ID/resourceGroups/$RG_NAME/providers/Microsoft.Compute/disks/$DISK_NAME \
 --os-type $OS_TYPE \
 --plan-name <name from Step1> \
 --plan-product <product from Step1> \
 --plan-publisher <publisher from Step1> \
 --size $VM_SIZE \
 --nsg ""
```

Thats it. I have outlined two methods for storage, one for short term with snapshot only, and other for more longer term with VHD export to storage.
The VM state will persist between decomission and re-deploy. And with these methods, you can save substantially on cost when the VM is not used, without worrying about eviction with Azure Spot, or resource limits with Azure Dev Box.