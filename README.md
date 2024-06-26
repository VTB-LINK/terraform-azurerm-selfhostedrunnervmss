<!-- BEGIN_TF_DOCS -->
# Self Hosted Runners Virtual Machine Scale Set

This module deploys a virtual machine scale set for self hosted runners for Azure DevOps and GitHub.

```hcl
provider "azurerm" {
  features {}
}

module "vmss" {
  source                         = "amestofortytwo/selfhostedrunnervmss/azurerm"
  operating_system               = "ubuntu"       # windows or ubuntu
  runner_platform                = "azure_devops" # azure_devops or github
}
```

After deploying the virtual machine scale set, you need to configure the Azure DevOps or GitHub side of things according to our documentation:

- [Configure Azure DevOps Agent Pool](https://docs.byfortytwo.com/Self%20Hosted%20Runners/Azure%20DevOps/step2/)
- [Configure GitHub](https://docs.byfortytwo.com/Self%20Hosted%20Runners/GitHub/step2/)

## Requirements

No requirements.

## Examples

### Minimum

```hcl
provider "azurerm" {
  features {}
}

module "vmss" {
  source               = "amestofortytwo/selfhostedrunnervmss/azurerm"
  operating_system     = "ubuntu"       # windows or ubuntu
  runner_platform      = "azure_devops" # azure_devops or github
  deploy_load_balancer = true
}

output "password" {
  value = nonsensitive(module.vmss.password)
}
```

### Advanced

```hcl
provider "azurerm" {
  features {}
}

# Create custom rg
resource "azurerm_resource_group" "rg" {
  location = "westeurope"
  name     = "runners"
}

# Create custom vnet
resource "azurerm_virtual_network" "vmss" {
  name                = "runner-network"
  address_space       = ["10.0.0.0/24"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "vmss" {
  name                 = "vmss"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vmss.name
  address_prefixes     = azurerm_virtual_network.vmss.address_space
}

module "vmss" {
  source                          = "amestofortytwo/selfhostedrunnervmss/azurerm"
  operating_system                = "ubuntu"       # windows or ubuntu
  runner_platform                 = "azure_devops" # azure_devops or github
  resource_group_name             = azurerm_resource_group.rg.name
  use_existing_resource_group     = true
  location                        = azurerm_resource_group.rg.location
  virtual_machine_scale_set_name  = "runners"
  sku                             = "Standard_D2s_v3"
  ssh_public_keys                 = ["ssh-rsa AAAAB3NzaC1yc2EAAAADA....QFv2PJ0= marius@42device"]
  subnet_id                       = azurerm_subnet.vmss.id
  use_custom_subnet               = true
  vmss_encryption_at_host_enabled = true
}

output "password" {
  value = nonsensitive(module.vmss.password)
}
```

## Providers

| Name | Version |
|------|---------|
| <a name="provider_azurerm"></a> [azurerm](#provider\_azurerm) | n/a |
| <a name="provider_random"></a> [random](#provider\_random) | n/a |

## Modules

No modules.

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_deploy_load_balancer"></a> [deploy\_load\_balancer](#input\_deploy\_load\_balancer) | (Optional) When using the built-in network (use\_custom\_subnet is false), should we create a NAT gateway? This will be required in the future. Defaults to false. | `bool` | `false` | no |
| <a name="input_enable_accelerated_networking"></a> [enable\_accelerated\_networking](#input\_enable\_accelerated\_networking) | (Optional) Does this Network Interface support Accelerated Networking? Possible values are true and false. Defaults to false. | `bool` | `false` | no |
| <a name="input_enable_automatic_instance_repair"></a> [enable\_automatic\_instance\_repair](#input\_enable\_automatic\_instance\_repair) | Enable automatic instance repair for the VMSS. This will automatically repair instances that fail health checks. | `bool` | `false` | no |
| <a name="input_enable_termination_notifications"></a> [enable\_termination\_notifications](#input\_enable\_termination\_notifications) | Enable termination notifications for the VMSS. This will send a notification to the Azure Instance Metadata Service (IMDS) when the VMSS is scheduled for maintenance or when the VMSS is deleted. | `bool` | `false` | no |
| <a name="input_load_balancer_backend_address_pool_id"></a> [load\_balancer\_backend\_address\_pool\_id](#input\_load\_balancer\_backend\_address\_pool\_id) | (Optional) Value of the backend address pool id to use for the load balancer. I.e. for static outbound NAT. | `string` | `""` | no |
| <a name="input_location"></a> [location](#input\_location) | The Azure region to create the scale set in | `string` | `"westeurope"` | no |
| <a name="input_operating_system"></a> [operating\_system](#input\_operating\_system) | The OS of the runners | `string` | `"ubuntu"` | no |
| <a name="input_password"></a> [password](#input\_password) | Password of the local user acocunt | `string` | `null` | no |
| <a name="input_resource_group_name"></a> [resource\_group\_name](#input\_resource\_group\_name) | The resource group name to create | `string` | `"self-hosted-runners"` | no |
| <a name="input_runner_platform"></a> [runner\_platform](#input\_runner\_platform) | Whether it is github or azure\_devops used for runners | `string` | `"azure_devops"` | no |
| <a name="input_sku"></a> [sku](#input\_sku) | The sku to create virtual machines with | `string` | `"Standard_D2s_v3"` | no |
| <a name="input_ssh_public_keys"></a> [ssh\_public\_keys](#input\_ssh\_public\_keys) | n/a | `list(string)` | `[]` | no |
| <a name="input_subnet_id"></a> [subnet\_id](#input\_subnet\_id) | When provided, this subnet will be used for the scale set, rather than creating a new virtual network and subnet | `string` | `null` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | n/a | `map(any)` | `{}` | no |
| <a name="input_use_custom_subnet"></a> [use\_custom\_subnet](#input\_use\_custom\_subnet) | Set to true if subnet\_id is provided in order to actually use it (works around a TF issue) | `bool` | `false` | no |
| <a name="input_use_existing_resource_group"></a> [use\_existing\_resource\_group](#input\_use\_existing\_resource\_group) | Whether to use an existing resource group or not | `bool` | `false` | no |
| <a name="input_username"></a> [username](#input\_username) | Username of the local user account | `string` | `"runneradmin"` | no |
| <a name="input_virtual_machine_scale_set_name"></a> [virtual\_machine\_scale\_set\_name](#input\_virtual\_machine\_scale\_set\_name) | n/a | `string` | `"self-hosted-runners"` | no |
| <a name="input_vmss_encryption_at_host_enabled"></a> [vmss\_encryption\_at\_host\_enabled](#input\_vmss\_encryption\_at\_host\_enabled) | Enables encryption at host for the VMSS virtual machines. In order to use this option, the EncryptionAtHost feature must be enabled for Microsoft.Compue resource provider must be enabled for the subscription. To enable, use this PowerShell command: Register-AzProviderFeature -FeatureName 'EncryptionAtHost' -ProviderNamespace 'Microsoft.Compute'. | `bool` | `false` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_password"></a> [password](#output\_password) | n/a |
| <a name="output_virtual_machine_scale_set_id"></a> [virtual\_machine\_scale\_set\_id](#output\_virtual\_machine\_scale\_set\_id) | n/a |

## Resources

| Name | Type |
|------|------|
| [azurerm_lb.load_balancer](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/lb) | resource |
| [azurerm_lb_backend_address_pool.load_balancer](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/lb_backend_address_pool) | resource |
| [azurerm_lb_outbound_rule.outbound_rule](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/lb_outbound_rule) | resource |
| [azurerm_linux_virtual_machine_scale_set.self_hosted_runners](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_virtual_machine_scale_set) | resource |
| [azurerm_public_ip.load_balancer_pip](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/public_ip) | resource |
| [azurerm_resource_group.rg](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group) | resource |
| [azurerm_subnet.vmss](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet) | resource |
| [azurerm_virtual_network.vmss](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network) | resource |
| [azurerm_windows_virtual_machine_scale_set.self_hosted_runners](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/windows_virtual_machine_scale_set) | resource |
| [random_password.password](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) | resource |
<!-- END_TF_DOCS -->