# 3. Compromise and rotation playbook (both tiers)

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) and quoted verbatim where referenced.

The threat model rests on the two-Safes rule: no single Safe can make a change bind, and either Safe can cancel a pending change during the hold. FIP-0118 fixes the L1 backstop that makes even a fully compromised Safe unable to bind early:

> "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

## 3.1 Key compromise inside a Safe (below the internal threshold)

1. The organization reports publicly in this repository.
2. Any pending change the Safe approved while the key was compromised is reviewed; if suspect, either Safe cancels it.
3. The organization rotates its internal membership (org-internal, immediate, no governance action) and announces the rotation here.

## 3.2 A whole Safe compromised (internal threshold reached by an attacker)

1. Nothing binds. Approval requires the other Safe, and the honest organization cancels each malicious pending change during the hold or window. Repeated resubmission restarts the hold and is publicly visible; the attacker cannot bind a change silently. On the SWA side, a malicious write also lacks its required published FIP, making the objection case unambiguous.
2. The tier is treated as frozen (halted/suspended), and frozen consequences are bounded: registry frozen means payments and `SetShares` continue; SWA frozen means discretionary changes stop while the ramp and the gate continue through the permissionless crank.
3. Exit: replace the compromised Safe address. This requires both Safes, so if the compromised Safe obstructs, the FIP backstop applies (see [3.3](#33-replacing-a-registered-safe)).

## 3.3 Replacing a registered Safe

**Cooperative case:** both Safes approve the replacement; it queues under the hold or window and is cancellable like any change; announced here with a post-mortem.

**Hostile or deadlocked case, always under a published FIP:** for SRA Governance, a SWA write re-points the service stream's designated writer to a redeployed SRA; registry state is reconstructible from public data, and no network upgrade is needed. For SWA Governance, a coordinated network upgrade migrates the SWA address in f02.

> **Open question:** whether any fast path short of a FIP is acceptable, and what covers simultaneous compromise of both tiers.

---

← Previous: [2. Governance Tiers and Safes](governance-tiers.md) · [Back to README](../README.md) · Next: [4. Tasks and Actions for Governance Tiers](tasks-actions.md) →
