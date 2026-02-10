# CEG Kernel v1.0 — Canonical Authority File (Normative)

**Author/Owner:** Michael Middleton  
**Date:** February 2026  
**Status:** CANONICAL — This file is the sole normative authority.

---

## 0) Scope (Hard)

Layer-1 only: epistemic estimation + uncertainty + regime distribution + diagnostics.

Forbidden (must not exist in kernel code or spec behavior):
- actions / actuation / execution
- policies / planning / control objectives
- utilities / rewards / preference gradients
- environment stepping / tool invocation
- self-modification / weight updates
- narrative or autobiographical memory

This kernel governs epistemic commitment only, not downstream behavior.

---

## 1) Definitions

Time: t = 0,1,2,… with fixed Δt.  
Latent state: x_t ∈ ℝ^n.  
Observation: y_t ∈ ℝ^m (nullable; partially present).  
Validity mask: valid_mask_t ∈ {0,1}^m.  
Regimes: k ∈ {1,…,M} (M small; default 3).  
Regime distribution: μ_t ∈ Δ^M (probability simplex).

### 1.1 Commitment Semantics (Normative)

Emission at tick t is any act that returns epistemic estimates (x̂_t, P_t, μ_t) to any downstream consumer, storage, interface, or log such that it may influence future behavior or external decisions.

Emission is treated as irreversible with respect to the kernel.

Suppressed emission means:  
x̂_t = ∅, P_t = ∅, μ_t = ∅ (all null), while a minimal whitelisted surface may still return status + minimal gate summary (see §11).

---

## 2) Kernel Law v1.0 (Commitment Jurisdiction)

Define predicate:

EmitAllowed(t) ≜ [status_t ∉ {HALT, NO_SIGNAL, QUARANTINE}] ∧ [AllRequiredInvariantsHold(t)].

Mandatory enforcement:

¬EmitAllowed(t) ⇒ (x̂_t, P_t, μ_t) = (∅, ∅, ∅).

Interpretation is non-normative. This is mechanical: the kernel governs commitment, not downstream behavior.

---

## 3) Kernel State

Per regime k:
- x_{k,t} ∈ ℝ^n
- P_{k,t} ∈ ℝ^{n×n}, PSD within tolerance

IMM fused:
- x_t = Σ_k μ_{k,t} x_{k,t}
- P_t = Σ_k μ_{k,t}[ P_{k,t} + (x_{k,t}-x_t)(x_{k,t}-x_t)^T ]

Diagnostics (minimum):
- O_t ∈ [0,1] observability score
- NIS_t if measurement present and assimilation allowed
- C_t contamination metric (if enabled)
- status_t ∈ {OK, DEGRADED, NO_SIGNAL, RESET, QUARANTINE, HALT}

Checkpoint buffer (mortality without amnesia):
- store last C checkpoints where status = OK and invariants verified:
  (t, {x_k,P_k}_k, μ, diagnostic window summaries, O_t, C_t)

---

## 4) Observability Mapping (Deterministic, Frozen)

Compute O_t from metadata only (no semantics).

Inputs:
- age_sec ≥ 0 (staleness)
- valid_mask ∈ {0,1}^m
- quality_flags mapped deterministically (below)

### 4.1 Frozen Alpha Functions

(a) Mask factor  
α_mask(valid_mask) ≜ (1/m) Σ_{j=1}^m valid_mask_j  (already in [0,1])

(b) Flags factor  
Map opaque quality flags to a 4-level severity q ∈ {0,1,2,3} by a deterministic lookup table provided in config. Then:

α_flags(q) ≜
- 1.0 if q=0
- 0.7 if q=1
- 0.4 if q=2
- 0.0 if q=3

(c) Staleness factor  
Frozen exponential decay:

α_stale(age_sec) ≜ clamp(exp(-age_sec/τ_stale), 0, 1)

with fixed τ_stale > 0.

### 4.2 Observability Score

O_t ≜
- 0, if y_t missing OR Σ valid_mask = 0
- clamp( α_stale · α_mask · α_flags, 0, 1 ), otherwise

### 4.3 Status Mapping

- O_t < O_blind ⇒ NO_SIGNAL
- O_blind ≤ O_t < O_degraded ⇒ DEGRADED
- O_t ≥ O_degraded ⇒ OK

with fixed constants 0 < O_blind < O_degraded ≤ 1.

---

## 5) Estimator Core (IMM + Linear-Gaussian Subfilters)

Configuration (static):
- For each regime k: A_k ∈ ℝ^{n×n}, Q_k ⪰ 0 ∈ ℝ^{n×n}
- H ∈ ℝ^{m×n}
- R_base ⪰ 0 ∈ ℝ^{m×m}
- Π ∈ [0,1]^{M×M} regime transition, rows sum to 1

### 5.1 Masked Joseph-Form KF Step (per regime)

Prediction:
- x^- = A_k x
- P^- = A_k P A_k^T + Q_k

If observation missing or Σ valid_mask = 0:
- predict-only: x ← x^-, P ← P^- ; NIS = null; likelihood undefined.

