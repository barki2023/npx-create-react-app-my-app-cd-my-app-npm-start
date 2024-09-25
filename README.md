# npx-create-react-app-my-app-cd-my-app-npm-start
import os
from azure.identity import DefaultAzureCredential
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.network import NetworkManagementClient

# Azure credentials
subscription_id = 'YOUR_SUBSCRIPTION_ID'  # Replace with your subscription ID
resource_group_name = 'myResourceGroup'
location = 'eastus'  # Change as needed

# Initialize clients
credential = DefaultAzureCredential()
resource_client = ResourceManagementClient(credential, subscription_id)
compute_client = ComputeManagementClient(credential, subscription_id)
network_client = NetworkManagementClient(credential, subscription_id)

# Create resource group
resource_client.resource_groups.create_or_update(resource_group_name, {'location': location})

# Create virtual network
vnet_name = 'myVNet'
subnet_name = 'mySubnet'
network_client.virtual_networks.begin_create_or_update(
    resource_group_name,
    vnet_name,
    {
        'location': location,
        'address_space': {
            'address_prefixes': ['10.0.0.0/16']
        },
        'subnets': [{
            'name': subnet_name,
            'address_prefix': '10.0.0.0/24'
        }]
    }
).result()

# Create public IP address
ip_name = 'myPublicIP'
network_client.public_ip_addresses.begin_create_or_update(
    resource_group_name,
    ip_name,
    {
        'location': location,
        'sku': {
            'name': 'Basic'
        },
        'public_ip_allocation_method': 'Static'
    }
).result()

# Create network interface
nic_name = 'myNIC'
network_client.network_interfaces.begin_create_or_update(
    resource_group_name,
    nic_name,
    {
        'location': location,
        'ip_configurations': [{
            'name': 'myIPConfig',
            'public_ip_address': {
                'id': f"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Network/publicIPAddresses/{ip_name}"
            },
            'subnet': {
                'id': f"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Network/virtualNetworks/{vnet_name}/subnets/{subnet_name}"
            }
        }]
    }
).result()

# Create a Windows 11 virtual machine
vm_name = 'myWin11VM'
compute_client.virtual_machines.begin_create_or_update(
    resource_group_name,
    vm_name,
    {
        'location': location,
        'hardware_profile': {
            'vm_size': 'Standard_DS1_v2'
        },
        'storage_profile': {
            'image_reference': {
                'publisher': 'MicrosoftWindowsDesktop',
                'offer': 'windows-11',
                'sku': 'win11-21h2-pro',
                'version': 'latest'
            },
            'os_disk': {
                'create_option': 'FromImage'
            }
        },
        'os_profile': {
            'computer_name': vm_name,
            'admin_username': 'yourusername',  # Replace with your username
            'admin_password': 'yourpassword123!'  # Replace with a secure password
        },
        'network_profile': {
            'network_interfaces': [{
                'id': f"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Network/networkInterfaces/{nic_name}"
            }]
        }
    }
).result()

print(f"Virtual machine '{vm_name}' created successfully!")
