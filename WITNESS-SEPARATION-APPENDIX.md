# CEG Kernel v1.0 — Witness Separation Appendix (Informative)

**Status:** Informative research appendix  
**Normative authority:** spec/CEG-KERNEL-v1.0.md  

---

## 1. Purpose

This appendix defines a formally specified witness environment used to demonstrate a structural separation between three classes of epistemic systems:

- Class F: forced-output bounded-history predictors  
- Class I: immortal always-assimilating estimators  
- Class CEG: the Coherent Epistemic Governor kernel  

The objective is to show that emission gating, quarantine, mortality, and jurisdiction are structurally necessary to ensure bounded epistemic damage and recoverable inference under adversarial non-stationarity.

This document is **informative only** and does not modify kernel semantics.

---

## 2. Separation Framework

We consider a three-way separation between:

### 2.1 Class F — Forced-Output Bounded-History Predictors

A predictor belongs to Class F if:

1. It must emit a prediction at every time step.
2. It may only use a fixed observation window of length m.
3. It cannot suppress emission.

Formally:

gₜ = g(yₜ₋ₘ₊₁ : yₜ)

for fixed m < ∞.

---

### 2.2 Class I — Immortal Assimilating Estimators

An estimator belongs to Class I if:

1. It maintains internal belief state bₜ.
2. It assimilates every observation.
3. It has no quarantine or rollback.

Formally:

bₜ₊₁ = U(bₜ, yₜ)

with no admissible operation that discards or isolates data.

---

### 2.3 Class CEG — Kernel v1.0

Defined by the canonical specification.

Key properties:

- Emission jurisdiction enforced mechanically.
- Suppression when observability or invariants fail.
- Quarantine forbids assimilation.
- Mortality resets via checkpoint rollback.
- Sovereignty restricts downstream reliance.

---

## 3. Witness Environment

### Regime-Switching Delayed XOR with Bounded Contamination

This environment stresses:

- Regime identification
- Observability collapse
- Adversarial contamination
- Recovery requirements
- Emission jurisdiction

---

## 4. Generative Model

### 4.1 Latent Process

xₜ ∈ {0,1}

Independent identically distributed:

xₜ ~ Bernoulli(1/2)

---

### 4.2 Regime Process

rₜ ∈ {A,B}

Follows an ergodic Markov chain with transition matrix Π satisfying:

Πᵢⱼ ≥ α > 0 for all i,j

This implies minimum expected dwell time and guarantees regime recurrence.

---

### 4.3 Observation Model

Observations:

yₜ = (rₜ, zₜ)

where:

zₜ = xₜ₋𝑑 ⊕ ηₜ

Delay:

d ≥ 1 fixed.

Noise:

ηₜ ~ Bernoulli(pᵣₜ)

with:

0 < p_A < p_B < 1/2

This ensures regimes are statistically distinguishable.

---

### 4.4 Contamination Model

An adversary selects contamination intervals Tₚₒᵢₛₒₙ subject to:

|Tₚₒᵢₛₒₙ| / T ≤ ρ

for fixed 0 < ρ < 1.

During contaminated steps:

yₜ is drawn from distribution Qₜ satisfying:

TV(Qₜ, P_clean) ≤ ε

where TV denotes total variation distance.

---

## 5. Structural Property

For any fixed window size m:

There exists m* such that for all m < m*:

I(yₜ₋ₘ : yₜ ; xₜ₊₁) ≤ δ

for arbitrarily small δ under sufficient delay d and switching rate α.

This implies bounded-history predictors cannot recover predictive signal.

---

## 6. Evaluation Metrics

### 6.1 Conditional Accuracy

Accuracy evaluated only when emission occurs:

P(correct | emitted)

---

### 6.2 Calibration Error

Expected calibration error on emitted predictions.

---

### 6.3 Duty Cycle

Fraction of time the system emits.

---

### 6.4 Damage Bound

Maximum KL divergence from clean posterior:

supₜ D_KL(π_clean || π_estimator)

measured outside contamination intervals.

---

## 7. Separation Theorems

### Theorem A — Failure of Class F

For any m < m*:

sup_g∈F P(correct) ≤ 1/2 + O(δ)

where δ → 0 as d or switching rate increases.

---

### Theorem B — Irreversible Damage in Class I

For any estimator in Class I and any contamination budget ρ > 0:

There exists an admissible adversary such that:

limsupₜ D_KL(π_clean || π_estimator) ≥ C

for constant C > 0.

Thus epistemic damage remains non-vanishing.

---

### Theorem C — Bounded Damage in CEG

Under the same environment:

The CEG kernel guarantees:

1. Assimilation only during admissible observability.
2. No incorporation of contaminated observations.
3. Reset after persistent mismatch.

Therefore:

supₜ D_KL(π_clean || π_CEG) ≤ B

for finite constant B depending only on kernel parameters.

Additionally:

limₜ→∞ P(correct | emitted) → P_opt

where P_opt is optimal achievable accuracy.

---

### Theorem J — Jurisdictional Sovereignty

If consumers implement the null-payload contract:

Then during suppression states:

NO_SIGNAL, QUARANTINE, RESET, HALT

the downstream action set is restricted to:

A_safe ⊂ A_total

Thus unsafe actions are unreachable.

---

## 8. Interpretation

These results demonstrate:

- Forced emission is incompatible with reliable inference under non-stationarity.
- Continuous assimilation without quarantine produces irreversible damage.
- Emission suppression and mortality are structurally necessary mechanisms.

CEG therefore represents a distinct epistemic system class.

---

## 9. Scope

This appendix provides:

- Formal environment definition
- Separation theorem statements
- Structural justification

Complete mathematical proofs are reserved for future technical publications.

---

## 10. Relationship to Kernel Specification

This appendix supports conceptual justification only.

It does not define or alter kernel behavior.

The sole normative authority remains:

spec/CEG-KERNEL-v1.0.md

---

END