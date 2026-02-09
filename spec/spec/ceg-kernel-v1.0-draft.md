# CEG Kernel v1.0 — Draft Fixed Specification (Non-Canonical)

Author/Owner: Michael Middleton  
Date: February 2026  
Status: DRAFT — Non-canonical, reviewable, subject to revision  
Authority: Informative specification only. Not the sole normative authority.

---

## 0) Scope (Hard)

Layer-1 only: epistemic estimation, uncertainty propagation, regime distribution, and diagnostics.

Explicitly forbidden:
- actions
- policies
- utilities
- rewards
- planning
- tool invocation
- environment stepping
- self-modification
- narrative or autobiographical memory

This kernel governs **epistemic commitment only**.

---

## 1) Definitions

Time index:  
t = 0,1,2,… with fixed Δt

Latent state:  
xₜ ∈ ℝⁿ

Observation:  
yₜ ∈ ℝᵐ (nullable; partially present via valid_mask)

Regimes:  
k ∈ {1 … M}

Regime distribution:  
μₜ ∈ Δᴹ (probability simplex)

---

### 1.1 Commitment Semantics (Draft)

**Emission** at tick t is any act that returns epistemic estimates  
(x̂ₜ, Pₜ, μₜ) to any downstream consumer, storage layer, interface, or log such
that it may influence future behavior or external decisions.

Emission is treated as **irreversible** with respect to the kernel.

**Suppressed emission** means:
- x̂ₜ = ∅
- Pₜ = ∅
- μₜ = ∅

while a minimal whitelisted surface may still return status and minimal
diagnostics defined in §6.

---

## 2) Kernel Law v1.0 (Commitment Jurisdiction)

Define predicate:

EmitAllowed(t) ≜  
[ statusₜ ∉ { HALT, NO_SIGNAL, QUARANTINE } ]  
∧ [ all required invariants hold at t ]

Mandatory enforcement:

¬EmitAllowed(t) ⇒ (x̂ₜ, Pₜ, μₜ) = (∅, ∅, ∅)

Interpretation is non-normative.  
This is a **mechanical constraint**, not an ethical or behavioral one.

---

## 3) State

Per-regime state:
- xₖ,ₜ ∈ ℝⁿ
- Pₖ,ₜ ∈ ℝⁿˣⁿ (PSD within tolerance)

IMM-fused estimates:
- xₜ = Σₖ μₖ,ₜ · xₖ,ₜ
- Pₜ = Σₖ μₖ,ₜ · [ Pₖ,ₜ + (xₖ,ₜ − xₜ)(xₖ,ₜ − xₜ)ᵀ ]

Diagnostics (minimum set):
- Oₜ ∈ [0,1] observability score
- NISₜ if measurement present and assimilation allowed
- Cₜ contamination metric (if enabled)
- statusₜ ∈ { OK, DEGRADED, NO_SIGNAL, RESET, QUARANTINE, HALT }

Checkpoint buffer:
Stores last C checkpoints where:
- status = OK
- all invariants verified

Each checkpoint contains:
(t, {xₖ, Pₖ}ₖ, μ, diagnostic summaries, Oₜ, Cₜ)

---

## 4) Observability Mapping (Deterministic)

Compute Oₜ using metadata only:

If yₜ missing OR sum(valid_mask) = 0  
⇒ Oₜ = 0

Else:

Oₜ = clamp(
  α_stale(age_sec) · α_mask(valid_mask) · α_flags(quality_flags),
  0, 1
)

Status mapping:
- Oₜ < O_blind ⇒ NO_SIGNAL
- O_blind ≤ Oₜ < O_degraded ⇒ DEGRADED
- Oₜ ≥ O_degraded ⇒ OK

---

## 5) Mortality and Rollback Semantics

Counters:
- fail_count
- blind_count
- mismatch_count

RESET triggers (any):
- fail_count ≥ K_fail
- blind_count ≥ K_blind
- mismatch_count ≥ K_mismatch
- optional TTL expiration

RESET procedure:
1. Load most recent OK checkpoint, else fixed priors
2. Expand uncertainty (humility policy)
3. Increase regime entropy:
   μ ← (1 − β)μ + β·Uniform
4. Clear diagnostic windows
5. status ← RESET

HALT only for:
- NaN / Inf
- unrecoverable numerical singularities
- invalid configuration
- irreparable PSD or simplex failure

---

## 6) Commitment Surface Contract (Draft)

### 6.1 Public surface (may be returned)

Always allowed:
- status
- minimal gate_summary
- optional Oₜ (must be frozen if exposed)
- spec version identifier

When EmitAllowed(t) is true:
- x̂ₜ
- Pₜ
- μₜ
- optional diagnostics (entropy(μ), age_sec, mask summary)

---

### 6.2 Private telemetry surface (never public)

- fail_count, blind_count, mismatch_count
- checkpoint contents
- per-regime likelihoods
- raw diagnostic windows
- exception traces

---

### 6.3 No-leak surface (absolute prohibition)

Must never be emitted or logged externally:
- checkpoint state matrices
- suppressed epistemic payloads
- timing channels encoding suppressed values

---

### 6.4 Suppression rule

If status ∈ { HALT, NO_SIGNAL, QUARANTINE }:
- Public surface limited to status + gate_summary (+ optional Oₜ)
- x̂ₜ, Pₜ, μₜ MUST be null
- NIS MUST be null

---

## 7) Configuration Legitimacy (G0 Meta-Gate)

Checked at init and config reload.

Failures ⇒ HALT with suppressed emission.

G0.1: 0 < O_blind < O_degraded ≤ 1  
G0.2: Zero-information ⇒ NO_SIGNAL reachable  
G0.3: IMM Π rows sum to 1; no zero columns  
G0.4: Qₖ ⪰ 0, R ⪰ 0  
G0.5: τ_nis_hi < τ_nis_hard < ∞  
G0.6: Mortality thresholds finite ≥ 1  
G0.7: Quarantine reachable or explicitly disabled

---

## 8) Runtime Gates (Hard Invariants)

G1: PSD enforcement (repair small; else fail)  
G2: Simplex enforcement (repair small; else fail)  
G3: Finite public outputs  
G4: NIS sanity and mismatch escalation  
G5: Emission jurisdiction enforced  
G6: QUARANTINE forbids assimilation  
G7: Static forbidden-ops scan enforced

---

## 9) Decision Table (Summary)

- G0 fail ⇒ HALT, suppression
- Numerical invalidity ⇒ HALT
- Oₜ < O_blind ⇒ NO_SIGNAL, suppression
- Cₜ > C_quarantine ⇒ QUARANTINE, suppression
- Invariant breach ⇒ RESET or HALT
- OK ⇒ assimilate + emit
- DEGRADED ⇒ emit with flag or suppress (policy-frozen)

---

## 10) Acceptance Tests (Draft)

T0: G0 illegitimacy halts at init  
T1: PSD invariance under stress  
T2: Simplex invariance under underflow  
T3: Silence reachability ⇒ suppression  
T4: QUARANTINE ⇒ suppression  
T5: QUARANTINE forbids assimilation  
T6: RESET restores admissible checkpoint  
T7: Forbidden-ops scan enforced  
T8: Suppression leaks impossible via public surface

---

END DRAFT SPECIFICATION
