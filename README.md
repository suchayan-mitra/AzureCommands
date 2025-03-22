# Azure Resource Cleanup Utility

A collection of commands to help identify and clean up unused Azure resources to optimize costs.

## Overview

This utility provides a set of Azure CLI commands to help you identify resources that may no longer be needed, such as:

- Orphaned snapshots
- Unused managed disks
- Disconnected network interfaces
- Unassigned public IP addresses
- Detached network security groups

## Prerequisites

- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) installed
- Appropriate permissions to view resources in your subscription

## Usage

### Authentication

Log in to your Azure account:

```bash
az login
```

### Resource Identification Commands

Below are commands to help identify potentially unused resources.

#### Snapshots

```bash
# List all snapshots with creation date and size
az snapshot list --resource-group YOUR_RESOURCE_GROUP --query "[].{Name:name, CreationDate:timeCreated, SizeGB:diskSizeGB, Location:location}" -o table

# Get detailed information for a specific snapshot
az snapshot show --resource-group YOUR_RESOURCE_GROUP --name SNAPSHOT_NAME
```

#### Managed Disks

```bash
# List all disks with creation date, size, and whether they're attached to a VM
az disk list --resource-group YOUR_RESOURCE_GROUP --query "[].{Name:name, CreationDate:timeCreated, SizeGB:diskSizeGB, ManagedBy:managedBy, Location:location}" -o table

# Disks with empty "ManagedBy" field are not attached to any VM and may be candidates for cleanup
```

#### Network Interfaces

```bash
# List all NICs and whether they're attached to a VM
az network nic list --resource-group YOUR_RESOURCE_GROUP --query "[].{Name:name, IPAddress:ipConfigurations[0].privateIpAddress, AttachedTo:virtualMachine.id, Location:location}" -o table

# NICs with null "AttachedTo" value are not connected to a VM
```

#### Public IP Addresses

```bash
# List all public IPs and whether they're attached to any resource
az network public-ip list --resource-group YOUR_RESOURCE_GROUP --query "[].{Name:name, IPAddress:ipAddress, AttachedTo:ipConfiguration.id, CreationDate:creationTime, Location:location}" -o table

# IPs with null "AttachedTo" value are not in use
```

#### Network Security Groups

```bash
# List all NSGs and if they're attached to a subnet or NIC
az network nsg list --resource-group YOUR_RESOURCE_GROUP --query "[].{Name:name, Subnets:subnets, NICs:networkInterfaces, Location:location}" -o table

# NSGs with empty arrays for both Subnets and NICs are not attached to anything
```

#### VM Images

```bash
# List all VM images with creation date
az image list --resource-group YOUR_RESOURCE_GROUP --query "[].{Name:name, CreationDate:tags.creationDate || 'N/A', Location:location}" -o table
```

#### Resource Costs

```bash
# List consumption/usage details for the past 30 days
az consumption usage list --query "[?contains(instanceId, 'YOUR_RESOURCE_GROUP')].{ResourceName:instanceName, ResourceType:resourceType, Cost:pretaxCost, Currency:currency, Date:usageEnd}" -o table
```

### Cleanup Commands (Caution Required)

After identifying resources to delete, you can use these commands to clean them up. **Use with caution!**

#### Delete Snapshots

```bash
az snapshot delete --resource-group YOUR_RESOURCE_GROUP --name SNAPSHOT_NAME --yes
```

#### Delete Orphaned Disks

```bash
az disk delete --resource-group YOUR_RESOURCE_GROUP --name DISK_NAME --yes
```

#### Delete Unused Network Interfaces

```bash
az network nic delete --resource-group YOUR_RESOURCE_GROUP --name NIC_NAME
```

#### Delete Unassigned Public IPs

```bash
az network public-ip delete --resource-group YOUR_RESOURCE_GROUP --name IP_NAME
```

#### Delete Detached NSGs

```bash
az network nsg delete --resource-group YOUR_RESOURCE_GROUP --name NSG_NAME
```

## Best Practices

1. **Always review** resources before deletion
2. **Test in non-production** environments first
3. **Take a backup or snapshot** before making significant changes
4. Consider using **Azure Policies** to prevent orphaned resource creation
5. Set up regular **cost analysis reviews**

## Contribution

Feel free to contribute to this project by adding new commands or improving existing ones.

## Disclaimer

Use these scripts at your own risk. Always verify resources before deletion. The author is not responsible for any data loss or service disruption.

## License

MIT
