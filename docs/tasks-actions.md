# 4. Tasks and Actions for Governance Tiers

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) and quoted verbatim where referenced.
>
> **How to read this page:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about.

FIP-0118 fixes the powers of each tier. The SWA tier's discretionary powers each additionally require a published FIP; the SRA tier's registry changes require no per-change FIP except a code upgrade:

> SWA Governance discretionary powers (FIP-0118): *"add a stream (RegisterStream) or remove one (RemoveStream)"*, *"alter a stream: rewrite its Weight record (SetWeightRecords) or re-route its Distribution"*, *"tune the gate parameters (SetGateParams)"*, and *"upgrade the SWA's code"*.
>
> SRA Governance discretionary powers (FIP-0118): *"admit, remove, freeze, unfreeze, or replace an Orchestrator"*, *"reassign contested (payer, operator) bindings"*, *"correct or supply posted volumes during the verification window"*, *"maintain the admitted lists that scope FPV"*, and *"upgrade the SRA's code, the one registry change that requires a FIP"*.

---

## 4.1 SWA Governance

SWA Governance approves nothing routine: the w0 ramp and the volume gate are mechanism-executed. Every discretionary action writes to f02, requires a published and accepted FIP first, and is held for `SWA_TIMELOCK` (7 days) before it binds.

- **During every `SWA_TIMELOCK`:** monitor the f02 queue and the off-chain objection process, and act on a sustained objection within the expected response-time commitment of 7 days. The cancel entry point is the enforcement of that process; a cancellation missed is a window wasted.
- **Per discretionary write:** confirm the backing FIP is published and accepted before approval, and verify the schedule envelope (the sum of weights at most 1 across the whole segment) before submission.
- **Continuous:** keep Safe rosters and internal-implementation disclosures in this repository current.

> **Standard SWA discretionary-write flow** (referenced by 4.1.1–4.1.4):
>
> 1. Draft the exact write and carry a FIP through to 'Accepted' (including Last Call). No write is submitted without an accepted FIP.
> 2. Pre-submission check: verify the schedule envelope (Σw ≤ 1 across the whole segment) and that the write matches the accepted FIP.
> 3. Both SWA Safes approve `ProposeWrite(write)`.
> 4. The SWA relays to f02; f02 queues it with an `effective_epoch` and holds it for `SWA_TIMELOCK` (7 days).
> 5. Objection window: the queued write is public; monitor the off-chain objection process. Either SWA Safe may `Cancel(id)` on a mismatch, a missing FIP, or a sustained objection; absent a cancellation it binds at `effective_epoch`.
> 6. Record the outcome in this repository.

The L1 envelope check restates a FIP invariant:

> "f02 requires Σ_{i>=1} ComputeWeight(W_i, e) ≤ 1 at every epoch, so the burn residual w0 stays ≥ 0."

### SWA actions

<details>
<summary><strong>4.1.1 — Alter a stream: rewrite a Weight record (SetWeightRecords)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow above; the call is `SetWeightRecords`. Reducing a weight to zero is held exactly like removing the stream.

</details>

