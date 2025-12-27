# Platform Standards Repository

This repository defines the **authoritative engineering standards and governance contracts** for Terraform, CI/CD, and IAM usage across the organization.

It serves as the **single source of truth** for how infrastructure and automation
must be designed, reviewed, versioned, and operated.

---

## ## Table of Contents

1. [Purpose](#purpose)
2. [Scope](#scope)
3. [Repository Layout](#repository-layout)
4. [How to Use This Repository](#how-to-use-this-repository)
5. [What This Repository Is NOT](#what-this-repository-is-not)
6. [Enforcement Model](#enforcement-model)
7. [Ownership & Governance](#ownership--governance)

---

## Purpose

The goals of this repository are to:

- Provide a centralized, authoritative reference for platform standards
- Provide a single source of truth for platform governance
- Ensure consistency, security, and auditability across all repositories
- Prevent platform drift and duplicated governance logic
- Enable safe reuse and automation at scale

## Scope
These standards apply to **all repositories and workflows** that interact with the platform, including:

- Terraform bootstrap, blueprint, module, and application repositories
- GitHub Actions workflows, reusable workflows, and reusable actions
- IAM roles, policies, and trust relationships
- Repository versioning and release processes
- CI/CD authentication and permission models

Any repository that provisions infrastructure or executes automation is expected to comply with the standards defined here.

---

## Repository Layout

```
platform-standards/
├── README.md
├── VERSION
│
├── architecture/
│   └── repository-interaction-model.md
│
├── terraform/
│   ├── terraform-standards.md
│   ├── terraform-module-checklist.md
│   └── iam-standards.md
│
├── cicd/
│   ├── repository-versioning.md
│   ├── github-actions-standards.md
│   └── reusable-workflow-guidelines.md
└── ownership/
    └── platform-ownership-model.md
```

---

## How to Use This Repository

- **Engineers** should consult these documents when designing or reviewing changes
- **Reviewers** should use them as the baseline for approval decisions
- **CI/CD pipelines** should enforce these standards where possible

When standards appear to conflict, the **more restrictive rule applies**.

---

## What This Repository Is NOT

This repository intentionally does **not** contain:

- Terraform modules or infrastructure code
- GitHub Actions workflows or reusable actions
- Scripts, binaries, or execution logic
- Environment-specific configuration

This repository defines **rules and contracts**.  
Other repositories are responsible for **implementing and enforcing** them.

---

## Enforcement Model

Compliance with these standards is enforced through:

- Code review and pull request approval
- CI/CD pipelines and reusable workflows
- Repository templates and PR checklists
- Automated scanning and validation
- Periodic platform and security audits

In case of conflict or ambiguity, **documents in this repository take precedence** over repository-level conventions or local practices.

---

## Ownership & Governance

This repository is owned and maintained by the **IMS Tooling Team**.

- Ownership and decision authority are defined in:
  - `ownership/platform-ownership-model.md`
- Standards are reviewed:
  - Quarterly
  - Or upon major platform, security, or tooling changes