# 6. Declaration, verification and escalation

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) and quoted verbatim where referenced.

## 6.1 Binding declarations and verification

Declarations are accepted by default, and rely upon a mix of on-chain verifiable metrics and other KPIs/SLAs. Verification is dispute-driven: it is triggered only based upon provable gaps highlighted by community members. A false claim surfaces when the legitimate orchestrator's registration and performance cannot be proven via either on-chain transactions or credible probing documentation for off-chain transactions. Verification methods per our integrity guidelines are set out in the sections below.

FIP-0118 fixes the consequence of a misreport discovered after binding:

> "There is no retroactive clawback: a misreport discovered after binding is grounds for audit, freeze, or removal."

## 6.2 Dispute resolution for contested bindings

1. The contesting party opens an issue in this repository identifying the pair and its claim.
2. SRA Governance requests evidence of the client relationship from both parties. Acceptable evidence, in increasing strength: business records showing the engagement; client confirmation through a verifiable channel; a signed client attestation (a message signed by the payer wallet naming the orchestrator).
3. Resolution target timeline: 7 days. The outcome (reassignment, removal, or no action) executes on-chain, subject to the hold, and is recorded in the issue.

A contested `CorrectVolume` follows the same evidence process. Per the FIP, bound values are final, so the appeal outcome can only affect later quarters:

> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter."

## 6.3 Verification, freeze, and removal

One inversion from the allocator program, and it is deliberate: allocator pathways were audited on a schedule, at each DataCap refresh. Orchestrator verifications are exception-based, per the FIP, and are never pre-clearance. A scheduled audit cycle would rebuild the review pipeline Solstice deprecates. A verification begins only when a trigger fires: a public-data anomaly, a community report filed as an issue, or a contested binding escalated from the dispute process ([6.2](#62-dispute-resolution-for-contested-bindings)).

**Grounds for freeze or removal:** wash trading, self-dealing (settling volume between parties under common control), misreported FPV (mechanically detectable, since FPV is recomputable from public events), and binding fraud.

## 6.4.1 What triggers a verification

- A public-data anomaly surfaced by the quarterly FPV recomputation or by monitoring (volume spikes at quarter/gate boundaries, figures that clear the target by a hair, volume with no matching storage activity).
- A community report filed as an issue in this repository.
- A contested binding escalated from the dispute process.

## 6.4.2 Escalation

1. **Trigger** → run the relevant on-chain checks (see [§5.2](orchestrator-guidelines.md#52-orchestrators-tasks-and-actions) and the evidence paths below); the recomputation is deterministic and public.
2. **Request evidence** if an anomaly holds: open a public issue and request on- and off-chain evidence; the orchestrator answers within the response window (72 hours).
3. **Interim freeze** where funds or the gate are at live risk: `Freeze(orch)` — the Orchestrator is suspended (cannot `RegisterPairs`/`PostVolume`; share = 0; excluded from `AggregatedFPV` for any quarter closing while frozen), its bindings stay put, and other Orchestrators keep being paid.
4. **Decide:** reinstate (unfreeze), reassign a binding, or `Remove(orch)`.
5. All on-chain actions run through the standard registry-change flow under **`SRA_CANCEL_HOLD`** and are recorded, with rationale, in the issue.

## 6.4.3 Cross-cutting verification measures

- **Reproducibility:** every verification conclusion must be reproducible from public data via the versioned reference indexer; findings cite the events and blocks used.
- **Sanctions cadence:** re-screen registered addresses on a fixed cadence and at each admission, not only on trigger.
- **Conflict attestations:** collect a conflict-of-interest / no-common-control attestation at admission and on any roster or binding change.
- **Verification trail:** triggers, evidence, decisions, and on-chain actions are all logged in this repository.

**Procedure (recap):**

1. Public issue in this repository.
2. Orchestrator response window (72 hours).
3. Governance decision.
4. On-chain action subject to `SRA_CANCEL_HOLD`.

## 6.5 Verification paths (on- and off-chain)

These are the integrity guidelines referenced in [6.1](#61-binding-declarations-and-verification).

### 6.5.1 On-chain evidence path

Where settlement events, payer wallet history, and any declared service-contract metadata resolve the question. No orchestrator action is required, and the conclusion cites the events and blocks used.

| Check | What it detects | How |
| :-- | :-- | :-- |
| FPV recomputation & reconciliation | misreported FPV | Re-derive FPV_i(Q) from public Filecoin Pay settlement events (`RailSettled` + reconstructed one-time payments) scoped to the orchestrator's registered (payer, operator) pairs; compare to the posted figure. |
| Uniqueness & overlap scan | double-attribution, collisions | Check the registry against on-chain rails; flag any (payer, operator) pair bound twice or claimed by two orchestrators. |
| Circular-flow / wash-trade graph analysis | wash trading | Trace payer → payee flows on the rails; flag funds round-tripping back to the payer, operator, or related wallets, the same value settling repeatedly, and quarter-end spikes timed to the gate. |
| Funding-source & common-control clustering | self-dealing | Trace who funds the declared payer wallets; cluster payer / operator / orchestrator / SP-payee addresses; flag common control or "clients" funded by the orchestrator's own treasury. |
| Volume-vs-activity cross-check | fabricated demand | Compare settled volume to real on-chain service for the same clients (sectors onboarded, PDP proofs, retrievals); payment with no corresponding storage or retrieval is a red flag. |
| Rail & asset scope validation | volume inflation via out-of-scope rails/assets | Confirm settlements use admitted Filecoin Pay contracts and admitted stablecoins, and that FIL volume converts at the correct 30-day TWAP; discard out-of-scope volume. |
| Share & payout verification | mis-split or payout error | Recompute `SplitRule` over the bound volumes; confirm the `SetShares` map and the f02 payouts match. |

### 6.5.2 Off-chain evidence path

Where a trigger fires and public data cannot resolve it, SRA Governance requests evidence through a public issue. Acceptable evidence, in increasing strength: business records showing the engagement; client confirmation through a verifiable channel; a signed payer-wallet attestation naming the orchestrator.

Documentation must show genuine separation between the parties whose relationship is in question.

### 6.5.3 Process

1. Trigger logged as a public issue in this repository.
2. The orchestrator responds within the response window (72 hours).
3. Interim freeze where funds or the gate are at live risk.
4. Decision: reinstate, reassign a binding, or remove.
5. On-chain actions run through the standard registry-change flow under `SRA_CANCEL_HOLD` and are recorded, with rationale, in the issue.

---

← Previous: [5. Orchestrator Operational Guidelines](orchestrator-guidelines.md) · [Back to README](../README.md) · Next: [7. Parameters](parameters.md) →
