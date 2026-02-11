# Governance — Coherent Epistemic Governor (CEG)

## 1. Normative Authority

The sole normative authority for CEG v1.0 is:

spec/CEG-KERNEL-v1.0.md

No other document may override, reinterpret, or extend kernel semantics for v1.0.

README.md is informational.
All other files are informational unless explicitly versioned and declared normative.

---

## 2. Specification-First Versioning Model

CEG uses a specification-first model:

- Versions correspond to frozen specification releases.
- Tags/releases anchor immutable specification states.
- Implementation code (if added) does not redefine the spec.
- Normative changes require a new version number.

The following anchors currently exist:

- Kernel v1.0 specification freeze (canonical): v1.0-spec
- Sidecar v0.1 protocol freeze (non-normative to Kernel v1.0): ceg-sidecar-v0.1

Tags must never be moved once published.

---

## 3. Change Policy

### Patch-level edits
- Typographical corrections
- Clarifications that do not alter semantics
- Documentation improvements

These may occur without incrementing the kernel version.

### Minor version (v1.x)
- Backward-compatible clarifications to normative language
- Additional informative documents
- Reference implementations

### Major version (v2.0+)
- Any change to kernel invariants
- Any change to emission semantics
- Any change to observability gating
- Any change to regime mechanics
- Any change to non-agenticity guarantees

Major changes require a new canonical spec file and new freeze tag.

---

## 4. Non-Agenticity Preservation

The following are permanently prohibited inside the Kernel:

- Action selection
- Planning
- Optimization
- Utility/reward functions
- Policy selection
- Tool invocation
- Self-modifying logic

Any proposal introducing these requires a new version outside v1.x.

---

## 5. Informative Documents

Informative documents may include:

- PROVENANCE.md
- related-work.md
- threat-model.md
- CEG-SIDECAR-v0.1.md
- Future examples or reference implementations

These documents may elaborate but may not override the canonical kernel specification.

---

## 6. Tag Integrity

Tags/releases function as public timestamp anchors.

- Tags must never be force-moved.
- Historical releases must remain intact.
- Corrections after a freeze require a new version tag.

---

CEG v1.0 remains frozen.
All future semantic changes require a new version.