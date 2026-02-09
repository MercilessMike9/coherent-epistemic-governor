# CEG Kernel v1.0 — Draft (Fixed)

Author/Owner: Michael Middleton  
Date: February 2026  
Status: DRAFT — Non-canonical  
Authority: Informative specification (not the sole normative authority)

---

## 0) Scope (Hard)

Layer-1 only: epistemic estimation, uncertainty, regime distribution, and diagnostics.

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

This kernel governs **epistemic commitment only**, not downstream behavior.

---

## 1) Definitions

Time index:  
- t = 0,1,2,… with fixed Δt

Latent state:
- x_t ∈ ℝⁿ

Observation:
- y_t ∈ ℝᵐ (nullable)
- valid_mask ∈ {0,1}ᵐ

Regimes:
- k ∈ {1,…,M}

Regime distribution:
- μ_t ∈ Δᴹ (probability simplex)

---

### 1.1 Commitment Semantics

**Emission** at tick t is any act that returns epistemic estimates  
(x̂_t, P_t, μ_t) to any downstream consumer, storage, interface, or log.

Emission is irreversible with respect to the kernel.

**Suppressed emission** means:
- x̂_t = ∅
- P_t = ∅
- μ_t = ∅

Only minimal diagnostics may be returned while suppressed.

---

## 2) Kernel Law v1.0 (Commitment Jurisdiction)

Define predicate:

EmitAllowed(t) ⇔  
- status_t ∉ {HALT, NO_SIGNAL, QUARANTINE}  
- all required invariants hold at t

Mandatory enforcement:

¬EmitAllowed(t) ⇒ (x̂_t, P_t, μ_t) = (∅, ∅, ∅)

This law is **mechanical**, not interpretive.

---

## 3) State

Per regime k:
- x_{k,t} ∈ ℝⁿ
- P_{k,t} ∈ ℝⁿˣⁿ (PSD within tolerance)

IMM-fused estimates:
- x_t = Σ_k μ_{k,t} · x_{k,t}
- P_t = Σ_k μ_{k,t} · [ P_{k,t} + (x_{k,t} − x_t)(x_{k,t} − x_t)ᵀ ]

Diagnostics (minimum set):
- O_t ∈ [0,1] observability score
- NIS_t (if measurement present and allowed)
- C_t contamination metric (if enabled)
- status_t ∈ { OK, DEGRADED, NO_SIGNAL, RESET, QUARANTINE, HALT }

Checkpoint buffer:
- Stores last C checkpoints where:
  - status = OK
  - all invariants verified

Each checkpoint contains:
- (t, {x_k, P_k}_k, μ_t, diagnostic summaries, O_t, C_t)

---

## 4) Observability Mapping

Compute O_t using metadata only:

- If y_t missing OR sum(valid_mask) = 0 ⇒ O_t = 0
- Else  
  O_t = clamp(
    α_stale(age_sec) · α_mask(valid_mask) · α_flags(quality_flags),
    0, 1
  )

Status mapping:
- O_t < O_blind ⇒ NO_SIGNAL
- O_blind ≤ O_t < O_degraded ⇒ DEGRADED
- O_t ≥ O_degraded ⇒ OK

---

## 5) Mortality / Rollback Semantics

Counters:
- fail_count
- blind_count
- mismatch_count

RESET triggers (any):
- fail_count ≥ K_fail
- blind_count ≥ K_blind
- mismatch_count ≥ K_mismatch
- optional TTL expiry

RESET action:
1. Load most recent admissible checkpoint; if none, reinitialize from priors.
2. Apply structured uncertainty expansion.
3. Increase regime entropy:
   μ ← (1 − β)μ + β·Uniform
4. Clear diagnostic windows.
5. status ← RESET

HALT only for:
- NaN / Inf
- unrecoverable singularities
- invalid configuration
- non-repairable PSD or simplex violations

---

## 6) Commitment Surface Contract

### 6.1 Public Surface (may be returned)

Always:
- status
- minimal gate_summary
- version identifier

When EmitAllowed(t):
- x̂_t
- P_t
- μ_t
- optional diagnostics: entropy(μ), age_sec, mask summary

### 6.2 Private Telemetry Surface (never exposed)

- fail_count, blind_count, mismatch_count
- checkpoint contents
- per-regime likelihoods
- raw NIS windows
- internal exception traces

### 6.3 No-Leak Surface (never emitted)

- raw checkpoint matrices
- suppressed epistemic payloads
- timing or encoding side-channels

### 6.4 Suppression Rule

If status ∈ {HALT, NO_SIGNAL, QUARANTINE}:
- Public surface restricted to status + gate_summary
- x̂_t, P_t, μ_t MUST be null
- NIS MUST be null

---

## 7) Configuration Legitimacy (G0 Meta-Gate)

Checked at init and config reload. Failure ⇒ HALT.

G0.1 Observability thresholds:
- 0 < O_blind < O_degraded ≤ 1

G0.2 Silence reachability:
- If y missing ⇒ status = NO_SIGNAL and emission suppressed

G0.3 IMM transition matrix Π:
- Π_ij ∈ [0,1]
- rows sum to 1
- no all-zero columns

G0.4 Noise covariances:
- Q_k ⪰ 0
- R ⪰ 0 (repair or HALT)

G0.5 NIS thresholds:
- τ_nis_hi < τ_nis_hard < ∞

G0.6 Mortality reachable:
- K_fail, K_blind, K_mismatch ≥ 1

G0.7 Quarantine non-vacuous (if enabled)

---

## 8) Runtime Invariants

- G1: PSD enforcement
- G2: Simplex enforcement
- G3: Finite outputs
- G4: NIS sanity
- G5: Emission jurisdiction enforced
- G6: QUARANTINE forbids assimilation
- G7: Forbidden-ops static scan

---

## 9) Decision Table

- G0 fail ⇒ HALT, suppress
- Numerical invalidity ⇒ HALT
- O_t < O_blind ⇒ NO_SIGNAL, suppress
- C_t > C_quarantine ⇒ QUARANTINE, suppress
- Invariant violation ⇒ RESET or HALT
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