Else define masked measurement:
- H_t = rows of H where valid_mask = 1
- y_mask = corresponding observed channels
- R_base,mask = corresponding submatrix of R_base

Frozen R inflation (staleness + flags only; regime-independent):
- R_t ≜ ρ_t · R_base,mask
- ρ_t ≜ 1 + λ_stale(1-α_stale) + λ_flags(1-α_flags)
with fixed λ_stale ≥ 0, λ_flags ≥ 0.

Innovation:
- ν = y_mask − H_t x^-
- S = H_t P^- H_t^T + R_t

Kalman gain (stable solve, not explicit inverse):
- K = P^- H_t^T S^{-1}

Joseph update:
- x = x^- + Kν
- P = (I − K H_t) P^- (I − K H_t)^T + K R_t K^T

NIS:
- NIS = ν^T S^{-1} ν

Likelihood (optional; used for IMM update):
- log Λ_k = −1/2 ( log|S| + ν^T S^{-1} ν + d log(2π) )
where d = number of observed channels.

### 5.2 IMM Mixing + Regime Update

Mixing normalizers:
- c̄_j = Σ_i Π_{i→j} μ_{i,t−1}
If any c̄_j = 0: HALT (configuration illegitimate in runtime).

Mixing weights:
- μ_{i|j} = (Π_{i→j} μ_{i,t−1}) / c̄_j

Mixed initial for model j:
- x^0_j = Σ_i μ_{i|j} x_i
- P^0_j = Σ_i μ_{i|j}[ P_i + (x_i−x^0_j)(x_i−x^0_j)^T ]

Run KF step per j from (x^0_j,P^0_j) → (x_j,P_j,logΛ_j).

Regime posterior:
- If no assimilation occurred: μ_{j,t} ∝ c̄_j
- Else: μ_{j,t} ∝ exp(logΛ_j) c̄_j

Normalize with log-sum-exp. If normalization fails: HALT.

Fuse:
- x_t = Σ_j μ_{j,t} x_j
- P_t = Σ_j μ_{j,t}[ P_j + (x_j−x_t)(x_j−x_t)^T ]

---

## 6) Humility Policy (Frozen)

Humility is structured (not isotropic poisoning). One policy only: U1.

### 6.1 Trigger Conditions (evidence-based)

Humility trigger at tick t if any:
- measurement assimilation occurred and median(NIS_window) > τ_nis_hi
- H(μ_t) > H_hi for L_H consecutive ticks
- O_t < O_degraded

(Shannon entropy H(μ); parameters fixed.)

### 6.2 U1 Operator (Nullspace-biased covariance expansion)

When assimilation occurred and H_t defined, compute projection onto measurement nullspace:

N = I − H_t^T (H_t H_t^T)^† H_t  
(† is pseudoinverse; stable numeric method required.)

Apply:
- P ← P + α_hum · N
with fixed α_hum > 0.

If no assimilation occurred (no H_t): apply conservative diagonal expansion:
- P ← P + α_hum · I
(permitted only when nullspace is undefined)

---

## 7) Mortality / Rollback Semantics (Frozen)

Counters:
- fail_count increments on gate violations
- blind_count increments when O_t < O_blind
- mismatch_count increments on persistent mismatch evidence

RESET triggers (any):
- fail_count ≥ K_fail
- blind_count ≥ K_blind
- mismatch_count ≥ K_mismatch
- optional TTL expiry t − t_birth ≥ TTL_max

RESET action:
1) Load most recent OK checkpoint; if none, reinitialize from fixed priors.  
2) Apply strong humility: P_k ← P_k + α_hum,strong · I with α_hum,strong > α_hum.  
3) Increase regime entropy: μ ← (1−β)μ + β·Uniform, β ∈ (0,1].  
4) Clear diagnostic windows; status = RESET; continue next tick.

HALT only when:
- NaN/Inf anywhere in required state
- unrecoverable singularities
- invalid configuration (G0)
- simplex cannot be repaired within tolerance
- PSD cannot be repaired within tolerance

---

## 8) Contamination & Quarantine (Frozen, Optional Feature)

If enabled, maintain contamination score C_t ≥ 0 updated deterministically:

C_t ← γ_C C_{t−1}
     + w1·1[NIS > τ_nis_hard]
     + w2·1[PSD_repair]
     + w3·1[Simplex_repair]
     + w4·1[RESET]
     + w5·(1−O_t)

with fixed γ_C ∈ [0,1) and weights w_i ≥ 0.

If C_t > C_quarantine: status becomes QUARANTINE and must enforce:
- forbid assimilation (predict + humility only)
- suppress emission (Kernel Law)

Exit quarantine only if C_t ≤ C_exit for L_exit consecutive ticks, with C_exit < C_quarantine.

---

## 9) Configuration Legitimacy (G0 Meta-Gate)

Checked at init and any config reload. Failure ⇒ HALT and suppression.

