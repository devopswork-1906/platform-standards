# Repository Interaction Model

## Purpose

This document defines **how platform repositories interact** and establishes **strict, non-negotiable boundaries** between responsibilities.

The goal is to:
- Prevent responsibility overlap
- Avoid platform drift
- Enforce separation of concerns
- Enable safe scaling across teams
- Provide audit-ready clarity on ownership and execution paths

---

## Table of Contents

1. [Repository Types & Responsibilities](#repository-types--responsibilities)
   - [Bootstrap Repository](#bootstrap-repository)
   - [Blueprint Repository](#blueprint-repository)
   - [Module Repositories](#module-repositories)
   - [Application Repositories](#application-repositories)
   - [Reusable Workflows](#reusable-workflows)
   - [Reusable Actions](#reusable-actions)
2. [Invocation Model](#invocation-model)
   - [Mandatory Control-Plane Invocation](#mandatory-control-plane-invocation)
   - [Allowed Direct Action Invocation (Utility Actions)](#allowed-direct-action-invocation-utility-actions)
3. [Explicit Non-Goals](#explicit-non-goals)
4. [Golden Rule](#golden-rule)
5. [Enforcement](#enforcement)

## Repository Types & Responsibilities

### Bootstrap Repository

**Purpose**
- AWS account initialization
- Terraform backend provisioning
- IAM trust, OIDC provider, and platform roles setup

**Rules**
- Executed **only** by the Platform (IMS Tooling) team
- Triggered **manually** using `workflow_dispatch`
- Protected by GitHub Environments and approval gates
- Executed **once per AWS account**
- Safe to re-run (idempotent)

**Forbidden**
- Application or workload infrastructure
- Repository or module versioning logic
- App-level CI/CD pipelines
- Runtime configuration for applications

> Bootstrap establishes the foundation.  
> It must never evolve into a deployment mechanism.

---

### Blueprint Repository

**Purpose**
- Provide a **standardized template** for Terraform module repositories
- Define:
  - Repository structure
  - CI expectations
  - Documentation layout
  - Versioning integration points

**Rules**
- Consumed via **GitHub “Use this template”**
- Must reference:
  - Platform standards
  - Repository versioning standards
- Evolves centrally; consumers inherit improvements

**Forbidden**
- Account bootstrap logic
- Custom or embedded CI/CD logic
- Org- or environment-specific secrets
- Runtime infrastructure assumptions

> Blueprint defines **how to build modules**, not how to run infrastructure.

---

### Module Repositories

**Purpose**
- Implement **reusable Terraform building blocks**
- Encapsulate AWS service-level infrastructure

**Rules**
- Environment-agnostic
- Stateless and reusable
- Versioned **only via centralized CI**
- Must follow Terraform & IAM standards
- Must expose clear inputs and outputs

**Forbidden**
- Terraform backend creation or configuration
- Account bootstrap execution
- Custom versioning scripts or tagging logic
- Application-specific configuration

> Modules are **libraries**, not deployments.

---

### Application Repositories

**Purpose**
- Consume Terraform modules
- Deploy application-specific infrastructure
- Represent the runtime environment of workloads

**Rules**
- Assume bootstrap is already completed
- Use standardized CI workflows and environments
- Handle environment-specific configuration

**Forbidden**
- Account bootstrap
- Terraform backend creation
- Platform IAM trust manipulation
- Custom CI/CD frameworks outside standards

> Application repos deploy infrastructure.  
> They must never redefine the platform.

---

### Reusable Workflows

**Purpose**
- Centralized **orchestration and policy enforcement**
- Implement platform standards
- Act as the control plane for CI/CD

**Rules**
- Owned and versioned by the Platform team
- Called by repository-level workflows
- Define:
  - Permissions
  - Approval gates
  - Execution order
- May call reusable actions

**Forbidden**
- Repository-specific assumptions
- Hardcoded secrets or credentials
- Environment-specific logic
- Bypassing approval or policy checks

> Reusable workflows decide **what is allowed to happen**.

---

### Reusable Actions

**Purpose**
- Provide **atomic, reusable execution units**
- Encapsulate single, well-defined tasks

**Rules**
- Stateless
- Policy-agnostic
- No implicit assumptions
- Explicit inputs and outputs only

**Forbidden**
- Decision-making or policy enforcement
- Direct repository mutation without workflow approval
- Secrets retrieval unless explicitly passed
- Environment awareness

> Reusable actions do the work —  
> they do **not** decide *whether* the work should happen.

---

## Invocation Model

The platform enforces a **layered invocation model** to ensure clear separation of concerns and safe automation.

### Mandatory Control-Plane Invocation
Any action that performs **policy enforcement**, **state mutation**, or **destructive operations** MUST follow this chain.

The following invocation chain is **mandatory**:
    ```
    Repository Workflow
    ↓
    Reusable Workflow
    ↓
    Reusable Action
    ```

This applies to:
- Versioning and tagging
- Terraform plan/apply
- Infrastructure provisioning
- IAM or backend changes
- Any operation requiring approvals or guardrails

---

### Allowed Direct Action Invocation (Utility Actions)

Repository workflows MAY directly invoke actions that are:

- Stateless
- Policy-agnostic
- Non-destructive
- Limited to setup, authentication, or observability

Examples include:
- OIDC authentication actions
- AWS Account or environment discovery
- Get Github-App token
- Context logging or summary generation

These actions MUST NOT:
- Make policy decisions
- Modify repository state (tags, commits)
- Deploy or destroy infrastructure
- Bypass approval gates

---

## Explicit Non-Goals

The platform model intentionally does **not** support:

- Repositories directly invoking **policy-enforcing or state-mutating actions**
- Application repositories invoking bootstrap workflows
- Modules managing Terraform state backends
- CI logic duplicated across repositories
- Reusable actions implementing policy decisions or approval logic

---

## Golden Rule

> **Repositories trigger workflows.  
> Workflows enforce policy.  
> Actions execute tasks.**

---

## Enforcement

- Violations may be blocked at PR or CI level
- Platform team may disable non-compliant workflows
- Repeated violations may trigger audit review
- Platform standards take precedence over repository conventions

---