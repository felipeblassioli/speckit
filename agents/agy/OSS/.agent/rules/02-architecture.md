---
trigger: model_decision
description: Apply this rule whenever the user asks for code implementation, technical architecture, test generation, debugging, or refactoring. It governs engineering standards like TDD, Library-First design, and Observability requirements.
---

# Architectural Constraints

## 1. Library-First & CLI Mandate
* Every feature must be designed as a modular library first.
* Functionality must be exposed via a text-based Interface (CLI/API) that accepts text/JSON and outputs text/JSON.
* **Why**: This ensures Observability (Article VII).

## 2. Test-Driven Development (TDD)
* **Rule**: No implementation code shall be written before tests.
* **Sequence**:
    1.  Write the Interface/Contract (Mock/Type definition).
    2.  Write the Test (Red).
    3.  Write the Implementation (Green).

## 3. Anti-Abstraction
* Use framework features directly. Do not create "Wrapper Classes" or "Helper Utils" unless they reduce code volume by >50%.
* If `plan.md` does not explicitly ask for a Design Pattern (e.g., Factory, Strategy), do not introduce it.

## 4. Observability
* All logical branches must emit logs or metrics.
* If you cannot verify a behavior via stdout, logs, or a file write, the feature is not complete.