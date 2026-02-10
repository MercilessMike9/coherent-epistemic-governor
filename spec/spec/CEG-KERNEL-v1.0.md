CEG Kernel v1.0 — Canonical Authority File (Normative)

Author/Owner: Michael Middleton  
Date: February 2026  
Status: CANONICAL — This file is the sole normative authority.

⸻

0) Scope (Hard)

Layer-1 only: epistemic estimation + uncertainty + regime distribution + diagnostics.

Forbidden (must not exist in kernel code or spec behavior):
• actions / actuation / execution  
• policies / planning / control objectives  
• utilities / rewards / preference gradients  
• environment stepping / tool invocation  
• self-modification / weight updates  
• narrative or autobiographical memory  

This kernel governs epistemic commitment only, not downstream behavior.

⸻

1) Definitions

Time: t = 0,1,2,… with fixed Δt.  
Latent state: x_t ∈ ℝⁿ.  
Observation: y_t ∈ ℝᵐ (nullable; partially present).  
Validity mask: valid_mask_t ∈ {0,1}ᵐ.  
Regimes: k ∈ {1,…,M} (M small; default 3).  
Regime distribution: μ_t ∈ Δᴹ (probability simplex).

1.1 Commitment Semantics (Normative)

Emission at tick t is any act that returns epistemic estimates (x̂_t, P_t, μ_t) to any downstream consumer, storage, interface, or log such that it may influence future behavior or external decisions.

Emission is treated as irreversible with respect to the kernel.

Suppressed emission means:
x̂_t = ∅,  P_t = ∅,  μ_t = ∅

(all null), while a minimal whitelisted surface may still return status + minimal gate summary (see §11).

⸻

2) Kernel Law v1.0 (Commitment Jurisdiction)

Define predicate:

EmitAllowed(t) ≜
[ status_t ∉ {HALT, NO_SIGNAL, QUARANTINE} ]
∧
[ AllRequiredInvariantsHold(t) ].

Mandatory enforcement:

¬EmitAllowed(t) ⇒ (x̂_t, P_t, μ_t) = (∅, ∅, ∅).

Interpretation is non-normative. This is mechanical: the kernel governs commitment, not downstream behavior.

⸻

3) Kernel State

Per regime k:
• x_{k,t} ∈ ℝⁿ  
• P_{k,t} ∈ ℝⁿˣⁿ, PSD within tolerance  

IMM fused:
x_t = Σₖ μ_{k,t} x_{k,t}  
P_t = Σₖ μ_{k,t}[ P_{k,t} + (x_{k,t}−x_t)(x_{k,t}−x_t)ᵀ ]

Diagnostics (minimum):
• O_t ∈ [0,1] observability score  
• NIS_t if measurement present and assimilation allowed  
• C_t contamination metric (if enabled)  
• status_t ∈ {OK, DEGRADED, NO_SIGNAL, RESET, QUARANTINE, HALT}

Checkpoint buffer (mortality without amnesia):
Store last C checkpoints where status = OK and invariants verified:
(t, {x_k,P_k}_k, μ, diagnostic window summaries, O_t, C_t)

⸻

4) Observability Mapping (Deterministic, Frozen)

Compute O_t from metadata only.

Inputs:
• age_sec ≥ 0  
• valid_mask ∈ {0,1}ᵐ  
• quality_flags mapped deterministically

4.1 Frozen Alpha Functions

Mask factor:
α_mask(valid_mask) ≜ (1/m) Σⱼ valid_maskⱼ

Flags factor:
Map quality flags to severity q ∈ {0,1,2,3} via fixed table.
α_flags(q) ≜ {1.0, 0.7, 0.4, 0.0} for q = {0,1,2,3} respectively.

Staleness factor:
α_stale(age_sec) ≜ clamp(exp(−age_sec / τ_stale), 0, 1)
with fixed τ_stale > 0.

4.2 Observability Score

If y_t missing OR Σ valid_mask = 0:
O_t = 0

Else:
O_t = clamp(α_stale · α_mask · α_flags, 0, 1)

4.3 Status Mapping

O_t < O_blind            ⇒ NO_SIGNAL  
O_blind ≤ O_t < O_degraded ⇒ DEGRADED  
O_t ≥ O_degraded          ⇒ OK  

with fixed 0 < O_blind < O_degraded ≤ 1.

⸻

5) Estimator Core (IMM + Linear-Gaussian Subfilters)

