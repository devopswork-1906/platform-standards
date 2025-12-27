# Reusable Workflow & Action Guidelines

## Purpose

This document defines **mandatory design and usage guidelines** for:
- Reusable GitHub Actions workflows
- Reusable GitHub Actions
- Repository-level workflows that consume them

The goal is to:
- Enforce clear separation of concerns
- Prevent duplication and policy drift
- Enable safe reuse at scale
- Maintain strong security and auditability guarantees

---

## Table of Contents

1. [Design Principles](#design-principles)
2. [Reusable Workflows](#reusable-workflows)
3. [Reusable Actions](#reusable-actions)
4. [Repository-Level Workflows](#repository-level-workflows)
5. [Invocation Rules (Summary)](#invocation-rules-summary)
6. [Versioning & Pinning](#versioning--pinning)

---

## Design Principles

All CI/CD components MUST follow these principles:

- **Separation of Concerns** – orchestration, policy, and execution are distinct
- **Least Privilege** – permissions granted only where required
- **Centralized Control** – policies are enforced in one place
- **Reusability** – logic should be shared, not copied
- **Auditability** – every action must be traceable

---

## Reusable Workflows

### Purpose

Reusable workflows act as the **control plane** for CI/CD.

They are responsible for:
- Orchestration
- Policy enforcement
- Approval gating
- Permission control

Reusable workflows define **what is allowed to happen**.

---

### Mandatory Characteristics

Reusable workflows MUST:

- Be centrally owned and versioned by the IMS Tooling team
- Be invoked by repository-level workflows
- Explicitly define required permissions
- Enforce approval gates using GitHub Environments
- Validate inputs received from repositories
- Call reusable actions to perform execution

---

### Allowed Responsibilities

Reusable workflows MAY:

- Decide whether an operation is permitted
- Enforce platform standards
- Control execution order
- Gate destructive operations
- Perform versioning and tagging (if applicable)

---

### Forbidden Responsibilities

Reusable workflows MUST NOT:

- Contain repository-specific logic
- Hardcode secrets or credentials
- Bypass approval or policy checks
- Assume environment-specific behavior
- Perform low-level execution logic directly

---

## Reusable Actions

### Purpose

Reusable actions provide **atomic, reusable execution units**.

They are responsible for **how** a task is executed, not **whether** it should be executed.

---

### Mandatory Characteristics

Reusable actions MUST:

- Perform a single, well-defined task
- Be stateless
- Be policy-agnostic
- Declare explicit inputs and outputs
- Assume no environment context
- Be reusable across repositories and workflows

---

### Allowed Responsibilities

Reusable actions MAY:

- Execute commands
- Perform calculations
- Interact with external systems (when credentials are provided)
- Produce outputs for downstream steps

---

### Forbidden Responsibilities

Reusable actions MUST NOT:

- Enforce policy decisions
- Make approval or gating decisions
- Modify repository state (tags, commits) without workflow approval
- Retrieve secrets unless explicitly passed
- Infer environment, branch, or deployment context

---

## Repository-Level Workflows

### Purpose

Repository-level workflows act as **entry points** for CI/CD.

They define **when** workflows run and **which reusable workflow** is invoked.

---

### Mandatory Characteristics

Repository workflows MUST:

- Be minimal and declarative
- Define triggers (push, PR, workflow_dispatch)
- Pass inputs to reusable workflows
- Select target environments

---

### Forbidden Responsibilities

Repository workflows MUST NOT:

- Contain business or policy logic
- Perform versioning or tagging
- Deploy or destroy infrastructure directly
- Implement approval logic
- Duplicate logic already present in reusable workflows

---

## Invocation Rules (Summary)

| Component | May Call | May Not Call |
|--------|--------|-------------|
| Repo Workflow | Reusable Workflow, Utility Actions | Policy Actions |
| Reusable Workflow | Reusable Actions | Repo-specific logic |
| Reusable Action | — | Policy / approvals |

---

## Versioning & Pinning

- Reusable workflows MUST be versioned
- Reusable actions MUST be version-pinned
- Floating references are forbidden in production paths
