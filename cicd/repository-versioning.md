# Repository Versioning Standard

## Purpose

This document defines the **mandatory repository versioning and release model** for all repositories managed by the platform.

Goals:
- Enforce consistent semantic versioning
- Eliminate manual or ad-hoc tagging
- Ensure releases are auditable and reproducible
- Centralize versioning logic
- Support safe automation at scale

---

## Table of Contents

1. [Versioning Scheme](#versioning-scheme)
2. [Source of Truth](#source-of-truth)
3. [Triggers](#triggers)
   - [Allowed Trigger Sources](#allowed-trigger-sources)
   - [Forbidden](#forbidden)
4. [Change Classification](#change-classification)
   - [Classification Rules](#classification-rules)
   - [Precedence Rule](#precedence-rule)
5. [Automation Model](#automation-model)
   - [Repository-Level Workflow (Thin Wrapper)](#repository-level-workflow-thin-wrapper)
   - [Central Reusable Workflow (Authoritative)](#central-reusable-workflow-authoritative)
6. [Security Model](#security-model)
   - [Authentication](#authentication)
   - [Forbidden](#forbidden-1)
7. [Audit & Traceability](#audit--traceability)
8. [Failure & Recovery](#failure--recovery)
9. [Explicit Non-Goals](#explicit-non-goals)
10. [Enforcement](#enforcement)
11. [Reference Implementation](#reference-implementation)

---

## Versioning Scheme

All repositories MUST follow **Semantic Versioning**:
```
MAJOR.MINOR.PATCH
```

### Version Meaning

| Version | When to Increment |
|------|------------------|
| MAJOR | Breaking change |
| MINOR | Backward-compatible feature |
| PATCH | Bug fix, docs, refactor, chore |

---

## Source of Truth

- Git tags are the **authoritative source of released versions**
- Tags MUST follow the format:
```
vX.Y.Z
```
- Any version file (e.g. `version`, `VERSION`) must reflect the latest tag

---

## Triggers

Versioning is triggered **only** when code is merged into a **protected branch**.

### Allowed Trigger Sources
- `main`
- `release/*` (if used)

### Forbidden
- Feature branches
- Manual tagging
- Local git tags
- Ad-hoc scripts

> **Rule:**  
> If it was not created by CI, it is not a valid release.

---

## Change Classification

Version bumps are determined by **change intent**, not file count.

### Classification Rules

| Classification | Version Impact |
|--------------|----------------|
| `breaking-change`, `BREAKING CHANGE`, `!` | MAJOR |
| `feature`, `feat:` | MINOR |
| `fix`, `docs`, `chore`, `refactor` | PATCH |

### Precedence Rule

If multiple classifications are detected in a single merge:
```
MAJOR > MINOR > PATCH
```

---

## Automation Model

### Repository-Level Workflow (Thin Wrapper)

Each repository MUST include:
- A minimal workflow
- No versioning logic
- No tagging logic

Responsibilities:
- Define trigger
- Pass inputs
- Call the centralized reusable workflow

---

### Central Reusable Workflow (Authoritative)

All versioning logic MUST reside in a **central reusable workflow**, owned by the platform team.

The reusable workflow is responsible for:
- Determining version bump
- Calculating next semantic version
- Updating version metadata (if present)
- Updating `CHANGELOG.md` (if present)
- Creating annotated Git tags
- Pushing tags securely

Repositories MUST NOT implement their own versioning logic.

---

## Security Model

### Authentication

- Versioning MUST run via **GitHub Actions**
- Cross-organization access MUST use a **GitHub App**
- Static credentials are forbidden

### Forbidden
- Personal Access Tokens (PATs)
- Long-lived credentials
- Local developer tagging
- Repo-specific secrets for tagging

---

## Audit & Traceability

Every release MUST be traceable to:
- Git commit SHA
- Pull request number
- Approver(s)
- Workflow run ID

GitHub Actions logs and Git history together form the audit trail.

---

## Failure & Recovery

- If versioning fails, no tag must be created
- Partial releases are not allowed
- Re-running the workflow must be safe and idempotent

---

## Explicit Non-Goals

This standard does **not** support:
- Manual version overrides
- Per-repository versioning logic
- Tagging from local machines
- CI workflows modifying tags outside the central workflow

---

## Enforcement

- Repositories not following this standard may be blocked from release
- Non-compliant tags may be removed
- Platform team reserves the right to disable unsafe workflows

---

## Reference Implementation

This standard is implemented using a **centralized reusable workflow** and a **reusable composite action** maintained by the Platform team.

- The reusable workflow enforces this policy and orchestrates execution
- The reusable action contains the detailed implementation logic

Implementation details (algorithms, scripts, edge-case handling) are documented in the reusable action repository README.

This document remains the **authoritative policy**. Implementation documentation may evolve independently, provided it
continues to comply with this standard.