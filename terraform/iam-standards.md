# IAM Standards

## Table of Contents

1. [Purpose](#purpose)
2. [Core Principles](#core-principles)
3. [IAM Policy Types & Usage](#iam-policy-types--usage)
4. [Trust Policy Standards](#trust-policy-standards)
5. [Forbidden Patterns (Unless Explicitly Approved)](#forbidden-patterns-unless-explicitly-approved)
6. [IAM Role Classification](#iam-role-classification)
7. [Review & Audit Expectations](#review--audit-expectations)

## Purpose

This document defines **mandatory IAM standards** for all AWS infrastructure
managed via Terraform across the organization.

Goals:
- Enforce least privilege
- Maintain clear trust boundaries
- Enable scalability and auditability
- Allow application flexibility without compromising governance
- Support safe AI-assisted development

---

## Core Principles

All IAM design MUST adhere to:

- **Least Privilege** – grant only what is required
- **Explicit Trust** – no implicit or inferred access
- **Clear Ownership** – every role and policy must have an owner
- **Auditability** – permissions must be discoverable and reviewable
- **Terraform-Only Management** – no console drift or manual changes

---

## IAM Policy Types & Usage

### AWS Managed Policies
**Status:** Allowed

Use when:
- The policy exactly matches the requirement
- No over-permissioning is introduced

Notes:
- Prefer read-only variants where possible
- Do not attach broad admin policies to application roles

---

### Customer Managed Policies
**Status:** Preferred for reusable permissions

Use when:
- Permissions are shared across multiple roles
- Permissions represent a common access pattern
- Central governance and reuse is required

Benefits:
- Reusable
- Versionable
- Auditable at scale
- Easier blast-radius analysis

Typical examples:
- Standard logging access
- Read-only access to shared services
- Platform-wide integration permissions

---

### Inline Policies
**Status:** Allowed with restrictions (not forbidden)

Inline policies are **valid and often the correct choice** for **application-specific permissions**, when used deliberately.

#### Inline Policies ARE ALLOWED when ALL are true:
- The policy is attached to **a single role**
- Permissions are **specific to one application or service**
- The policy lifecycle is **tightly coupled to the role**
- The policy is **fully managed via Terraform**
- The policy grants **narrow, resource-scoped access**

Typical valid examples:
- App role → specific DynamoDB table
- App role → specific SQS queue
- App role → specific S3 bucket prefix

#### Inline Policies MUST NOT be used for:
- Cross-application shared permissions
- Platform or administrative roles
- Org-wide access patterns
- Broad service-level permissions
- Reusable access patterns

---

## Trust Policy Standards

### Trust Relationships MUST:
- Explicitly define principals
- Use exact ARNs (no wildcards)
- Document cross-account trust clearly
- Restrict trust using conditions where applicable (OIDC audience, repo, branch, environment)

### Cross-Account Trust
- Must be intentional and documented
- Must align with the platform authentication model (e.g., OIDC two-hop)
- Must not trust entire accounts unless explicitly approved

---

## Forbidden Patterns (Unless Explicitly Approved)

The following are **not allowed by default**:

- `"Action": "*"` without documented justification
- `"Resource": "*"` without documented justification
- Wildcard principals (`"AWS": "*"`)
- Admin-level permissions for application roles
- IAM policies managed outside Terraform
- Console-edited IAM resources

Any exception:
- MUST be documented
- MUST be reviewed by Platform/Security
- MUST be time-bound

---

## IAM Role Classification

### Platform Roles
Examples:
- Bootstrap roles
- CI/CD deployment roles
- Org-level integration roles

Rules:
- Inline policies are **not allowed by default**
- Prefer customer-managed policies
- Strict review required

---

### Application Roles
Examples:
- Application runtime roles
- App-specific deployment roles

Rules:
- Inline policies allowed (with restrictions)
- Permissions must be resource-scoped
- No admin or cross-service wildcard access
- Ownership must be clearly documented

---

## Review & Audit Expectations

### Review Requirements
- All IAM changes MUST be reviewed with security context (To be discussed)
- Reviews must explicitly consider:
  - Permission scope
  - Trust relationships
  - Blast radius
  - Reusability vs specificity

### Audit Expectations
- All IAM resources must be traceable via Git history
- Changes must be attributable to:
  - Pull request
  - Reviewer(s)
  - Workflow execution
- Drift detection should be enabled where possible
---