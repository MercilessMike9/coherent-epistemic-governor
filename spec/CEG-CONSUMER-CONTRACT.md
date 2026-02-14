# CEG Kernel v1.0 — Consumer Contract: Null-Payload Semantics (Informative)

**Status:** Informative (integration contract)  
**Normative authority:** `spec/CEG-KERNEL-v1.0.md`  
**Scope:** This document specifies **consumer-side** requirements for interpreting kernel emissions, especially **suppression** (null payload) behavior. It does **not** modify kernel semantics. If any discrepancy exists, the canonical kernel specification governs.

---

## 1. Purpose

The CEG Kernel can **suppress** epistemic output (emit **no estimates**) when invariants cannot be guaranteed.  
This contract prevents downstream consumers from defeating suppression by **imputing**, **smoothing**, **caching**, or otherwise continuing as if a trustworthy estimate exists.

---

## 2. Definitions

**Kernel payload (emission):** A structured output containing, at minimum, an estimate triple:
- `x_hat` (state estimate)
- `P` (uncertainty / covariance)
- `mu` (regime weights)

**Suppressed payload (triple-null):**  
A payload is **suppressed** iff:
- `x_hat = ∅` AND `P = ∅` AND `mu = ∅`  
(Concrete encoding is consumer-defined; recommended is explicit JSON `null` values for each field.)

**SAM — Suppressed Authority Mode:**  
A consumer state indicating: **no kernel epistemic authority is available**.

**Tick:** One discrete kernel emission time step `t`.

---

## 3. Compliance Requirements (Normative for Consumers Claiming CEG-Compatibility)

### C.1 — Suppression Recognition (MUST)
Upon receiving a kernel emission with a **suppressed payload (triple-null)**, the consumer **MUST** enter **SAM** immediately (same tick).

- No configuration flag, environment setting, compatibility mode, or runtime option may bypass C.1.
- Failure to implement C.1 means the consumer **cannot claim** CEG-compatible epistemic jurisdiction.

### C.2 — SAM Behavior (MUST)
While in SAM, the consumer **MUST**:
1. Treat kernel epistemic authority as **NONE**.
2. Prohibit any internal or external decision that **relies on kernel estimates**.
3. Propagate a machine-readable signal downstream:
   - `epistemic_authority = "NONE"` AND
   - `epistemic_veto_active = true`

### C.3 — Forbidden Continuity / Imputation (MUST NOT)
While in SAM, the consumer **MUST NOT** do any of the following (whether explicit or implicit):

- **Hold-last-value** beyond **one tick** as a substitute for kernel estimates.
- **Smoothing / filtering** over prior non-null kernel outputs to fill gaps.
- **Imputation** from other modalities unless those modalities are themselves governed by a non-suppressed CEG-compatible epistemic channel.
- **Fallback to regime centroid / mode mean** without explicit operator authorization (see C.5).
- **Silent substitution** of cached estimates or defaults while presenting normal confidence.

### C.4 — Degraded vs Suppressed Distinction (MUST)
Consumers **MUST** distinguish:
- **DEGRADED**: kernel emits estimates but marks reduced authority (consumer may continue with explicit degraded flagging).
- **SUPPRESSED (SAM)**: kernel emits no estimates; consumer authority is NONE.

Consumers **MUST NOT** treat suppression as “degraded.”

### C.5 — Human Acknowledgement for Override (SHOULD; MUST in High-Stakes)
If a consumer permits any override path during SAM (not recommended), it must be:
- **Explicit**, **operator-initiated**, and **time-bounded**
- Recorded in the audit log (C.6)
- Visibly indicated in UI as “Operating without kernel epistemic authority”

Recommended baseline:
- SAM requires **operator acknowledgement** before resuming reliance after recovery.

High-stakes requirement (medical/financial/physical actuation contexts):
- SAM implies a **hard actuation lock** until recovery is confirmed.

### C.6 — Auditability (MUST)
Every transition into and out of SAM **MUST** be logged as an append-only event containing:
- `timestamp`
- `consumer_id`
- `kernel_status` (e.g., `NO_SIGNAL`, `QUARANTINE`, `RESET`, `HALT`, if provided)
- `reason_code` (consumer-local mapping allowed; see §4)
- `operator_action` (if any; include operator identity/auth token reference if applicable)
- `previous_authority` → `new_authority`

Logs **MUST** be tamper-evident (append-only store, or hash-chained records, or external write-once logging).

---

## 4. Minimal Reason Codes (Recommended)

Consumers may map kernel status to reason codes without adding semantics:

- `SUPPRESS_NO_SIGNAL` — observability below threshold
- `SUPPRESS_QUARANTINE` — contamination present; assimilation forbidden
- `SUPPRESS_RESET` — transient reset / reload in progress
- `SUPPRESS_HALT` — terminal; no recovery
- `SUPPRESS_UNKNOWN` — catch-all (still triggers SAM)

---

## 5. Reference State Logic (Informative)

### 5.1 — Decision Procedure (single-tick)
1. If triple-null ⇒ enter SAM, emit veto flags, stop reliance.
2. Else if non-null and previously in SAM:
   - remain conservative: require explicit recovery gating (operator acknowledgement or a stability window).
3. Else normal consumption path.

### 5.2 — Recommended Recovery Gate (Informative)
To avoid “flapping”:
- require `N_recover` consecutive non-null emissions before exiting SAM, OR
- require an explicit operator acknowledgement after first non-null emission, OR
- both (preferred for high-stakes).

---

## 6. Reference Shim Pseudocode (Informative)

```python
def is_suppressed(payload: dict) -> bool:
    return payload.get("x_hat") is None and payload.get("P") is None and payload.get("mu") is None

def on_kernel_emission(payload: dict, state: dict) -> dict:
    if is_suppressed(payload):
        state["mode"] = "SAM"
        audit("SAM_ENTER", payload, state)
        return {
            "epistemic_authority": "NONE",
            "epistemic_veto_active": True,
            "action_permitted": False,
        }

    # Non-null emission:
    if state.get("mode") == "SAM":
        # Conservative recovery: require operator ack OR N consecutive good ticks
        if not recovery_gate_satisfied(state, payload):
            return {
                "epistemic_authority": "NONE",
                "epistemic_veto_active": True,
                "action_permitted": False,
            }
        state["mode"] = "NORMAL"
        audit("SAM_EXIT", payload, state)

    return {
        "epistemic_authority": "KERNEL",
        "epistemic_veto_active": False,
        "action_permitted": True,
        "kernel_payload": payload,
    }