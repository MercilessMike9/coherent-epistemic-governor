# CEG Sidecar v0.1 — Epistemic Sidecar Protocol (Normative for this file only)

Author/Owner: Michael Middleton  
Date: February 2026  
Status: NON-NORMATIVE relative to CEG Kernel v1.0; NORMATIVE for Sidecar v0.1 only.

## 0) Relationship to CEG Kernel v1.0 (Hard)
- This document does NOT modify, override, or extend the normative authority of `spec/CEG-KERNEL-v1.0.md`.
- CEG Kernel v1.0 remains the sole normative authority for kernel behavior and kernel semantics.
- The Sidecar is an integration pattern and interface contract for deploying the kernel logic externally.

## 1) Purpose
The Sidecar is a separable “belief appliance” that governs epistemic commitment at runtime by enforcing:
- emission jurisdiction (allow/suppress)
- observability gating
- quarantine / reset semantics
- structured humility (as defined by the kernel spec)
- leak-minimizing public surface

The Sidecar is designed to be attachable to any upstream producer (LLM, estimator, tool controller, sensor fusion stack) without retraining or modifying weights.

## 2) Hard Non-Goals (must not exist in Sidecar)
The Sidecar must not implement:
- action selection, planning, optimization, utility/reward, policies
- tool invocation (the Sidecar may veto a tool call, but never issue one)
- self-modification or learning updates
- narrative memory or persona persistence

The Sidecar is a gatekeeper for epistemic outputs only.

## 3) Deployment Models (Two Modes)
### Mode A — Independent Estimation (Recommended)
The Sidecar runs its own CEG Kernel instance as an estimator using observations and config:
- It produces its own (x_hat, P, mu, status, gate_summary).
- Upstream claims are treated as untrusted and may be ignored or compared only as telemetry.

### Mode B — Claim Verification (Optional)
The Sidecar does not estimate; it verifies upstream-provided claims against kernel invariants and gates:
- It checks PSD/simplex, observability semantics, suppression rules, quarantine rules, and leak rules.
- If verification cannot be made airtight, Sidecar must fail-closed (suppress).

Normative requirement: An implementation MUST declare which mode it runs (A or B).

## 4) Sidecar I/O Contract (Normative)
### 4.1 Inputs per tick t
The Sidecar receives a single tick message containing:

A) Observation bundle (required)
- t : integer tick index
- dt_sec : number (fixed cadence)
- y : array[number] OR null
- valid_mask : array[0|1] length m (required if y present; else allowed but ignored)
- age_sec : number >= 0
- quality_flags : opaque flags (implementation-defined), mapped deterministically to severity q in {0,1,2,3}

B) Optional upstream claim bundle (optional; used in Mode B or telemetry)
- x_hat_up : array[number] OR null
- P_up : array[array[number]] OR null
- mu_up : array[number] OR null
- upstream_metadata : object (optional, non-normative)

C) Static config identity (required)
- config_id : string
- kernel_spec_id : string = "ceg-kernel-v1.0"
- sidecar_spec_id : string = "ceg-sidecar-v0.1"

### 4.2 Outputs per tick t (Public Surface)
The Sidecar outputs:

Always:
- status : enum {OK, DEGRADED, NO_SIGNAL, RESET, QUARANTINE, HALT}
- gate_summary : fixed bitmask or fixed enums (implementation-defined but frozen per deployment)
- emit_allowed : boolean
- kernel_spec_id : "ceg-kernel-v1.0"
- sidecar_spec_id : "ceg-sidecar-v0.1"
- config_id : string

If emit_allowed = true:
- x_hat : array[number]
- P : array[array[number]]
- mu : array[number]
- optional_public_diagnostics (optional, must be explicitly whitelisted):
  - O_t, H(mu), age_sec, mask_fraction, flags_severity_q

If emit_allowed = false:
- x_hat = null
- P = null
- mu = null
- NIS must be null / omitted

### 4.3 Output Suppression (Hard)
Sidecar MUST enforce the kernel’s suppression semantics:
If status ∈ {HALT, NO_SIGNAL, QUARANTINE} then:
- emit_allowed = false
- x_hat, P, mu must be null
- no other epistemic content may be leaked

DEGRADED:
- emit_allowed may be true (allowed) but must be explicitly flagged in status/gate_summary.

## 5) Attestation Token (Optional but Recommended)
If emit_allowed = true, the Sidecar MAY attach an attestation object:

attestation:
- attestation_version : "v0"
- issued_at_unix_sec : integer
- decision : "ALLOW"
- public_fields_hash : hex string (hash of the emitted public fields)
- signature : string (HMAC or public-key signature; deployment-defined)

If emit_allowed = false:
- decision : "SUPPRESS"
- signature may be omitted (fail-closed)

Hard rule: attestation must not include x_hat/P/mu themselves—only hashes of whitelisted public fields.

## 6) Security & Failure Semantics (Hard)
- Fail-closed is mandatory: if parsing fails, mapping fails, invariants cannot be verified, or mode declaration is inconsistent → status = HALT and suppress.
- The Sidecar must treat upstream claims as adversarial by default.
- The Sidecar must not become a covert channel: timing, error strings, or variable-length fields must not encode suppressed epistemic content.

## 7) Acceptance Tests (Normative for Sidecar v0.1)
S0 Mode declaration required; unknown mode → HALT + suppress.
S1 If NO_SIGNAL/QUARANTINE/HALT → emitted payload must be null.
S2 DEGRADED may emit only if explicitly flagged; never emit under NO_SIGNAL/QUARANTINE/HALT.
S3 If inputs missing required fields → HALT + suppress.
S4 Attestation (if present) contains only hashes/signatures of public fields; no raw epistemic content.
S5 Long adversarial run: every tick either suppresses or emits a state satisfying kernel invariants when emission is allowed.

END — CEG Sidecar v0.1
