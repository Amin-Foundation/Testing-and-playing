# 2. Governance Tiers and Safes

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) and quoted verbatim where referenced. This page describes *how* the two tiers operate.

Solstice governance is split across two contracts, each operated by a tier composed of two independent organization multisigs (Safes). FIP-0118 fixes the separation and the approval structure:

> "No address in the registry may participate in either contract governance tier, as a Safe or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

## 2.1 Standards for a governance Safe (both tiers)

- The organization is publicly identified, has a track record in Filecoin governance or engineering, and is independent of the other organization in the same tier.
- Recommended, not protocol-enforced: an internal threshold of at least 2, and hardware key custody.
- No orchestrator role for the organization or any of its key holders. Per the FIP, no registry address may participate in either tier, as a Safe or as a key holder within one.
- Internal member rotation is org-internal and immediate; it requires no governance action.

## 2.2 Approval and cancellation rules (both tiers)

These rules are fixed by the FIP and restated here for operators.

> FIP-0118, on the change lifecycle:
>
> > "Every discretionary change follows the same path: both Safes approve, the change is published and held, and either Safe can cancel during the hold."

**Mechanism-executed updates** take no approval and are not cancellable. The quarterly gate check and the `SetShares` recompute run permissionlessly through entry points that can emit only the value the mechanism computes; their integrity guard sits upstream in the FPV verification window. The tiers exercise no judgment over these updates because there is no judgment to exercise.

> FIP-0118 on why mechanism writes cannot be cancelled:
>
> > "Mechanism-executed updates cannot be cancelled: the gate's quarterly w2 write queues in f02 under SWA_TIMELOCK for visibility, but it is tagged as a mechanism write at queue time and CancelPending rejects it."

**Discretionary changes** are as follows:

- **Approval:** both of the tier's Safes. SWA writes additionally require a published and accepted FIP. Registry changes deliberately carry no per-change FIP requirement; the carve-out covers data within the rules, never code. An upgrade of either contract's code requires a FIP.
- **Cancellation:** either Safe alone, during the hold or window. This grants no new power, since either Safe could have withheld approval. It lets an organization act on what is learned during the window, such as a community objection or a discovered mistake, without a fresh joint round. The off-chain objection process is the judgment layer; the Safe-level cancellation is its enforcement.

## 2.3 Summary of holds

Informative summary; the lifecycle table in the FIP is normative.

| Action | Hold | Enforced by | Cancellable by |
| :-- | :-- | :-- | :-- |
| Discretionary SWA write to f02 (alter, add, or remove a stream; re-point a Distribution) | `SWA_TIMELOCK`, 7 days (FIP-fixed) | f02 | Either SWA Safe |
| SWA internal change (Safe replacement, gate parameters, code upgrade) | Internal timelock, equal to the f02 window (FIP-fixed) | SWA | Either SWA Safe |
| Quarterly gate step | 7-day queue, visibility only | f02 | No one (mechanism-executed) |
| Registry change (admit, remove, freeze, unfreeze, replace, reassign, Safe replacement) | `SRA_CANCEL_HOLD` (TBD) | SRA | Either SRA Safe |
| Registry code upgrade | Requires an approved FIP | SRA | Either SRA Safe |
| CorrectVolume | None; bounded by the verification window | SRA | Not cancellable |
| SetShares, PostVolume, RegisterPairs | None; bounded by the window, the posting period, and the uniqueness check | SRA | Not cancellable |

> FIP-0118 fixes the `SWA_TIMELOCK` hold in L1:
>
> > "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

---

← [Back to README](../README.md) · Next: [3. Compromise and rotation playbook](compromise-playbook.md) →
