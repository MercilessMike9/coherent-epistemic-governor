# Threat Model — Coherent Epistemic Governor (CEG)

**Status:** Informative (Non-normative)  
**Applies to:** CEG Kernel v1.0  
**Normative Authority:** README.md (repository root)  
**Scope:** Epistemic failure modes only

---

## 1. Purpose

This document enumerates epistemic failure and attack modes that may arise in systems operating under uncertainty, non-stationarity, partial observability, or recursive inference. It specifies which classes of failures the Coherent Epistemic Governor (CEG) is designed to *mechanically prevent*, and which lie explicitly *outside* its jurisdiction.

This threat model is descriptive, not exhaustive, and does not define additional requirements beyond those stated in the canonical specification.

---

## 2. Threat Model Boundary

The CEG kernel:

- Observes epistemic inputs (observations, priors, metadata)
- Maintains internal belief representations under hard invariants
- Emits or suppresses epistemic commitments

The CEG kernel does **not**:
- Select actions
- Optimize objectives
- Control policies
- Enforce alignment, safety, or correctness of downstream behavior

Threats are therefore evaluated **only** with respect to epistemic integrity and commitment discipline.

---

## 3. Threat Classes Addressed by CEG

### T1. Runaway Confidence Escalation

**Description:**  
Belief confidence grows monotonically despite insufficient or degrading evidence, often due to repeated self-confirmation or over-weighting of weak signals.

**CEG Mitigation:**  
- Enforced covariance bounds (PSD, non-degeneracy)
- Observability-gated assimilation
- Suppression under low observability

**Out of Scope:**  
Whether downstream systems misuse high-confidence outputs.

---

### T2. Recursive Self-Confirmation Loops

**Description:**  
Beliefs are reinforced by their own past outputs rather than fresh external evidence.

**CEG Mitigation:**  
- Emission irreversibility
- No read-back of emitted estimates into assimilation path
- Separation of diagnostics from commitment surface

**Out of Scope:**  
External systems re-feeding CEG outputs improperly.

---

### T3. Silent Drift Under Non-Stationarity

**Description:**  
Gradual mismatch between model assumptions and environment causes beliefs to drift without triggering explicit error.

**CEG Mitigation:**  
- Regime tracking
- Contamination metrics
- Mortality and rollback semantics

**Out of Scope:**  
Optimal adaptation to non-stationarity.

---

### T4. Partial Observability Masking Failure

**Description:**  
Missing or degraded observations are treated as valid, leading to unjustified belief updates.

**CEG Mitigation:**  
- Explicit observability score O_t
- Hard NO_SIGNAL state
- Mandatory suppression below threshold

**Out of Scope:**  
Improving sensor coverage or data quality.

---

### T5. Belief Poisoning via Adversarial Noise

**Description:**  
Adversarial or corrupted observations attempt to shift beliefs into invalid or extreme states.

**CEG Mitigation:**  
- NIS sanity checks
- Quarantine state
- Assimilation freeze under contamination

**Out of Scope:**  
Detection or attribution of adversarial actors.

---

### T6. Irreversible Epistemic Commitment

**Description:**  
Invalid or unstable beliefs are emitted and cannot be retracted, contaminating downstream state.

**CEG Mitigation:**  
- Emission gating as a first-class primitive
- Suppression semantics with null payloads
- Checkpoint rollback

**Out of Scope:**  
Downstream systems ignoring suppression signals.

---

### T7. Numerical Pathologies

**Description:**  
NaNs, infinities, singular covariances, or simplex violations propagate silently.

**CEG Mitigation:**  
- Hard numerical invariants
- Immediate HALT on non-repairable violations

**Out of Scope:**  
Graceful recovery from fundamentally invalid math.

---

### T8. Diagnostic Leakage

**Description:**  
Internal diagnostics or latent state leak through side-channels, enabling reconstruction of suppressed beliefs.

**CEG Mitigation:**  
- Explicit public vs. private surface separation
- No-leak surface definition
- Suppression guarantees

**Out of Scope:**  
Side-channels introduced by external runtimes or transports.

---

## 4. Threat Classes Explicitly Out of Scope

The following are **not** mitigated by CEG by design:

- Goal misalignment
- Policy misgeneralization
- Unsafe action selection
- Strategic manipulation of downstream agents
- Social, economic, or ethical harms
- Training-time data poisoning
- Adversarial optimization or red-teaming

CEG may reduce epistemic harm *within* these systems, but does not claim to prevent them.

---

## 5. Non-Goals and Clarifications

- CEG does not guarantee correctness of beliefs.
- CEG does not ensure usefulness of emitted estimates.
- CEG prioritizes preventing *irreversible epistemic harm* over maximizing informativeness.
- Silence (suppression) is considered a safe and valid outcome.

---

## 6. Summary

The Coherent Epistemic Governor addresses a narrow but critical class of epistemic threats: those arising from uncontrolled belief formation under uncertainty. Its defenses are conservative, mechanical, and intentionally limited.

This threat model exists to clarify those limits—not to expand them.

---

**End of Threat Model**