# Coherent Epistemic Governor (CEG)

**Status:** Stable Reference Specification  
**Version:** v1.0 (Frozen)  
**License:** Apache License 2.0  

## Overview

The **Coherent Epistemic Governor (CEG)** is a **non-agentic**, **invariant-enforcing** epistemic kernel designed to bound belief formation under uncertainty, non-stationarity, and recursive self-reference.

CEG operates strictly at **Layer-1** (epistemic estimation only). It does **not** act, plan, optimize, decide, pursue objectives, or select policies.

Its sole function is to govern **epistemic commitment** by enforcing hard structural invariants on belief updates, uncertainty propagation, regime tracking, and emission. When those invariants cannot be guaranteed, the kernel fails closed via suppression, rollback, quarantine, or halt.

## What problem this addresses

Long-horizon and recursive inference systems (including self-referential or self-improving systems) are vulnerable to epistemic pathologies such as:

- Runaway confidence escalation
- Recursive self-confirmation loops
- Silent belief drift under non-stationarity
- Irreversible epistemic damage from invalid updates

CEG treats these as **structural hazards**, not behavioral or moral failures, and addresses them mechanically at the epistemic boundary.

## What this is

- A sovereign epistemic constraint automaton
- A fail-closed governor over belief emission
- A regime-aware estimator with bounded recoverability
- A non-semantic, domain-neutral kernel
- A Layer-1 primitive intended to sit beneath any agentic, planning, or policy layer

## What this is not

CEG is not:

- An agent
- A planner or optimizer
- A reinforcement learning system
- A policy selector or controller
- A value alignment mechanism
- A safety overlay or moderation layer
- A cognitive or philosophical model

It introduces no goals, utilities, rewards, preferences, policies, or action-selection logic.

## Core guarantees (v1.0)

CEG v1.0 enforces:

- **Emission jurisdiction:** Epistemic estimates may only be emitted when all invariants hold.
- **Fail-closed semantics:** Below observability thresholds or during quarantine, emission is suppressed.
- **Outcome isolation:** Downstream outcomes cannot feed back into epistemic state.
- **Bounded belief states:** Covariances remain PSD; regime weights remain on the simplex.
- **Recoverability:** Checkpoint mortality enables rollback without amnesia.
- **Non-agenticity:** The kernel cannot act or optimize by construction.

## Canonical authority (normative)

The sole normative authority for CEG v1.0 is:

- `spec/CEG-KERNEL-v1.0.md`

All other files (including `README.md`, `CHANGELOG.md`, `GOVERNANCE.md`, `spec/PROVENANCE.md`, and supporting materials) are informative unless explicitly stated otherwise.

## Releases / frozen anchors

This repository uses a specification-first versioning model. Tags/releases anchor immutable specification states:

- **Kernel v1.0 specification freeze (canonical):** `v1.0-spec`
- **Sidecar v0.1 protocol freeze (non-normative to Kernel v1.0):** `ceg-sidecar-v0.1`

Do not move tags. New work must land under new versions/tags.

## Provenance

This specification was iteratively developed in private, refined through adversarial self-review and control-theoretic analysis, and publicly frozen as v1.0 in February 2026.

The Git tags/releases and commit history constitute the authoritative public timestamp.

## Repository structure

- `README.md`
- `LICENSE`
- `CHANGELOG.md`
- `GOVERNANCE.md`
- `spec/CEG-KERNEL-v1.0.md`
- `spec/PROVENANCE.md`
- `spec/related-work.md`
- `spec/threat-model.md`
- `spec/CEG-SIDECAR-v0.1.md`

## Usage & implementation

This repository currently provides a **reference specification**, not a production implementation.

The specification is intended to be implemented independently, cited, audited, and used as a bounded epistemic layer beneath agentic systems.

A minimal reference implementation may be added in future versions, but v1.0 is specification-complete.

## License

This project is licensed under the **Apache License 2.0**.

## Status

CEG v1.0 is frozen. All future changes must occur under new versions according to `GOVERNANCE.md`.