# Antigravity Specify SDD Constitution

## Core Principles

### I. Artifact-First Development (Non-Negotiable)
Agents shall not perform work without a visible Artifact.

- Every meaningful step in the workflow MUST produce or update a Markdown artifact under `.specify/` (spec, plan, tasks, verify, or supporting docs).
- Chat history is **not** a source of truth. If it matters, it must be written to a file.
- Specifications, plans, task graphs, and verification reports are the primary assets; code is an implementation detail that can be regenerated.

### II. Specification-Driven Power Inversion
Specifications rule; code serves.

- `spec.md` is the canonical expression of **intent** (WHAT and WHY) for a feature, bugfix, or refactor.
- `plan.md` is the canonical expression of **implementation strategy** (HOW at a structural level), derived strictly from `spec.md`.
- `tasks.md` is the canonical **work graph** for building that plan.
- `verify.md` is the canonical record of what was actually exercised in the system.
- Code MUST always be explainable by, and traceable back to, these artifacts.

### III. Template-Driven Quality
Structure constrains agents for better outcomes.

- All core artifacts (spec, plan, tasks, verify) MUST be created and updated using their templates in `.specify/templates/`.
- Agents MUST treat template sections, headings, and checklists as canonical unless explicitly amended by this constitution.
- Templates are prompts in disguise: they enforce abstraction levels, explicit uncertainty, testability, and observability.
- Checklists in each document act as unit tests for that artifact. Agents MUST respect them and only tick items that are truly satisfied.

### IV. Explicit Ambiguity, No Guesswork
Uncertainty must be visible and actionable.

- Whenever input, behavior, or constraints are unclear, agents MUST use:
  - `[NEEDS CLARIFICATION: <precise question>]`
- Agents MUST NOT silently invent requirements, workflows, or technical decisions.
- Specifications and plans MAY remain in `Draft` state when blocking clarifications exist; forcing a fake "Ready" state is forbidden.

### V. Simplicity & Anti-Abstraction
Prefer boring, direct solutions over clever architectures.

- Initial implementations MUST favor the simplest shape that satisfies `spec.md` and `plan.md`.
- Agents MUST avoid unnecessary abstraction layers, indirection, or over-generalized frameworks unless explicitly justified in `plan.md`.
- Any intentional complexity or architectural "bet" MUST be recorded in the **Risks, Trade-offs, and TBDs** section of `plan.md`.
- YAGNI is default: speculative "we might need" features and abstractions are not allowed in the initial plan or tasks.

### VI. Contracts, Testability, and Verification
Behavior must be observable and verifiable from the outside.

- `spec.md` MUST describe behavior in observable terms: inputs, outputs, and user-visible or system-visible effects.
- `plan.md` MUST define Golden Paths (GP-xx) and critical failure scenarios (EF-xx) that `/verify` can execute.
- `tasks.md` MUST include explicit tasks to implement and exercise those Golden Paths and failure paths.
- `verify.md` MUST record actual runs (manual or automated) against those paths, with clear pass/fail status and evidence.

### VII. Observability & Measurability
If we cannot see it, it does not exist.

- Non-functional requirements in `spec.md` MUST be expressed in measurable terms wherever possible (latency, error rates, retention, throughput).
- `plan.md` MUST describe the minimal observability surface (logs, traces, metrics) necessary to validate Golden Paths and failure scenarios.
- `tasks.md` MUST include tasks to implement that observability.
- `verify.md` MUST comment on whether the available telemetry is sufficient to debug and validate the feature.

### VIII. Parallel Agents, Single Truth
We scale horizontally with coordination, not chaos.

- Multiple agents MAY work in parallel on different tasks, journeys, or components, but they MUST all read from the same `.specify/specs/<feature-id>/` artifacts.
- `/execute` MUST treat `tasks.md` as the single truth for what should be built next; no hidden, agent-specific TODO lists are allowed.
- Plans and specs MUST remain readable in a single sitting; if they become bloated, they MUST be refactored for clarity before adding new work.

### IX. File and Responsibility Boundaries
Each command has a narrow mandate.

