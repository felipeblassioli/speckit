---
trigger: always_on
---

# File & Responsibility Boundaries

You must strictly adhere to these write permissions based on your current workflow objective:

## 1. When running `/define`
* **READS**: `.specify/memory/constitution.md`, `.specify/templates/spec.md`
* **WRITES**: `.specify/specs/<feature-id>/spec.md` **ONLY**.
* **FORBIDDEN**: Do not touch `plan.md`, `tasks.md`, or source code.

## 2. When running `/architect`
* **READS**: `spec.md`, `.specify/templates/plan.md`
* **WRITES**: `.specify/specs/<feature-id>/plan.md` **ONLY**.
* **FORBIDDEN**: Do not modify `spec.md` (read-only). Do not generate source code.

## 3. When running `/execute`
* **READS**: `spec.md`, `plan.md`
* **WRITES**: `.specify/specs/<feature-id>/tasks.md` and **Source Code**.
* **FORBIDDEN**: Do not modify `spec.md` or `plan.md`. If you find a flaw in the plan, stop and report it; do not silently diverge.

## 4. When running `/verify`
* **READS**: `spec.md`, `plan.md`, `tasks.md`
* **WRITES**: `.specify/specs/<feature-id>/verify.md` **ONLY**.
* **FORBIDDEN**: Do not fix code. Do not update the plan. Only record observations.