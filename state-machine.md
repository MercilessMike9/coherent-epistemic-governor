CEG Kernel v1.0 — State Machine (Informative)
Status: Informative visualization
Normative authority: spec/CEG-KERNEL-v1.0.md

This document illustrates runtime state transitions defined normatively in the canonical kernel specification.
It does not override or modify kernel semantics.
If any discrepancy exists, the canonical specification governs.


State Diagram

stateDiagram-v2
direction LR

[*] --> OK : Initialize (G0 pass)

OK --> DEGRADED : O_t drops but ≥ O_blind
OK --> NO_SIGNAL : O_t < O_blind
OK --> QUARANTINE : C_t > C_quarantine
OK --> RESET : fail/blind/mismatch ≥ thresholds

DEGRADED --> OK : Observability recovers
DEGRADED --> NO_SIGNAL : O_t < O_blind
DEGRADED --> QUARANTINE : C_t exceeds threshold
DEGRADED --> RESET : Persistent violations

NO_SIGNAL --> OK : Observability restored
NO_SIGNAL --> DEGRADED : Partial observability
NO_SIGNAL --> RESET : blind_count ≥ K_blind

QUARANTINE --> RESET : Violations persist
QUARANTINE --> OK : C_t ≤ exit threshold for required duration

RESET --> OK : Checkpoint OR prior reinit succeeds
RESET --> HALT : Numerical invalidity or illegal configuration

OK --> HALT : Unrecoverable numerical failure
DEGRADED --> HALT
NO_SIGNAL --> HALT
QUARANTINE --> HALT
RESET --> HALT

HALT --> [*]


State Semantics Summary

OK — emission allowed; assimilation allowed
DEGRADED — emission allowed (flagged); assimilation allowed
NO_SIGNAL — emission suppressed; predict-only
QUARANTINE — emission suppressed; assimilation forbidden
RESET — emission suppressed; rollback + humility expansion
HALT — emission suppressed; terminal


Status Precedence Rule (Normative Reminder)

When multiple conditions hold simultaneously:

HALT > QUARANTINE > NO_SIGNAL > DEGRADED > OK


Interpretation Notice

This diagram is non-normative and illustrative.
The sole normative authority is:

spec/CEG-KERNEL-v1.0.md

END