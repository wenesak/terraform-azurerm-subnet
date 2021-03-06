# Azure network - Subnet
[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/claranet/subnet/azurerm/)

Common Azure module to generate a [Virtual Newtork Subnet](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-manage-subnet). 
This module must be used within a [Virtual Network](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview).

## Requirements

* [AzureRM Terraform provider](https://www.terraform.io/docs/providers/azurerm/) >= 1.31

## Terraform version compatibility

| Module version | Terraform version |
|----------------|-------------------|
| >= 2.x.x       | 0.12.x            |
| <  2.x.x       | 0.11.x            |

## Usage

This module is optimized to work with the [Claranet terraform-wrapper](https://github.com/claranet/terraform-wrapper) tool
which set some terraform variables in the environment needed by this module.
More details about variables set by the `terraform-wrapper` available in the [documentation](https://github.com/claranet/terraform-wrapper#environment).

```hcl
module "azure-region" {
  source  = "claranet/regions/azurerm"
  version = "x.x.x"

  azure_region = var.azure_region
}

module "rg" {
  source  = "claranet/rg/azurerm"
  version = "x.x.x"

  location    = module.azure-region.location
  client_name = var.client_name
  environment = var.environment
  stack       = var.stack
}

module "azure-network-vnet" {
  source  = "claranet/vnet/azurerm"
  version = "x.x.x"
    
  environment      = var.environment
  location         = module.azure-region.location
  location_short   = module.azure-region.location_short
  client_name      = var.client_name
  stack            = var.stack
  custom_vnet_name = var.custom_vnet_name

  resource_group_name = module.rg.resource_group_name
  vnet_cidr           = ["10.10.0.0/16"]
}

module "azure-network-subnet" {
  source  = "claranet/subnet/azurerm"
  version = "x.x.x"

  environment         = var.environment
  location_short      = module.azure-region.location_short
  client_name         = var.client_name
  stack               = var.stack
  custom_subnet_names = var.custom_subnet_names

  resource_group_name  = module.rg.resource_group_name
  virtual_network_name = module.azure-network-vnet.virtual_network_name
  subnet_cidr_list     = ["10.10.10.0/24"]

  # Those lists must be the same size as the associated count value and `subnet_cidr_list` size and or not set (default count value is "0")
  # This limitation should be removed with Terraform 0.12
  route_table_count            = "1"
  route_table_ids              = var.route_table_ids
  network_security_group_count = "1"
  network_security_group_ids   = var.network_security_group_ids

  service_endpoints = var.service_endpoints
}

```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| client\_name | Client name/account used in naming | string | n/a | yes |
| custom\_subnet\_names | Optional custom subnet names | list(string) | `[ "" ]` | no |
| environment | Project environment | string | n/a | yes |
| location\_short | Short string for Azure location. | string | n/a | yes |
| name\_prefix | Optional prefix for subnet names | string | `""` | no |
| network\_security\_group\_count | Count of Network Security Group to associate with the subnet | number | `"1"` | no |
| network\_security\_group\_ids | The Network Security Group Ids list to associate with the subnet | list(string) | `[]` | no |
| resource\_group\_name | Resource group name | string | n/a | yes |
| route\_table\_count | Count of Route Table to associate with the subnet | number | `"0"` | no |
| route\_table\_ids | The Route Table Ids list to associate with the subnet | list(string) | `[]` | no |
| service\_endpoints | The list of Service endpoints to associate with the subnet | list(string) | `[]` | no |
| stack | Project stack name | string | n/a | yes |
| subnet\_cidr\_list | The address prefix list to use for the subnet | list(string) | n/a | yes |
| virtual\_network\_name | Virtual network name | string | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| subnet\_cidr\_list | CIDR list of the created subnets |
| subnet\_ids | IDs of the created subnets |
| subnet\_ip\_configurations | The collection of IP Configurations with IPs within this subnet |
| subnet\_names | Names list of the created subnet |
| subnets\_cidrs\_map | Map with names and CIDRs of the created subnets |
| subnets\_ids\_map | Map with names and IDs of the created subnets |

## Related documentation

Terraform resource documentation: [terraform.io/docs/providers/azurerm/r/subnet.html](https://www.terraform.io/docs/providers/azurerm/r/subnet.html)

Microsoft Azure documentation: [docs.microsoft.com/en-us/azure/virtual-network/virtual-network-manage-subnet](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-manage-subnet)
