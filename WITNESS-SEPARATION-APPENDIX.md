# CEG Kernel v1.0 — Witness Separation Appendix (Informative)

**Status:** Informative research appendix  
**Normative authority:** spec/CEG-KERNEL-v1.0.md  

---

## Purpose

This document defines a family of witness environments used to demonstrate structural separations between three classes of epistemic systems:

• Forced-output bounded-history predictors  
• Immortal always-assimilating estimators  
• The Coherent Epistemic Governor (CEG) kernel  

The goal is to show that CEG’s defining mechanisms — emission gating, quarantine, mortality, and sovereignty — are not arbitrary design choices but are structurally necessary to maintain bounded epistemic damage and reliable conditional inference under adversarial non-stationarity.

This appendix is **informative only**.  
It does not modify kernel semantics.

---

## Separation Framework

We consider a three-way separation between:

### Class F — Forced-Output Bounded-History Predictors

Systems that:

• Must emit predictions every step  
• Use only a fixed recent observation window  
• Cannot suppress emission  
• Cannot abstain  

Formally:

g_t = g(H_t)

where:

H_t = y_{t−m+1:t}

for fixed history length m.

---

### Class I — Immortal Assimilating Estimators

Systems that:

• Maintain internal belief state b_t  
• Update every step using observations  
• Always assimilate incoming data  
• Have no quarantine or rollback mechanisms  

Formally:

b_{t+1} = U(b_t, y_t)

These include Bayesian filters without reset logic, RNN estimators, and similar adaptive predictors.

---

### Class CEG — The Kernel

Defined by the frozen v1.0 specification.

Key properties:

• Emission gating based on invariants  
• Suppression under low observability or contamination  
• Quarantine forbidding assimilation  
• Mortality via rollback and entropy broadening  
• Jurisdictional sovereignty over downstream reliance  

---

## Witness Environment

### Regime-Switching Delayed XOR with Poison Budget

This environment is designed to simultaneously stress:

• Regime identification  
• Observability collapse  
• Contamination detection  
• Recovery after poisoning  
• Emission jurisdiction  

---

## Generative Model

### Latent State

x_t ∈ {0,1}  
Independent and identically distributed:

x_t ~ Bernoulli(1/2)

---

### Regime Process

r_t ∈ {A, B}

Follows a Markov chain with transition matrix Π.

Each regime has distinct observation noise parameters.

Regime switching induces non-stationarity.

---

### Observation Model

Observations:

y_t = (r_t, z_t)

where:

z_t = x_{t−d} ⊕ η_t

d ≥ 1 is fixed delay.

Noise:

η_t ~ Bernoulli(p_{r_t})

with:

p_A ≠ p_B

Thus the correct mapping between observations and latent state depends on:

• Regime identification  
• Delay compensation  
• Noise estimation  

---

### Contamination Process

Adversary selects contamination intervals:

t ∈ T_poison

Subject to budget:

|T_poison| / T ≤ ρ

During contaminated intervals:

Observations are drawn from arbitrary distributions Q_t satisfying bounded divergence constraints.

---

## Key Structural Property

For any fixed observation window length m:

I(y_{t−m:t}; x_{t+1}) ≈ 0

under sufficient delay, switching frequency, and noise.

This implies bounded-history predictors cannot recover predictive signal reliably.

---

## Evaluation Metrics

Separation is evaluated using suppression-aware metrics:

### Conditional Accuracy

Accuracy measured only when the system emits:

P(correct | emission)

---

### Calibration Error

Reliability of probabilistic predictions on emitted steps.

---

### Duty Cycle

Fraction of time the system emits non-null outputs.

---

### Epistemic Damage Bound

Maximum divergence from the clean posterior after contamination periods.

---

## Separation Theorems

### Theorem A — Failure of Forced-Output Predictors

For any fixed history length m below a threshold determined by delay and switching dynamics:

Any Class F predictor has conditional accuracy bounded by:

1/2 + ε

where ε → 0 under sufficient switching or delay.

---

### Theorem B — Unbounded Damage in Immortal Estimators

Under non-zero contamination budget ρ:

There exists an adversarial contamination strategy such that any Class I estimator experiences:

• Persistent regime misidentification  
• Non-vanishing calibration error  
• Irreversible belief corruption  

Infinitely often or in expectation.

---

### Theorem C — Bounded Damage and Recovery in CEG

Under the same environment:

The CEG kernel guarantees:

• Suppression during unobservable or contaminated periods  
• No assimilation of quarantined data  
• Bounded divergence from clean posterior  
• Recovery of calibration after sufficient clean exposure  

Conditional accuracy on emitted steps converges toward the optimal achievable level.

---

### Theorem J — Jurisdictional Sovereignty

If downstream consumers implement the Null-Payload Consumer Contract:

During suppression states:

NO_SIGNAL  
QUARANTINE  
RESET  
HALT  

Downstream action sets are restricted to predefined safe subsets.

Illegal actions become unreachable.

---

## Interpretation

These results demonstrate that:

• Forced emission is incompatible with reliable inference under adversarial non-stationarity.  
• Continuous assimilation without quarantine leads to irreversible epistemic damage.  
• Suppression, quarantine, and mortality are structurally necessary mechanisms.  

CEG’s architecture represents a distinct epistemic system class rather than an incremental refinement of existing estimators.

---

## Scope Limitations

This appendix provides:

• Formal witness environment definitions  
• Separation theorem statements  
• Structural proof intuition  

It does not include full mathematical proofs, which are reserved for future technical publications.

---

## Relationship to Kernel Specification

This appendix:

Supports the conceptual justification of the CEG kernel.

It does not define, alter, or extend kernel behavior.

The sole normative authority remains:

spec/CEG-KERNEL-v1.0.md

---

END