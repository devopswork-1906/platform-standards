# GitHub Actions Standards

## Purpose

This document defines **mandatory standards** for all GitHub Actions workflows, reusable workflows, and reusable actions used across the organization.

Goals:
- Enforce least-privilege CI/CD execution
- Prevent credential leakage and privilege escalation
- Ensure auditability and consistency
- Enable safe reuse at scale
- Align with enterprise security and compliance expectations

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [Permissions Model (Critical)](#permissions-model-critical)
   - [Explicit Permissions Required](#explicit-permissions-required)
   - [## Permission Scope Guidelines](#permission-scope-guidelines)
3. [Secrets Management](#secrets-management)
   - [Approved Secret Sources](#approved-secret-sources)
   - [Forbidden Patterns](#forbidden-patterns)
4. [Authentication & Cloud Access](#authentication--cloud-access)
5. [Role Assumption Rules](#role-assumption-rules)
6. [Environments & Approval Gates](#environments--approval-gates)
   - [Environment Usage](#environment-usage)
   - [Approval Requirements](#approval-requirements)
7. [Workflow Structure Standards](#workflow-structure-standards)
   - [Repository-Level Workflows](#repository-level-workflows)
   - [Reusable Workflows](#reusable-workflows)
   - [Reusable Actions](#reusable-actions)
8. [Logging & Observability](#logging--observability)
9. [Secret Safety](#secret-safety)
10. [Prohibited Practices (Strict)](#prohibited-practices-strict)
11. [Version Pinning](#version-pinning)

## Core Principles

All GitHub Actions usage MUST follow:

- **Least Privilege** – grant only required permissions
- **Explicit Permissions** – never rely on defaults
- **Separation of Duties** – approvals before destructive actions
- **Centralized Control** – logic lives in reusable workflows
- **Auditability** – every execution must be traceable

---

## Permissions Model (Critical)

### Explicit Permissions Required

All workflows MUST explicitly declare permissions.

```yaml
# Implicit default permissions
permissions:
  contents: read
  id-token: write
```

### Permission Scope Guidelines

| Permission             | Usage                                   |
|------------------------|-----------------------------------------|
| `contents: read`       | Checkout, read-only repository access   |
| `contents: write`      | Tagging and releases (restricted, centralized only)|
| `id-token: write`      | OIDC authentication to cloud providers  |
| `pull-requests: read`  | Pull request metadata inspection        |
| `pull-requests: write` | Commenting and labeling (restricted)    |

Rules:
- Write permissions must be explicitly justified
- Tag creation is allowed only in centralized reusable workflows
- Reusable actions must not elevate permissions

## Secrets Management
- No hardcoded secrets
- No plaintext secrets in workflows
- No secrets committed to repositories
- No secrets written to logs or files
- Secrets should be handled from AWS secret Manager

### Approved Secret Sources

AWS Secrets Manager
- Centralized secret storage
- Accessed at runtime via OIDC-assumed IAM roles
- Never persisted to disk

### Forbidden Patterns
- Base64-encoded secrets in environment variables
- Echoing secrets (even if masked)
- Writing secrets to files
- Passing secrets between jobs via artifacts

## Authentication & Cloud Access

OIDC Authentication (Mandatory)
- Static credentials (access keys, PATs) are forbidden
- All cloud authentication must use:
  - GitHub Actions OIDC
  - Short-lived credentials

This applies to:
- AWS
- Any cloud or SaaS platform supporting OIDC

## Role Assumption Rules

IAM roles assumed by GitHub Actions MUST:
- Be tightly scoped
- Restrict trust by:
  - Repository
  - Branch
  - Environment (where applicable)

---

## Environments & Approval Gates

### **Environment Usage**

GitHub Environments MUST be used for:
- Production deployments
- Terraform apply
- Bootstrap workflows
- Any destructive or irreversible action

---

### **Approval Requirements**
- At least one human approval required for apply
- Production environments must enforce:
- Mandatory reviewers
- Optional wait timers
- All approvals must be auditable

---

## Workflow Structure Standards

### Repository-Level Workflows

Repo workflows MUST:
- Be minimal
- Contain no business logic
- Delegate execution to reusable workflows

Responsibilities limited to:
- Trigger definitions
- Passing inputs
- Selecting environments

---

### Reusable Workflows

Reusable workflows:
- Own orchestration and policy enforcement
- Validate inputs
- Control permissions
- Call reusable actions

Reusable workflows **MUST NOT**:
- Contain repository-specific logic
- Store or fetch secrets directly
- Bypass approval gates

---

### Reusable Actions

Reusable actions:
- Perform atomic, well-defined tasks
- Must be stateless
- Must not make policy decisions
- Must not access secrets unless explicitly passed

---

## Logging & Observability

All workflows MUST log:
- Repository name
- Workflow name
- Job name
- Actor
- Target environment

This ensures:
- Audit traceability
- Faster incident response
- Easier debugging

---

## Secret Safety
- Secrets must never appear in logs
- Masking must be verified
- Debug logging disabled by default in production workflows

---

## Prohibited Practices (Strict)

The following are not allowed:
- Floating references (@main, @latest) in production
- Direct terraform apply without approval
- Self-hosted runners without platform approval
- Unverified third-party actions
- Executing scripts fetched at runtime
- Modifying IAM trust or Terraform backend from repo workflows

---

## Version Pinning
- All actions and reusable workflows MUST be version-pinned
- Floating tags allowed only in non-production experimentation
- Production paths require immutable versions or tags