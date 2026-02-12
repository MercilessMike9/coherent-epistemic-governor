# CEG Kernel v1.0 — State Machine (Informative)

Status: Informative visualization
Normative authority: spec/CEG-KERNEL-v1.0.md

This document illustrates the runtime state transitions defined normatively in the canonical kernel specification. It does not override, extend, or modify any kernel semantics. If any discrepancy exists, the canonical specification governs.

---

## State Diagram

stateDiagram-v2
    direction LR

    [*] --> OK : Initialize (G0 pass)

    OK --> DEGRADED : O_t drops to degraded range but ≥ O_blind
    OK --> NO_SIGNAL : O_t < O_blind
    OK --> QUARANTINE : C_t > C_quarantine
    OK --> RESET : fail_count / blind_count / mismatch_count ≥ K_*

    DEGRADED --> OK : Observability recovers
    DEGRADED --> NO_SIGNAL : O_t < O_blind
    DEGRADED --> QUARANTINE : C_t > threshold
    DEGRADED --> RESET : Persistent violations

    NO_SIGNAL --> OK : Observability restored
    NO_SIGNAL --> DEGRADED : Partial observability returns
    NO_SIGNAL --> RESET : blind_count ≥ K_blind

    QUARANTINE --> RESET : Violations persist
    QUARANTINE --> NO_SIGNAL : Contamination persists but observability drops

    RESET --> OK : Checkpoint reload succeeds
    RESET --> HALT : Reload fails

    OK --> HALT : Unrecoverable numerical invalidity
    DEGRADED --> HALT
    NO_SIGNAL --> HALT
    QUARANTINE --> HALT
    RESET --> HALT

    HALT --> [*]

---

## State Semantics Summary

OK — emission allowed; assimilation allowed  
DEGRADED — emission allowed (flagged); assimilation allowed  
NO_SIGNAL — emission suppressed; predict-only  
QUARANTINE — emission suppressed; assimilation forbidden  
RESET — emission suppressed; reset step  
HALT — emission suppressed; stopped

---

## Interpretation Notice

This diagram is provided solely as an implementation and audit aid.

It is non-normative and illustrative.

The sole normative authority for kernel behavior is:

spec/CEG-KERNEL-v1.0.md

END