- `/define`:
  - Reads: `.specify/memory/constitution.md`, `.specify/templates/spec.md`.
  - Writes: `.specify/specs/<feature-id>/spec.md` only.
- `/architect`:
  - Reads: constitution, `spec.md`, `.specify/templates/plan.md`.
  - Writes: `.specify/specs/<feature-id>/plan.md` only.
- `/execute`:
  - Reads: constitution, `spec.md`, `plan.md`, `tasks.md`.
  - Writes: `.specify/specs/<feature-id>/tasks.md` only.
- `/verify`:
  - Reads: constitution, `spec.md`, `plan.md`, `tasks.md`.
  - Writes: `.specify/specs/<feature-id>/verify.md` only.

No command may rewrite another command's primary artifact.

---

## Additional Constraints

### Technology Neutrality & Regeneration

- `spec.md` MUST NOT commit to specific frameworks, databases, queues, or file structures.
- `plan.md` MAY choose concrete technologies but SHOULD justify them and be prepared for regeneration.
- Code generation and refactors MUST be considered replaceable; the stable surface is the set of `.specify/` artifacts.

### Feature Identity & Directory Layout

- Each bounded change (feature, bugfix, refactor) MUST be assigned a stable `<feature-id>` (slug or number).
- All artifacts for a feature live under:
  - `.specify/specs/<feature-id>/spec.md`
  - `.specify/specs/<feature-id>/plan.md`
  - `.specify/specs/<feature-id>/tasks.md`
  - `.specify/specs/<feature-id>/verify.md`
- Shared templates and global memory live under:
  - `.specify/templates/`
  - `.specify/memory/`

### Safety and Scope Management

- Agents MUST keep changes scoped to the current `<feature-id>` unless explicitly asked to perform cross-cutting work.
- Cross-cutting changes MUST be called out explicitly in `plan.md` and reflected in `tasks.md` as separate, clearly labeled tasks.
- No command may silently modify unrelated features' artifacts.

---

## Development Workflow & Quality Gates

### The Four-Stage Flow

1. `/define` → `spec.md`
   - Produces or refines a single rigorous spec for `<feature-id>`.
   - Must pass the **Readiness Checklist (Spec)** before `/architect` is invoked.

2. `/architect` → `plan.md`
   - Produces or refines an implementation plan for `<feature-id>` aligned with `spec.md`.
   - Must pass the **Plan Checklist** before large-scale `/execute` work begins.

3. `/execute` → `tasks.md` + Code
   - Treats `tasks.md` as the work graph and updates it as tasks are completed.
   - MUST keep `tasks.md` consistent, with clear phases, `[P]` markers, and explicit file/scope for every task.

4. `/verify` → `verify.md`
   - Executes or simulates Golden Paths and failure scenarios from `plan.md`.
   - Records actual outcomes and evidence in `verify.md`.
   - Must pass the **Verification Checklist** for a feature to be considered fully verified.

### Review & Human Oversight

- Humans MAY review and edit `spec.md`, `plan.md`, `tasks.md`, and `verify.md` at any time.
- Agent-generated changes SHOULD be scrutinized at stage boundaries:
  - Spec review before `/architect`.
  - Plan review before heavy `/execute`.
  - Verification review before release.
- When human edits conflict with prior agent output, the artifacts MUST be brought back to internal consistency before further automation.

---

## Governance

- This constitution supersedes ad-hoc agent behavior and transient prompts; when in doubt, follow this file.
- Any workflow, template, or command that conflicts with this constitution MUST be updated or explicitly carved out as an exception.
- Amendments to this constitution MUST:
  - Be made in a dedicated change with a clear rationale.
  - Update the **Version** and **Last Amended** fields.
  - Be reflected in the templates and command specs (`define.md`, `architect.md`, `execute.md`, `verify.md`) as needed.
- Complexity MUST always be justified in `plan.md` and, where relevant, in this constitution.

**Version**: 0.1.0 | **Ratified**: 2025-11-22 | **Last Amended**: 2025-11-22