Static configuration:
• A_k ∈ ℝⁿˣⁿ  
• Q_k ⪰ 0  
• H ∈ ℝᵐˣⁿ  
• R_base ⪰ 0  
• Π ∈ [0,1]ᴹˣᴹ, rows sum to 1  

5.1 Masked Joseph-Form KF Step

Prediction:
x⁻ = A_k x  
P⁻ = A_k P A_kᵀ + Q_k

If no valid measurement:
predict-only, no NIS.

Else:
Define masked H_t, y_t, R_t.

Frozen R inflation:
R_t = ρ_t · R_base_mask
ρ_t = 1 + λ_stale(1−α_stale) + λ_flags(1−α_flags)

Innovation:
ν = y_t − H_t x⁻  
S = H_t P⁻ H_tᵀ + R_t  

Gain:
K = P⁻ H_tᵀ S⁻¹

Joseph update:
x = x⁻ + Kν  
P = (I−KH_t)P⁻(I−KH_t)ᵀ + K R_t Kᵀ

NIS = νᵀ S⁻¹ ν

Likelihood (optional):
log Λ_k = −½(log|S| + NIS + d log 2π)

5.2 IMM Mixing + Update

Compute mixing weights, mixed initial states, run filters, update μ via log-sum-exp.
Failure to normalize ⇒ HALT.

⸻

6) Humility Policy (Frozen)

Single policy: U1.

Triggers:
• median(NIS_window) > τ_nis_hi  
• entropy(μ_t) > H_hi for L_H ticks  
• O_t < O_degraded  

U1 operator:
If H_t defined:
N = I − H_tᵀ(H_tH_tᵀ)†H_t  
P ← P + α_hum · N

Else:
P ← P + α_hum · I

⸻

7) Mortality / Rollback Semantics

Counters:
fail_count, blind_count, mismatch_count

RESET triggers:
fail_count ≥ K_fail  
blind_count ≥ K_blind  
mismatch_count ≥ K_mismatch  
optional TTL

RESET action:
1. Restore last OK checkpoint or priors  
2. Apply strong humility: P_k ← P_k + α_hum,strong I  
3. μ ← (1−β)μ + β·Uniform  
4. Clear windows; status = RESET

HALT only for unrecoverable numerical or configuration failure.

⸻

8) Contamination & Quarantine (Optional)

C_t update:
C_t ← γ_C C_{t−1}
+ w₁·[NIS>τ_nis_hard]
+ w₂·[PSD_repair]
+ w₃·[Simplex_repair]
+ w₄·[RESET]
+ w₅·(1−O_t)

If C_t > C_quarantine:
status = QUARANTINE
• forbid assimilation
• suppress emission

Exit after sustained recovery.

⸻

9) Configuration Legitimacy (G0)

Checked at init/reload. Any failure ⇒ HALT.

Includes:
• observability thresholds valid
• silence reachability
• Π valid, no dead regimes
• PSD noise
• NIS thresholds finite
• mortality reachable
• quarantine parameters valid
• alpha parameters legal

⸻

10) Runtime Gates (Hard Invariants)

G1 PSD invariance  
G2 Simplex invariance  
G3 Finite outputs  
G4 NIS sanity  
G5 Emission jurisdiction  
G6 Quarantine semantics  
G7 Forbidden ops static exclusion  

10.1 Bounded Repair Accumulation

Repeated PSD/simplex repairs beyond limits escalate mismatch_count and trigger RESET / QUARANTINE.

⸻

11) Commitment Surface Contract

Public (always):
• status
• gate_summary
• version id

Public (if EmitAllowed):
• x̂_t, P_t, μ_t
• optional diagnostics

Private telemetry never exposed.  
No-leak surface strictly enforced.

⸻

12) Decision Table (Normative)

• G0 failure ⇒ HALT  
• Numerical invalidity ⇒ HALT  
• O_t < O_blind ⇒ NO_SIGNAL  
• C_t > C_quarantine ⇒ QUARANTINE  
• Invariant breach ⇒ RESET or HALT  
• OK ⇒ assimilate + emit  
• DEGRADED ⇒ assimilate + emit (flagged)

⸻

13) Acceptance Tests (Normative)

T0–T9 as defined: invariants, silence reachability, quarantine, rollback, forbidden ops, long adversarial run safety.

⸻

END CANONICAL — CEG Kernel v1.0