<details>
<summary><strong>4.1.2 — Add a stream (RegisterStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RegisterStream(id, weight_record, distribution, activation_epoch)`. Step 2 also confirms the stream and recipient caps and that `activation_epoch ≥ current_epoch + SWA_TIMELOCK`.

</details>

<details>
<summary><strong>4.1.3 — Remove a stream (RemoveStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RemoveStream(id)`. The removed weight reverts to the burn residual; removal creates no free value.

</details>

<details>
<summary><strong>4.1.4 — Re-point a stream's Distribution (SetDistribution)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `SetDistribution(id, distribution)`. Can be used, for instance, to re-point the service stream's designated writer to a redeployed SRA (such as the SRA-deadlock backstop). The current wallet-to-share map stays in force until the new writer overwrites it, so payments continue across the change.

</details>

<details>
<summary><strong>4.1.5 — Tune the gate parameters (SetGateParams)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

1. Publish and get an accepted FIP for the new gate parameters (step size, volume targets, escalation ratio).
2. Both SWA Safes approve `SetGateParams(params)`.
3. The change queues in the SWA's own state under the SWA-internal timelock (7 days). The gate rule itself is code and cannot change via a parameter write. Either Safe may cancel during the hold.
4. Record in this repository.

</details>

<details>
<summary><strong>4.1.6 — Upgrade the SWA's code</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

Same flow as 4.1.5; the FIP carries the upgrade. It executes through the pre-upgrade code, so the hold and checks still bind.

</details>

<details>
<summary><strong>4.1.7 — Replace an SWA Safe (cooperative)</strong></summary>

> **Requires:** both SWA Safes · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

1. Both SWA Safes approve replacing the registered Safe address.
2. It queues under the SWA-internal timelock; either Safe may cancel.
3. It binds; announce here with a post-mortem.

Hostile/deadlocked case (a Safe blocks its own replacement): the exit is one level up — a coordinated network upgrade migrates the SWA address in f02, always under a published FIP.

</details>

<details>
<summary><strong>4.1.8 — Cancel a pending SWA write</strong></summary>

> **Requires:** either SWA Safe alone · **When:** during the hold/window

Either Safe calls `Cancel(id)` (relayed to `f02.CancelPending(id)`), discarding a queued discretionary write. A cancellation only preserves the status quo. Mechanism writes (the gate step) are tagged at queue time and cannot be cancelled.

</details>

<details>
<summary><strong>4.1.9 — Monitor mechanism-executed updates</strong></summary>

The w0 ramp and the quarterly gate step (`QuarterlyGateCheck` → `SetWeightRecords`) run permissionlessly. Each quarter:

1. Confirm the queued w1 write matches the gate rule and the bound `AggregatedFPV(Q)`.
2. Note it is a scheduled write: it queues 7 days for visibility only and is not cancellable.
3. If a mismatch suggests compromise, escalate per the [compromise playbook](compromise-playbook.md); the remedy will be a Safe/code replacement, not cancelling the mechanism write.

</details>

<details>
<summary><strong>4.1.10 — Continuous upkeep</strong></summary>

During every `SWA_TIMELOCK`, monitor the f02 queue and the objection process and act within the published response-time commitment.

1. Keep Safe rosters and internal-implementation disclosures current.

</details>

---

## 4.2 SRA Governance

SRA Governance runs the registry and the quarterly verification duty. Registry changes need both Registry Safes and are held under **`SRA_CANCEL_HOLD`**, but need no per-change FIP (the one exception is a code upgrade). The quarterly verification runs against a public record and is deterministic.

> **Standard registry-change flow** (referenced by 4.2.1–4.2.5, 4.2.9–4.2.10):
>
> 1. Trigger and diligence (application, dispute outcome, audit finding, rotation request, or list update).
> 2. Both Registry Safes approve the relevant call.
> 3. The SRA queues the change under `SRA_CANCEL_HOLD`.
> 4. Either Registry Safe may `CancelPending(id)` during the hold; absent a cancellation it binds.
> 5. Record the outcome in the issue/repository.

The tasks are designed to be verifiable and for which corrections can be implemented against a public record. The recomputation is deterministic: any observer running the reference indexer over the same settlement events and registry state reaches the same figure.

### Admission (Phase 1: discretionary)

An Orchestrator's application is filed as an issue in this repository. SRA Governance scores the application and, if approved, executes the admit on-chain, subject to `SRA_CANCEL_HOLD`.

A uniqueness rule is fixed by the FIP:

> "Each (payer, operator) pair is bound to at most one orchestrator, so a registration that duplicates an existing binding reverts."

For Phase 2 (subject to a future FIP): admission becomes permissionless, enabled by a standard attribution-metadata interface specified in a future FIP. The rubric and diligence in this section apply to Phase 1 only.

### SRA actions

<details>
<summary><strong>4.2.1 — Admit an Orchestrator (Admit)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

1. Application filed as an issue (identity/team, funding plan, declared (payer, operator) pairs and measurement rules).
2. SRA Governance scores it against the admission rubric. Both Registry Safes approve `Admit(orch)`; the uniqueness rule reverts any pair already bound elsewhere.
3. Queues under `SRA_CANCEL_HOLD`; either Safe may cancel.
4. Binds; record in the issue.

</details>

<details>
<summary><strong>4.2.2 — Remove an Orchestrator (Remove)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Remove is permanent; unlike Freeze it is not reversible. It **releases** the Orchestrator's (payer, operator) bindings, freeing those pairs for re-registration.

</details>

<details>
<summary><strong>4.2.3 — Freeze an Orchestrator (Freeze)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Frozen means *admitted but suspended, and reversible*. While an Orchestrator is frozen:

- The frozen Orchestrator cannot call `RegisterPairs` or `PostVolume`;
- its FPV is excluded from `SplitRule` (its **share = 0**) and from `AggregatedFPV` for any quarter whose posting period closes while it is frozen — so a frozen Orchestrator neither earns nor counts toward the gate for that quarter;
- its existing (payer, operator) bindings **stay bound**: they cannot be poached during a dispute and move only via an explicit `ReassignBinding`.

Freeze and unfreeze are ordinary registry changes — both Safes, queued under `SRA_CANCEL_HOLD`, no FIP. Remove is the permanent counterpart, and it releases the Orchestrator's bindings.

</details>

<details>
<summary><strong>4.2.4 — Unfreeze an Orchestrator (Unfreeze)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

- Standard registry-change flow; the call is `Unfreeze(orch)`.
- Restores an admitted-but-suspended Orchestrator: it can again `RegisterPairs` and `PostVolume`, and its FPV counts in `SplitRule` and `AggregatedFPV` again from the next quarter whose posting period it participates in.
- Bindings are unchanged — they were retained throughout the freeze.

</details>

<details>
<summary><strong>4.2.5 — Replace / rotate an Orchestrator address (Replace)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Standard registry-change flow; the call is `Replace(old, new)`. Rotates a compromised or non-functioning address; f02 pays the new address from the effective epoch, so no payment is missed.

</details>

<details>
<summary><strong>4.2.6 — Reassign a contested (payer, operator) binding (ReassignBinding)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Standard registry-change flow following the dispute procedure ([4.2.8](#)); the call is `ReassignBinding(pair, orch)`. See also [§6.2 Dispute resolution](verification.md#62-dispute-resolution-for-contested-bindings).

</details>

<details>
<summary><strong>4.2.7 — Quarterly FPV verification & correction (CorrectVolume)</strong></summary>

> **Requires:** both SRA Safes (jointly) · **Hold:** none (the verification window is the hold) · **Enforced by:** SRA · **Cancel:** not cancellable

1. Recompute each posted FPV_i(Q) from public settlement events + registry state, using the versioned reference indexer.
2. Publish the recomputation.
3. During the verification window, both Safes jointly call `CorrectVolume(orch, Q, value)` to replace a misreport or supply a figure for a non-poster. A value neither posted nor supplied binds at zero (conservative under-count).
4. Values bind when the window closes; there is no separate cancellation.
5. A contested correction is appealed via the [dispute process](verification.md#62-dispute-resolution-for-contested-bindings); the outcome affects only later quarters.

FIP-0118 fixes the binding behavior:

> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter."
>
> "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."

</details>

<details>
<summary><strong>4.2.8 — Dispute resolution for contested bindings</strong></summary>

1. The contesting party opens an issue identifying the pair and its claim.
2. SRA Governance requests evidence from both parties (increasing strength: business records; client confirmation via a verifiable channel; a signed payer-wallet attestation naming the orchestrator).
3. Decide the outcome. Reassignment or removal executes on-chain via the standard registry-change flow (4.2.2–4.2.6), subject to `SRA_CANCEL_HOLD`.
4. Record the resolution in the issue.

Full procedure and evidence standards: [§6.2](verification.md#62-dispute-resolution-for-contested-bindings).

</details>

<details>
<summary><strong>4.2.9 — Exception-based verification (freeze/removal)</strong></summary>

1. Trigger: an anomaly in public settlement data, and/or monitoring dashboards, and/or a community report. Audits are never initiated as pre-verification.
2. Investigate: recompute FPV and examine the (payer, operator) relationships for wash trading, self-dealing, misreported FPV, or binding fraud.
3. Decide grounds.
4. Act: `Remove(orch)` (4.2.2) or `Freeze(orch)` (4.2.3) via the standard registry-change flow, subject to `SRA_CANCEL_HOLD`.
5. Record the finding and rationale.

</details>

<details>
<summary><strong>4.2.10 — Maintain admitted lists & oracle config (SetAdmittedLists, oracle settings)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Standard registry-change flow for the admitted-stablecoin whitelist, the admitted Filecoin Pay contract addresses, and the FIL/USD oracle configuration (feed address, staleness threshold). These are gate-consequential, so they are held like any registry change — never a silent edit.

</details>

<details>
<summary><strong>4.2.11 — Upgrade the SRA's code</strong></summary>

> **Requires:** both SRA Safes + accepted FIP · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe

The one registry change that requires a FIP (e.g., the Phase 2 permissionless-admission transition). Otherwise follows the standard flow.

</details>

<details>
<summary><strong>4.2.12 — Replace an SRA Safe (cooperative)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe

Both Safes approve the replacement; it queues under the hold; either may cancel; it binds and is announced with a post-mortem.

Hostile/deadlocked case: a SWA write re-points the service stream's writer to a redeployed SRA, always under a published FIP; registry state is reconstructible from public data.

</details>

<details>
<summary><strong>4.2.13 — Cancel a pending registry change</strong></summary>

> **Requires:** either SRA Safe alone · **When:** during `SRA_CANCEL_HOLD`

Either Safe calls `CancelPending(id)`, discarding the queued change. It only preserves the status quo.

</details>

<details>
<summary><strong>4.2.14 — Monitor mechanism-executed updates (oversight, no approval)</strong></summary>

`FinalizeConversion(Q)` and `SubmitShares(Q)` run permissionlessly and are not held. Duty: confirm each has run and that `SubmitShares` wrote the correct wallet-to-share map (integrity rests on the upstream verification window). Neither is cancellable.

</details>

<details>
<summary><strong>4.2.15 — Continuous upkeep</strong></summary>

Maintain and version the reference indexer; monitor anomaly reports, contested bindings, and the cancellation queue; keep Safe rosters and disclosures current.

</details>

---

← Previous: [3. Compromise and rotation playbook](compromise-playbook.md) · [Back to README](../README.md) · Next: [5. Orchestrator Operational Guidelines](orchestrator-guidelines.md) →
