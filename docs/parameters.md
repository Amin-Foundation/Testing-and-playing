# 7. Parameters

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) and quoted verbatim where referenced.

## 7.1 Safe Addresses

Each tier consists of two organization multisigs (Safes) registered in the contract it governs. The protocol sees only the two addresses.

| Tier | Contract governed | Filecoin Foundation Safe (address) | FilOZ Safe (address) | Rule (fixed by the FIP) |
| :-- | :-- | :-- | :-- | :-- |
| SWA Governance | Stream Weights Actor (SWA) | TBD | TBD | Both approve; either alone cancels |
| SRA Governance | Service Rewards Actor (SRA) | TBD | TBD | Both approve; either alone cancels |

Each organization runs a separate Safe per tier — four accounts in total — so approvals cannot be replayed across surfaces.

> The approval rule is fixed by FIP-0118:
>
> > "Both Safes approve, the change is published and held, and either Safe can cancel during the hold."

## 7.2 Governed Parameters

| Parameter | Where set | Value |
| :-- | :-- | :-- |
| `SWA_TIMELOCK` (objection window on SWA writes to f02) | FIP-fixed, enforced in f02 | 7 days |
| SWA-internal timelock (own state and code) | FIP-fixed, equal to the f02 window | 7 days |
| `SRA_CANCEL_HOLD` | This repository | TBD |
| `POST_PERIOD` (FPV posting) | This repository | TBD |
| `VERIFICATION_WINDOW` (FPV verification) | This repository | TBD |
| Dispute resolution target ([6.2](verification.md#62-dispute-resolution-for-contested-bindings)) | This repository | 7 days |
| Orchestrator response window ([6.4.2](verification.md#642-escalation)) | This repository | 72 hours |
| Admitted-stablecoin whitelist | This repository | TBD |
| FIL/USD oracle feed and staleness threshold | This repository | TBD |
| Admission rubric | This repository | TBD, after the first application cycle |

> `SWA_TIMELOCK` is FIP-fixed and enforced in L1:
>
> > "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

---

← Previous: [6. Declaration, verification and escalation](verification.md) · [Back to README](../README.md)
