# Azure CosmosDB
[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/claranet/cosmos-db/azurerm/)

This terraform module creates an [Azure CosmosDB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction).
[Diagnostic settings](https://docs.microsoft.com/en-us/azure/diagnostics) are also deployed.

<!-- BEGIN_TF_DOCS -->
## Global versioning rule for Claranet Azure modules

| Module version | Terraform version | AzureRM version |
| -------------- | ----------------- | --------------- |
| >= 6.x.x       | 1.x               | >= 3.0          |
| >= 5.x.x       | 0.15.x            | >= 2.0          |
| >= 4.x.x       | 0.13.x / 0.14.x   | >= 2.0          |
| >= 3.x.x       | 0.12.x            | >= 2.0          |
| >= 2.x.x       | 0.12.x            | < 2.0           |
| <  2.x.x       | 0.11.x            | < 2.0           |

## Usage

This module is optimized to work with the [Claranet terraform-wrapper](https://github.com/claranet/terraform-wrapper) tool
which set some terraform variables in the environment needed by this module.
More details about variables set by the `terraform-wrapper` available in the [documentation](https://github.com/claranet/terraform-wrapper#environment).

```hcl
module "azure_region" {
  source  = "claranet/regions/azurerm"
  version = "x.x.x"

  azure_region = var.azure_region
}

module "rg" {
  source  = "claranet/rg/azurerm"
  version = "x.x.x"

  location    = module.azure_region.location
  client_name = var.client_name
  environment = var.environment
  stack       = var.stack
}

module "logs" {
  source  = "claranet/run-common/azurerm//modules/logs"
  version = "x.x.x"

  client_name         = var.client_name
  environment         = var.environment
  stack               = var.stack
  location            = module.azure_region.location
  location_short      = module.azure_region.location_short
  resource_group_name = module.rg.resource_group_name
}

module "cosmosdb" {
  source  = "claranet/cosmos-db/azurerm"
  version = "x.x.x"

  environment    = var.environment
  location       = module.azure_region.location
  location_short = module.azure_region.location_short
  client_name    = var.client_name
  stack          = var.stack

  resource_group_name = module.rg.resource_group_name

  logs_destinations_ids = [module.logs.log_analytics_workspace_id]

  backup = {
    type                = "Periodic"
    interval_in_minutes = 60 * 3 # 3 hours
    retention_in_hours  = 24
  }

  extra_tags = {
    managed_by            = "Terraform"
    foo                   = "bar"
    monitor_autoscale_max = 2
  }
}
```

## Providers

| Name | Version |
|------|---------|
| azurecaf | ~> 1.1 |
| azurerm | ~> 3.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| diagnostics | claranet/diagnostic-settings/azurerm | 5.0.0 |

## Resources

| Name | Type |
|------|------|
| [azurecaf_name.cosmosdb](https://registry.terraform.io/providers/aztfmod/azurecaf/latest/docs/resources/name) | resource |
| [azurerm_cosmosdb_account.db](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/cosmosdb_account) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| allowed\_cidrs | CosmosDB Firewall Support: This value specifies the set of IP addresses or IP address ranges in CIDR form to be included as the allowed list of client IP's for a given database account. | `list(string)` | `[]` | no |
| analytical\_storage\_enabled | Enable Analytical Storage option for this Cosmos DB account. Defaults to `false`. Changing this forces a new resource to be created. | `bool` | `false` | no |
| analytical\_storage\_type | The schema type of the Analytical Storage for this Cosmos DB account. Possible values are `FullFidelity` and `WellDefined`. | `string` | `null` | no |
| backup | Backup block with type (Continuous / Periodic), interval\_in\_minutes and retention\_in\_hours keys | `map(string)` | <pre>{<br>  "interval_in_minutes": 180,<br>  "retention_in_hours": 168,<br>  "type": "Periodic"<br>}</pre> | no |
| capabilities | Configures the capabilities to enable for this Cosmos DB account:<br>Possible values are<br>  AllowSelfServeUpgradeToMongo36, DisableRateLimitingResponses,<br>  EnableAggregationPipeline, EnableCassandra, EnableGremlin,EnableMongo, EnableTable, EnableServerless,<br>  MongoDBv3.4 and mongoEnableDocLevelTTL. | `list(string)` | `[]` | no |
| client\_name | Name of client | `string` | n/a | yes |
| consistency\_policy\_level | Consistency policy level. Allowed values are `BoundedStaleness`, `Eventual`, `Session`, `Strong` or `ConsistentPrefix` | `string` | `"BoundedStaleness"` | no |
| consistency\_policy\_max\_interval\_in\_seconds | When used with the Bounded Staleness consistency level, this value represents the time amount of staleness (in seconds) tolerated. Accepted range for this value is 5 - 86400 (1 day). Defaults to 5. Required when consistency\_level is set to BoundedStaleness. | `number` | `10` | no |
| consistency\_policy\_max\_staleness\_prefix | When used with the Bounded Staleness consistency level, this value represents the number of stale requests tolerated. Accepted range for this value is 10 – 2147483647. Defaults to 100. Required when consistency\_level is set to BoundedStaleness. | `number` | `200` | no |
| custom\_diagnostic\_settings\_name | Custom name of the diagnostics settings, name will be 'default' if not set. | `string` | `"default"` | no |
| custom\_server\_name | Custom Server Name identifier | `string` | `""` | no |
| default\_tags\_enabled | Option to enable or disable default tags. | `bool` | `true` | no |
| environment | Name of application's environnement | `string` | n/a | yes |
| extra\_tags | Map of custom tags. | `map(string)` | `{}` | no |
| failover\_locations | The name of the Azure region to host replicated data and their priority. | `map(map(string))` | `null` | no |
| is\_virtual\_network\_filter\_enabled | Enables virtual network filtering for this Cosmos DB account | `bool` | `false` | no |
| kind | Specifies the Kind of CosmosDB to create - possible values are `GlobalDocumentDB` and `MongoDB`. | `string` | `"GlobalDocumentDB"` | no |
| location | Azure region to use | `string` | n/a | yes |
| location\_short | Short string for Azure location/region | `string` | n/a | yes |
| logs\_categories | Log categories to send to destinations. | `list(string)` | `null` | no |
| logs\_destinations\_ids | List of destination resources Ids for logs diagnostics destination. Can be Storage Account, Log Analytics Workspace and Event Hub. No more than one of each can be set. Empty list to disable logging. | `list(string)` | n/a | yes |
| logs\_metrics\_categories | Metrics categories to send to destinations. | `list(string)` | `null` | no |
| logs\_retention\_days | Number of days to keep logs on storage account | `number` | `30` | no |
| mongo\_server\_version | The Server Version of a MongoDB account. Possible values are `4.0`, `3.6`, and `3.2`. | `string` | `"4.0"` | no |
| name\_prefix | Optional prefix for the generated name | `string` | `""` | no |
| name\_suffix | Optional suffix for the generated name | `string` | `""` | no |
| offer\_type | Specifies the Offer Type to use for this CosmosDB Account - currently this can only be set to Standard. | `string` | `"Standard"` | no |
| resource\_group\_name | Name of the application ressource group, herited from infra module | `string` | n/a | yes |
| stack | Name of application stack | `string` | n/a | yes |
| use\_caf\_naming | Use the Azure CAF naming provider to generate default resource name. `custom_server_name` override this if set. Legacy default name is used if this is set to `false`. | `bool` | `true` | no |
| virtual\_network\_rule | Specifues a virtual\_network\_rules resource used to define which subnets are allowed to access this CosmosDB account | <pre>list(object({<br>    id                                   = string,<br>    ignore_missing_vnet_service_endpoint = bool<br>  }))</pre> | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| cosmosdb\_connection\_strings | A list of connection strings available for this CosmosDB account. |
| cosmosdb\_endpoint | The endpoint used to connect to the CosmosDB account. |
| cosmosdb\_id | The CosmosDB Account ID. |
| cosmosdb\_name | The CosmosDB Account Name. |
| cosmosdb\_primary\_master\_key | The Primary master key for the CosmosDB Account. |
| cosmosdb\_primary\_readonly\_master\_key | The Primary read-only master Key for the CosmosDB Account. |
| cosmosdb\_read\_endpoints | A list of read endpoints available for this CosmosDB account. |
| cosmosdb\_secondary\_master\_key | The Secondary master key for the CosmosDB Account. |
| cosmosdb\_secondary\_readonly\_master\_key | The Secondary read-only master key for the CosmosDB Account. |
| cosmosdb\_write\_endpoints | A list of write endpoints available for this CosmosDB account. |
<!-- END_TF_DOCS -->
## Related documentation

  * [Microsoft Azure documentation](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)
