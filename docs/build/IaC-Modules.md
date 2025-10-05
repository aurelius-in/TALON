# IaC Module Specifications

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑10‑05  

---

## 1  Purpose and Scope

Infrastructure as Code (IaC) modules encapsulate repeatable provisioning logic so that developers and operations teams can consume a standard pattern rather than copy‑pasting configuration.  
In the context of **TALON**, modules provide the building blocks for provisioning network slices, compute clusters, vision pipelines and other components required by Blue Eagle Robotics.  
This specification defines how modules should be structured, how inputs and outputs are handled, examples of usage, security and secrets management, and versioning guidelines.  It applies to all Terraform modules authored by RAIN for BER.

## 2  Module Structure

Modules must follow a predictable layout to simplify maintenance, onboarding and reuse.  At a minimum each module **must** include:

- `main.tf` — contains the core resources.  
- `variables.tf` — defines configurable inputs for the module.  
- `outputs.tf` — declares values returned to the calling configuration.  
- `versions.tf` — declares the required Terraform version and provider versions to prevent incompatibility.  
- `README.md` — documents the module’s purpose, required and optional variables, outputs, usage examples and any assumptions.  
- optional files such as `locals.tf` (for local values) or `data.tf` (for data sources) if needed.

A `.gitignore` should be included to exclude state files and transient outputs from source control, and a `terraform.tfvars.example` file can illustrate variable definitions and defaults.  Consistent directory names (for example `examples/` for usage samples, `test/` for Terratest code) promote discoverability.

## 3  Inputs and Outputs

### 3.1  Variable Definitions

Variables are the mechanism that makes a module configurable.  Variables **must** be clearly defined in `variables.tf` with a `description` and, where appropriate, a `type` and `default`.  The following conventions apply:

- **Required inputs:** Variables without defaults.  These represent deliberate choices the caller must provide; the module fails if they are omitted.  Example: a VPC identifier or cluster name.
- **Optional inputs:** Variables with sensible defaults.  Defaults should represent common use cases but still be override‑able.  Document the default in the README.
- **Data types:** Specify types for all variables (`string`, `number`, `bool`, `list` or `map`) for clarity and to catch mistakes at plan time.
- **Naming:** Use lowercase letters and underscores to separate words, never dashes.  Names should be descriptive but not repeat the resource type (avoid `aws_instance_instance`).  Use singular nouns for single values and plural nouns for lists or maps.
- **Ordering:** In variable blocks, order attributes as `description`, `type`, `default` then `validation`.

### 3.2  Outputs

Outputs expose resource attributes back to the caller.  Each output should have a human‑readable `description` and return only necessary information.  Wrap important values in user‑friendly templates, and avoid exposing sensitive data such as passwords or API keys as outputs.  When returning lists or maps, use plural names to convey multiplicity.

### 3.3  Tagging

Resources created by modules should support tagging and accept a `tags` variable wherever possible.  Tags should be included as the last argument in the resource block, after core attributes, and before optional `depends_on` or `lifecycle` blocks.  Tags help cost allocation and compliance reporting.

## 4  Example Module

The following example illustrates a minimal module used to create a resource group and log analytics workspace for Azure‑based TALON deployments.

**Directory layout**

```
example-module/
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── README.md
```

**main.tf**

```hcl
resource "azurerm_resource_group" "this" {
  name     = "${var.prefix}-rg"
  location = var.location
  tags     = var.tags
}

resource "azurerm_log_analytics_workspace" "this" {
  name                = "${var.prefix}-law"
  location            = var.location
  resource_group_name = azurerm_resource_group.this.name
  sku                 = "PerGB2018"
  tags                = var.tags
}
```

**variables.tf**

```hcl
variable "prefix" {
  description = "Prefix used for resource naming"
  type        = string
}

variable "location" {
  description = "Azure region for deployment"
  type        = string
  default     = "eastus"
}

variable "tags" {
  description = "Optional tags to apply to all resources"
  type        = map(string)
  default     = {}
}
```

**outputs.tf**

```hcl
output "resource_group_name" {
  description = "Name of the created resource group"
  value       = azurerm_resource_group.this.name
}

output "workspace_id" {
  description = "ID of the Log Analytics workspace"
  value       = azurerm_log_analytics_workspace.this.id
}
```

This example shows how required and optional variables are defined, how outputs are specified, and how tags are applied consistently.  A corresponding `README.md` would describe the module’s purpose, inputs, outputs and provide usage examples.

## 5  Versioning and Release Management

Modules are treated like libraries and must be versioned.  Follow these guidelines:

- **Source control:** Store each module in its own repository with version tags.  Use semantic versioning (e.g. `v1.2.0`) to indicate breaking, minor or patch changes.
- **Immutable releases:** Once a version is tagged, the module must not be changed.  Changes require a new version tag to avoid breaking consumers.
- **Change log:** Maintain a `CHANGELOG.md` detailing features, fixes and breaking changes for each release.
- **Ownership:** Assign an owner or team responsible for maintaining each module.
- **Consumption:** Encourage callers to reference modules by tag rather than a branch to ensure reproducibility.

## 6  Security and Secrets Management

Handling sensitive data correctly is crucial for compliance and safety:

- **Never store secrets in state files or plain‑text code:** Terraform state should not include passwords or API keys.  Do not hardcode secrets in module configuration.
- **Pass secrets via variables or environment:** Use input variables for sensitive data and set their values at runtime, for example via `TF_VAR_<name>` environment variables.  Do not output sensitive variables from modules.
- **Use secrets management tools:** When possible, retrieve secrets from services like HashiCorp Vault or AWS Secrets Manager rather than storing them locally.
- **Rotate keys and credentials:** Implement periodic key rotation through your secret management system to limit the impact of credential exposure.

By following these practices, modules remain generic and safe to consume in different environments without leaking credentials or secrets.

## 7  Testing and Documentation

Before publishing a module, validate its behaviour and document it thoroughly:

- **Testing:** Use `terraform validate` and `terraform plan` to catch syntax and configuration errors.  For complex modules, implement automated tests using tools such as Terratest or the open‑source `kitchen-terraform`.  Testing modules in isolation prevents regressions when they are consumed by larger stacks.
- **Documentation:** Every module must include a README file explaining its purpose, inputs, outputs and an example usage.  Clear documentation improves usability and encourages collaboration.
- **Continuous Integration:** Integrate modules with CI/CD pipelines to run validation, unit tests and enforce policies such as linting and security scans on each change.

## 8  Additional Best Practices

To ensure high‑quality modules:

- **Keep modules focused:** A module should do one thing well.  Combining unrelated resources increases complexity and makes reuse harder.
- **Encapsulate infrastructure appropriately:** Group resources that are always deployed together and respect privilege boundaries.  Separate long‑lived infrastructure (e.g. databases) from rapidly changing components (e.g. compute nodes).
- **Collaboration:** Maintain an internal roadmap for modules, prioritise features based on user feedback, and adopt community‑style contribution practices.
- **Policy integration:** Use tools like Sentinel or Open Policy Agent (OPA) to enforce organisational policies (naming conventions, tag requirements, region restrictions) at plan time.

## 9  Conclusion

Well‑designed IaC modules enable RAIN to deliver repeatable, secure and auditable infrastructure for Blue Eagle Robotics.  By adhering to the structure, naming, versioning, and security guidelines described here, teams can build a library of reliable modules that accelerate deployment while reducing risk.  Continuous testing, documentation and collaboration ensure that modules evolve safely over time.
