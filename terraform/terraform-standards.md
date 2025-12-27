# Terraform Standards

## ## Table of Contents

1. [Purpose](#purpose)
2. [Terraform Version](#terraform-version)
3. [Providers](#providers)
4. [Module Design Standards](#module-design-standards)
5. [State Management](#state-management)
   - [Remote State](#remote-state)
   - [Backend Configuration](#backend-configuration)
   - [State Isolation](#state-isolation)
6. [Lifecycle & Safety Controls](#lifecycle--safety-controls)
   - [Resource Protection](#resource-protection)
   - [Replacement Safety](#replacement-safety)
   - [Destructive Changes](#destructive-changes)
7. [Tagging](#tagging)
   - [Mandatory Tagging](#mandatory-tagging)
   - [CI Context Tags](#ci-context-tags)
8. [IAM Integration](#iam-integration)
9. [Versioning](#versioning)
10. [CI & Execution Model](#ci--execution-model)
11. [Documentation Requirements](#documentation-requirements)
12. [Forbidden Practices](#forbidden-practices)

---

## Purpose

This document defines **mandatory standards** for all Terraform code managed across the organization.

Goals:
- Ensure consistency, safety, and predictability
- Reduce blast radius of infrastructure changes
- Enable scalable reuse through modules
- Support strong auditability and automation
- Align Terraform usage with platform governance

---

## Terraform Version

- Minimum required Terraform version: `>= 1.10.0`
- The version MUST be pinned using `required_version`
- Repositories MUST NOT rely on developer-local Terraform versions

Example:
```hcl
terraform {
  required_version = ">= 1.10.0"
}
```

---

## Providers
- All providers MUST define explicit version constraints
- Avoid overly loose constraints (e.g. >= 3.0 without upper bound awareness)
- Provider versions must be compatible with the minimum Terraform version
Example:

```hcl
required_providers {
  aws = {
    source  = "hashicorp/aws"
    version = ">= 5.30"
  }
}
```

---

## Module Design Standards
Terraform modules MUST follow these principles:

- Single responsibility
  - One module = one logical concern
- Environment-agnostic
  - No hardcoded environment names
  - No hardcoded account IDs
- Region-Agnostic
  - Regions must be passed as inputs or injected by the caller
- No Embedded Providers
  - Providers must be injected from the root module
- Typed Inputs
  - All variables must declare type
  - Complex inputs must use objects or maps
- Input Validation
  - Use validation blocks for critical constraints
- Minimal Outputs
  - Expose only what consumers need
  - Mark sensitive outputs explicitly

Modules are libraries, not deployments.

---

## State Management

### Remote State

- Local state is not allowed
- All Terraform state MUST be stored remotely
- Remote state only (S3 bucket)

### Backend Configuration

- Backend configuration MUST NOT be hardcoded in modules
- Backend configuration MUST be injected externally
(e.g. via backend config files or CI-generated configs)

### State Isolation

- One Terraform state per:
  - Environment
  - Workload / application
- Shared state across unrelated workloads is forbidden

---

## Lifecycle & Safety Controls

### Resource Protection
- prevent_destroy = true MUST be used for critical resources, including:
  - Terraform state buckets
  - Logging buckets
  - Security-sensitive resources

### Replacement Safety

- `create_before_destroy` = true MUST be used where supported and safe
- Explicit depends_on MUST be declared when implicit dependencies are insufficient

### Destructive Changes
- Destructive operations MUST be gated by:
  - CI approval
  - GitHub Environments
- No direct terraform apply from developer machines in production environments

---

## Tagging

### Mandatory Tagging

All resources MUST include standardized tags, including (but not limited to):
- Environment
- Cost Centre
- Application / Workload identifier

### CI Context Tags

CI-driven Terraform executions MUST propagate GitHub context tags, such as:
- Repository name
- Workflow name
- Workflow run URL
- Actor
- PR number (where applicable)

Tags MUST be merged centrally (e.g. via locals) and not hardcoded per resource.

---

## IAM Integration
- All IAM resources MUST comply with `iam-standards.md`
- Least privilege is mandatory
- Inline policies are allowed only under documented conditions
- Trust policies must follow platform authentication models

Terraform modules MUST NOT bypass IAM standards.

---

## Versioning
Terraform repositories MUST comply with `cicd/repository-versioning.md`

Rules:
- No manual version bumps
- No local git tagging
- CI is the sole authority for releases

---

## CI & Execution Model
- Terraform execution MUST happen via CI pipelines
- CI pipelines MUST:
  - Run terraform fmt
  - Run terraform validate
  - Run terraform plan before apply
- Production applies MUST require human approval

---

## Documentation Requirements
Every Terraform module MUST include:
- README.md explaining purpose and usage
- Usage examples
- Inputs and outputs tables
- Assumptions, limitations

Documentation MUST be kept in sync with code.

---

## Forbidden Practices

The following are not allowed:
- Hardcoded credentials
- Hardcoded backend configuration in modules
- Environment-specific logic inside reusable modules
- Unversioned or ad-hoc Terraform execution
- Console-driven infrastructure changes
- Drift introduced outside Terraform