# Terraform Module Review Checklist

This checklist MUST be completed for all Terraform module pull requests.

The reviewer is responsible for ensuring **compliance with platform standards**. Unchecked items must be explicitly justified.

---
## Table of Content

1. [Security](#security)
2. [Module Design](#module-design)
3. [State & Backend](#state--backend)
4. [Lifecycle & Safety](#lifecycle--safety)
5. [CI & Versioning](#ci--versioning)
6. [Tagging](#tagging)
7. [Documentation](#documentation)

---

## Security

- [ ] No wildcard IAM permissions (`"*"` actions or resources) without documented justification
- [ ] IAM policies comply with `iam-standards.md`
- [ ] Inline IAM policies used only for app-specific, tightly scoped access
- [ ] Sensitive outputs explicitly marked with `sensitive = true`
- [ ] No secrets, credentials, or tokens committed
- [ ] `prevent_destroy` applied to critical resources (state, logs, security controls)
- [ ] No security-sensitive defaults are weakened

---

## Module Design

- [ ] Module follows **single responsibility principle**
- [ ] Module is **environment-agnostic**
- [ ] No hardcoded:
  - Account IDs
  - Regions
  - Environment names
- [ ] Providers are **not embedded** in the module
- [ ] Inputs are strongly typed (`string`, `number`, `bool`, `object`, `map`)
- [ ] Critical inputs have `validation` blocks
- [ ] Reasonable defaults are provided where applicable
- [ ] Outputs are minimal and necessary only

---

## State & Backend

- [ ] No backend configuration included in the module
- [ ] No `terraform { backend {} }` blocks in module code
- [ ] Module assumes backend is injected externally
- [ ] Module does not reference shared or global state
- [ ] State is isolated per workload/environment

---

## Lifecycle & Safety

- [ ] `create_before_destroy` used where supported and safe
- [ ] Explicit `depends_on` declared where implicit dependencies are insufficient
- [ ] Destructive changes are intentional and documented
- [ ] No hidden or implicit resource replacement risks

---

## CI & Versioning

- [ ] Centralized repository versioning workflow is used
- [ ] No manual version bumps
- [ ] No local or ad-hoc git tags created
- [ ] Commit messages follow agreed classification (`feat`, `fix`, `chore`, etc.)
- [ ] CI pipeline runs:
  - [ ] `terraform fmt`
  - [ ] `terraform validate`

## Tagging

- [ ] Standardized tags applied to all resources
- [ ] Tags merged centrally via locals
- [ ] CI context tags included where required:
  - Repository
  - Workflow
  - Actor
  - Commit / PR reference

---

## Documentation

- [ ] `README.md` clearly describes module purpose
- [ ] Usage examples are present and accurate
- [ ] Inputs table is complete and up to date
- [ ] Outputs table is complete and up to date
- [ ] Assumptions and limitations are documented
- [ ] Examples successfully run `terraform init` and `terraform plan`