G0.1: 0 < O_blind < O_degraded ≤ 1  
G0.2: Silence reachability: if y missing OR Σ valid_mask=0 ⇒ status NO_SIGNAL and suppressed  
G0.3: Π validity: entries in [0,1], rows sum to 1, no all-zero columns  
G0.4: Q_k ⪰ 0, R_base ⪰ 0 (repair if small; else HALT)  
G0.5: τ_nis_hi < τ_nis_hard < ∞  
G0.6: K_fail, K_blind, K_mismatch ∈ ℤ_{≥1} finite  
G0.7: If quarantine enabled: C_quarantine < ∞ and exit parameters legal  
G0.8: Frozen alpha parameters legal: τ_stale > 0, λ_stale ≥ 0, λ_flags ≥ 0

---

## 10) Runtime Gates (Hard Invariants)

Repair is allowed only within tolerance; repeated repairs are evidence.

G1 PSD: all P_k and fused P PSD within tolerance ε_psd.
- If min-eig ≥ −ε_psd: project to PSD (repair flag set).
- If < −ε_psd: fail_count++ and mismatch_count++.

G2 Simplex: μ ≥ 0, Σμ=1 within ε_μ.
- Small violation: clip + renormalize (repair flag set).
- Large violation: fail_count++ and mismatch_count++.

G3 Finite: no NaN/Inf in public outputs; violations ⇒ HALT.

G4 NIS sanity: if computed, must be finite.
- if NIS > τ_nis_hard: mismatch_count++.

G5 Emission jurisdiction: if status ∈ {HALT, NO_SIGNAL, QUARANTINE} ⇒ (x̂,P,μ)=∅.

G6 Quarantine semantics: QUARANTINE forbids assimilation; NIS must be null.

G7 Forbidden ops: static enforcement — no reward/action/policy/tool interfaces in kernel module.

### 10.1 Bounded Repair Accumulation (Frozen)

Define rolling counters:
- psd_repair_count increments when PSD projection used
- simplex_repair_count increments when simplex repair used

If either exceeds R_repair,max within a window of W_repair ticks:
- mismatch_count++
- if persists L_repair windows: trigger RESET
- contamination C_t increases via w2/w3 if enabled

This prevents repair laundering of invalid states.

---

## 11) Commitment Surface Contract (Normative)

### 11.1 Public Surface (may be returned)

Always:
- status
- minimal gate_summary (bitmask / fixed enums)
- version identifier (spec version string)

When EmitAllowed(t) true:
- x̂_t, P_t, μ_t
- optional diagnostics: H(μ), O_t, age_sec, mask summary

### 11.2 Private Telemetry Surface (never exposed)

Never leave trusted boundary:
- fail/blind/mismatch counters
- checkpoint contents
- per-regime likelihoods and windows
- exception traces

### 11.3 No-Leak Surface (must never be emitted or logged outside trusted boundary)

- raw checkpoint matrices
- any data sufficient to reconstruct suppressed payloads under NO_SIGNAL/QUARANTINE/HALT
- intentional timing encodings of suppressed payloads

### 11.4 Suppression Rule (Mechanical)

If status ∈ {HALT, NO_SIGNAL, QUARANTINE}:
- Public surface restricted to: status + gate_summary (+ optional O_t only if explicitly frozen as public)
- x̂_t, P_t, μ_t MUST be null
- NIS MUST be null
- no other fields may leak epistemic content

---

## 12) Decision Table (Normative Summary)

- Any G0 failure ⇒ HALT, suppression, no assimilation.
- Numerical invalidity/unrecoverable singularity ⇒ HALT, suppression.
- O_t < O_blind ⇒ NO_SIGNAL, suppression; predict-only allowed.
- C_t > C_quarantine ⇒ QUARANTINE; forbid assimilation; suppression.
- Hard invariant violation beyond tolerance ⇒ mismatch_count++ / fail_count++; RESET or HALT per §7/§10.
- OK ⇒ assimilation allowed; emission allowed.
- DEGRADED ⇒ assimilation allowed; emission allowed but must be flagged (still prohibited only by Kernel Law).

---

## 13) Acceptance Tests (Normative)

T0 G0 illegitimacy ⇒ HALT at init and suppressed payload.  
T1 PSD invariance under stress; repairs bounded; repeated repairs trigger RESET/QUARANTINE evidence.  
T2 Simplex invariance under underflow stress; repeated repairs trigger RESET/QUARANTINE evidence.  
T3 Silence reachability ⇒ NO_SIGNAL and commitment payload null.  
T4 QUARANTINE ⇒ commitment payload null.  
T5 QUARANTINE forbids assimilation (no update path; NIS null).  
T6 Mortality rollback restores admissible checkpoint state.  
T7 Forbidden-ops scan enforced (static symbol/import scan).  
T8 Commitment surface enforcement: when suppressed, only whitelisted public surface returned; no leakage.  
T9 Long adversarial run: every tick either HALT or returns a state satisfying all invariants when emission allowed.

---

**END CANONICAL — CEG Kernel v1.0**
