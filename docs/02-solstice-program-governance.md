# 2. Solstice Program Governance

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) and referenced throughout. This section describes *how* the program is governed and operated.

This section consolidates the operating manual for the two governance tiers: the standards behind each Safe, the tasks and actions each tier performs, the registry of admitted Orchestrators, the program parameters, and the safety and rotation playbook.

**Contents**

- [2.1 Governance Tiers and Safes](#21-governance-tiers-and-safes)
- [2.2 SWA Governance Tier — Tasks and Actions](#22-swa-governance-tier--tasks-and-actions)
- [2.3 SRA Governance Tier — Tasks and Actions](#23-sra-governance-tier--tasks-and-actions) (including the Orchestrator Registry at 2.3.12)
- [2.4 Parameters](#24-parameters)
- [2.5 Safety and rotation playbook (both tiers)](#25-safety-and-rotation-playbook-both-tiers)

---

## 2.1 Governance Tiers and Safes

Solstice governance is split across two contracts, each operated by a tier composed of two independent organization multisigs (Safes): **Tier 1, the SWA Governance Tier (§2.2)**, and **Tier 2, the SRA Governance Tier (§2.3)**. FIP-0118 fixes the separation and the approval structure:

> "No address in the registry may participate in either contract governance tier, as a Safe or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

Governance is a bespoke UnanimousGovernance contract, not a nested Safe. Each entity's Safe handles co-signing, but the post-approval hold and the standing unilateral cancel are the contract's own logic, as nesting Safes solves co-signing, not objection. For SWA writes the hold is additionally enforced at f02 (SWA_TIMELOCK), so it survives a compromised or maliciously upgraded governance contract. How each owner is implemented internally (incl. a nested multisig) is the entity's operational discretion.

### 2.1.1 Standards for a governance Safe (both tiers)

- The organization has a track record in Filecoin governance or engineering, and is independent of the other organization in the same tier.
- Recommended, not protocol-enforced: an internal threshold of at least 2, and hardware key custody.
- No registry address may participate in either governance tier, as a Safe or as a key holder within one.
- Internal member rotation is org-internal and immediate; it requires no governance action.

### 2.1.2 Approval and cancellation rules (both tiers)

These rules are fixed by the FIP and restated here for operators.

> FIP-0118, on the change lifecycle:
>
> > "Every discretionary change follows the same path: both Safes approve, the change is published, and either Safe can cancel."

**Mechanism-executed updates** take no approval and are not cancellable. The quarterly gate check and the `SetShares` recompute run permissionlessly through entry points that can emit only the value the mechanism computes; their integrity guard sits upstream in the FPV verification window. The tiers exercise no judgment over these updates because there is no judgment to exercise.

> FIP-0118 on why mechanism writes cannot be cancelled:
>
> > "Mechanism-executed updates cannot be cancelled: the gate's quarterly w2 write queues in f02 under SWA_TIMELOCK for visibility, but it is tagged as a mechanism write at queue time and CancelPending rejects it."

**Discretionary changes** are as follows:

- **Approval:** both of the tier's Safes. SWA writes additionally require a published and accepted FIP. Registry changes deliberately carry no per-change FIP requirement; the carve-out covers data within the rules, never code. An upgrade of either contract's code requires a FIP.
- **Cancellation:** SWA discretionary writes are published and held for SWA_TIMELOCK; either Safe alone can cancel during the hold. SRA registry changes bind at once: both Safes approve, the change emits an on-chain event and carries a mandatory published rationale in this repository. The one exception is an SRA code upgrade, which is held before it binds.

### 2.1.3 Summary of changes

Informative summary; the lifecycle table in the FIP is normative.

| Action | Hold | Enforced by | Cancellable by |
| :-- | :-- | :-- | :-- |
| Discretionary SWA write to f02 (alter, add, or remove a stream; re-point a Distribution) | `SWA_TIMELOCK`, 7 days (FIP-fixed) | f02 | Either SWA Safe |
| SWA internal change (gate parameters, code upgrade) | Internal timelock, equal to the f02 window (FIP-fixed) | SWA | Either SWA Safe via `veto(taskId)` |
| Cooperative Safe replacement (`ReplaceOwner`) | None; binds at once | SWA or SRA | Not cancellable |
| Quarterly gate step | 7-day queue, visibility only | f02 | No one (mechanism-executed) |
| Registry change (add/remove orchestrator, replace wallet, set admitted lists, set pricing, replace owner) | None; binds at once | SRA | Not cancellable |
| Registry / code upgrade | Requires an approved FIP; held for `SRA_UPGRADE_HOLD` (20,160 epochs / 7 days) before it binds | SRA | Either SRA Safe via `veto(taskId)` during the hold |
| CorrectVolume | None; bounded by the verification window | SRA | Not cancellable |
| SetShares, PostVolume, RegisterPairs | None; bounded by the window, the posting period, and the uniqueness check | SRA | Not cancellable |

> FIP-0118 fixes the `SWA_TIMELOCK` hold in L1:
>
> > "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

The full compromise and rotation procedures for both tiers are in §2.5, Safety and rotation playbook.

---

## 2.2 SWA Governance Tier — Tasks and Actions

> **How to read this subsection:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about.

SWA Governance is **Tier 1**: it governs the Stream Weights Actor (SWA) — the list of streams and the split among them. FIP-0118 fixes its discretionary powers, each of which additionally requires a published FIP:

> SWA Governance discretionary powers (FIP-0118): *"add a stream (RegisterStream) or remove one (RemoveStream)"*, *"alter a stream: rewrite its Weight record (SetWeightRecords) or re-route its Distribution"*, *"tune the gate parameters (SetGateParams)"*, and *"upgrade the SWA's code"*.

SWA Governance approves nothing routine: the w1 ramp and the volume gate are mechanism-executed. Every discretionary action writes to f02, requires a published and accepted FIP first, and is held for `SWA_TIMELOCK` (7 days) before it binds.

- **During every `SWA_TIMELOCK`:** monitor the f02 queue and the off-chain objection process, and act on a sustained objection within the expected response-time commitment of 7 days.
- **Per discretionary write:** confirm the backing FIP is published and accepted before approval, and verify the schedule envelope (the sum of weights at most 1 across the whole segment) before submission.
- **Continuous:** keep Safe rosters and internal-implementation disclosures in this repository current.

> **Standard SWA discretionary-write flow** (referenced by 2.2.1–2.2.4):
>
> 1. Draft the exact write and carry a FIP through to 'Accepted' (including Last Call). No write is submitted without an accepted FIP.
> 2. Pre-submission check: verify the schedule envelope (Σw ≤ 1 across the whole segment) and that the write matches the accepted FIP.
> 3. Both SWA Safes approve `ProposeWrite(write)`. The first Safe’s transaction only records its approval (`UnanimousGovernance`); the action body runs in the second Safe’s transaction. The second Safe signs only if a dry-run simulation of **its own** approval succeeds, and notes the simulation result on the task-register issue. On a simulated revert, either Safe `veto`s and the action is resubmitted (a failed second approval leaves the task half-approved and burns the signature round).
> 4. The SWA relays to f02; f02 queues it with an `effective_epoch` and holds it for `SWA_TIMELOCK` (7 days).
> 5. Objection window: the queued write is public; monitor the off-chain objection process. Either SWA Safe may `cancelPending` / `cancelPendingWeight` / `veto(taskId)` (as applies) on a mismatch, a missing FIP, or a sustained objection; absent a cancellation it binds at `effective_epoch`.
> 6. Record the outcome in this repository (and in the [Program Change Log](06-changelog.md).

The L1 envelope check restates a FIP invariant:

> "f02 requires Σ_{i>=1} ComputeWeight(W_i, e) ≤ 1 at every epoch, so the burn residual w0 stays ≥ 0."

<details>
<summary><strong>2.2.1 — Alter a stream: rewrite a Weight record (SetWeightRecords)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow above; the call is `SetWeightRecords`. Reducing a weight to zero is held exactly like removing the stream. A discretionary write that changes the w2 Weight record must be paired with the matching `steps` adjustment via `SetGateParams` (see §2.2.5 for sequencing and recovery).

</details>

<details>
<summary><strong>2.2.2 — Add a stream (RegisterStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RegisterStream(id, weight_record, distribution, activation_epoch)`. Step 2 also confirms the stream and recipient caps and that `activation_epoch ≥ current_epoch + SWA_TIMELOCK`.

</details>

<details>
<summary><strong>2.2.3 — Remove a stream (RemoveStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RemoveStream(id)`. The removed weight reverts to the burn residual; removal creates no free value.

</details>

<details>
<summary><strong>2.2.4 — Re-point a stream's Distribution (SetDistribution)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `SetDistribution(id, distribution)`. Can be used, for instance, to re-point the service stream's designated writer to a redeployed SRA (such as the SRA-deadlock backstop). The current wallet-to-share map stays in force until the new writer overwrites it, so payments continue across the change.

</details>

<details>
<summary><strong>2.2.5 — Tune the gate parameters (SetGateParams)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

1. Publish and get an accepted FIP for the new gate parameters (`base`, `stepRatio`, and `steps` — not a free “step size”; the 5pp step itself is a contract constant).
2. Both SWA Safes approve `SetGateParams(params)`. A discretionary write that changes the w2 Weight record must go in the same governance action as the matching `steps` adjustment via `SetGateParams`. Per FIP-0118 §3.1.1, sequence the pair so that no `QuarterlyGateCheck` applies between the two effective epochs; the SWA enforces this only when the two holds overlap.
3. The change queues in the SWA's own state under the SWA-internal timelock (7 days). The gate rule itself is code and cannot change via a parameter write. Either Safe may cancel during the hold.
4. If one half of a paired write is cancelled after the other has applied, send a new paired write so w2 and `steps` do not stay mismatched.
5. Record in this repository.

</details>

<details>
<summary><strong>2.2.6 — Upgrade the SWA's code</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

Same flow as 2.2.5; the FIP carries the upgrade. It executes through the pre-upgrade code, so the hold and checks still bind.

</details>

<details>
<summary><strong>2.2.7 — Replace an SWA Safe (cooperative)</strong></summary>

> **Requires:** both SWA Safes · **Hold:** none · **Enforced by:** SWA · **Cancel:** not cancellable

1. Veto every open task in the SWA register first. `removeOwner` does not clear pending approvals; a freed owner bit can recycle to a future owner still carrying old approvals (`Owners.sol`).
2. Both SWA Safes approve replacing the registered Safe address (`ReplaceOwner`), with the action’s calldata and `taskId` recorded in the issue before the first approval (see task register above).
3. It binds at once when the second Safe approves (FIP: cooperative rotation is not held).
4. Announce here with a post-mortem.

The SWA-internal timelock covers gate-parameter changes and code upgrades only — not cooperative Safe replacement.

Hostile/deadlocked case (a Safe blocks its own replacement): the exit is one level up — a coordinated network upgrade migrates the SWA address in f02, always under a published FIP. See §2.5, Safety and rotation playbook.

</details>

<details>
<summary><strong>2.2.8 — Cancel a pending SWA write</strong></summary>

> **Requires:** either SWA Safe alone · **When:** during the hold/window

Either Safe can stop a bad SWA change on the matching path:

1. `cancelPending(id, op)` — queued stream operation on f02 (relayed to `f02.CancelPending`).
2. `cancelPendingWeight(op)` — queued weight write on f02.
3. `veto(taskId)` — a task still inside the SWA (half-approved, or in the SWA-internal hold).

A cancellation only preserves the status quo. Mechanism writes (the gate step via `StepWeightRecords`) are tagged at queue time and cannot be cancelled.

</details>

<details>
<summary><strong>2.2.9 — Monitor mechanism-executed updates</strong></summary>

The w1 ramp and the quarterly gate step (`QuarterlyGateCheck` → `StepWeightRecords`) run permissionlessly. Each quarter:

1. Confirm the queued w2 write matches the gate rule and the bound `AggregatedFPV(Q)`.
2. Note it is a scheduled write: it queues 7 days for visibility only and is not cancellable.
3. If a mismatch suggests compromise, escalate per §2.5, Safety and rotation playbook; the remedy will be a Safe/code replacement, not cancelling the mechanism write.

</details>

<details>
<summary><strong>2.2.10 — Continuous upkeep</strong></summary>

During every `SWA_TIMELOCK`, monitor the f02 queue and the objection process and act within the published response-time commitment. Keep Safe rosters and internal-implementation disclosures current.

</details>

---

## 2.3 SRA Governance Tier — Tasks and Actions

> **How to read this subsection:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about. The [Orchestrator Registry](#2312-orchestrator-registry-admitted-orchestrators) sits at the bottom (2.3.12).

SRA Governance is **Tier 2**: it governs the Service Rewards Actor (SRA) — the orchestrator registry and the split within the service stream. FIP-0118 fixes its discretionary powers; registry changes require no per-change FIP except a code upgrade:

> SRA Governance discretionary powers (FIP-0118): *"admit, remove, replace wallet, reassign or replace owner or replace an Orchestrator"*, *"correct or supply posted volumes during the verification window"*, *"maintain the admitted lists that scope FPV"*, and *"upgrade the SRA's code, the one registry change that requires a FIP"*.

Registry changes need both Registry Safes but need no per-change FIP (the one exception is a code upgrade). The quarterly verification runs against a public record and is deterministic.

> **Standard registry-change flow** (referenced by 2.3.1–2.3.5, 2.3.9–2.3.10):
>
> 1. Trigger and diligence (application, dispute outcome, audit finding, rotation request, or list update).
> 2. Both Registry Safes approve the relevant call. The first Safe’s transaction only records its approval (`UnanimousGovernance`); the action body runs in the second Safe’s transaction (SRA header: the second Safe MUST dry-run the calldata before approving). The second Safe signs only if a dry-run simulation of **its own** approval succeeds, and notes the simulation result on the task-register issue. On a simulated revert (wrong wallet, closed window, pending shares, etc.), either Safe `veto`s and the action is resubmitted.
> 3. The change binds when the second SRA Safe approves (no pending queue, no cancellation path). Record it in the Change Log.
> 4. Either Registry Safe have visibility on changes.
> 5. Record the outcome in the issue/repository, and update the [Orchestrator Registry](#2312-orchestrator-registry-admitted-orchestrators) where the change affects an Orchestrator's status.

The recomputation is deterministic: any observer running the reference indexer over the same settlement events and registry state reaches the same figure.

### Admission (Phase 1: discretionary)

An Orchestrator's application is filed as an issue in this repository. If application is approved, SRA Governance executes the admit on-chain (action 2.3.1). **Every admitted Orchestrator is recorded in the [Orchestrator Registry (2.3.12)](#2312-orchestrator-registry-admitted-orchestrators).**

A uniqueness rule is fixed by the FIP:

> "Each (payer, operator) pair is bound to at most one orchestrator, so a registration that duplicates an existing binding reverts."

For Phase 2 (subject to a future FIP): admission becomes permissionless, enabled by a standard attribution-metadata interface specified in a future FIP. The rubric and diligence apply to Phase 1 only.

<details>
<summary><strong>2.3.1 — Admit an Orchestrator (AddOrchestrator)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

1. Application filed as an issue (identity/team, funding plan, declared (payer, operator) pairs and measurement rules).
2. SRA Governance scores it against the admission rubric. **Admission checklist — payout wallet:** resolve the proposed wallet to its actor ID and confirm the actor code is **not** a payment channel. Since solstice #78 the SRA rejects a wallet with no actor, but payment channels can still pass `_assertWalletAdmissible` and then fail in `SetShares` / `ReplaceAddress` (`ServiceRewardsActor.sol`).
3. Both Registry Safes approve `AddOrchestrator(orch, wallet)` using the task-register issue’s canonical calldata; the uniqueness rule reverts any pair already bound elsewhere.
4. Either Safe may cancel (veto while half-approved).
5. Binds; record in the issue **and add the Orchestrator to the [Orchestrator Registry (2.3.12)](#2312-orchestrator-registry-admitted-orchestrators)**.

</details>

<details>
<summary><strong>2.3.2 — Remove an Orchestrator (RemoveOrchestrator)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

Remove is permanent. It **releases** the Orchestrator's (payer, operator) bindings, f099 repoint (future income burns), accrued stays claimable, releases bindings, freeing those pairs for re-registration. Update the Orchestrator Registry to mark the entry *Removed*.

</details>

<details>
<summary><strong>2.3.3 — Replace / rotate an Orchestrator address (ReplaceWallet)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

Standard registry-change flow; the call is `ReplaceWallet(old, new)`. Rotates a compromised or non-functioning **payout wallet**; per FIP-0118 §3.2, `ReplaceWallet` swaps the payout wallet through an immediate `ReplaceAddress` call, so f02 pays the new wallet at once. Update the payout-wallet field in the Orchestrator Registry.

**Payment-channel wallet recovery.** A payout wallet that is a payment channel can pass admission’s actor-exists check but breaks `SubmitShares` / `SetShares` for **every** Orchestrator (Rod’s devnet). `RemoveOrchestrator` also reverts while a quarter’s share map is still pending (`PendingShares`). Exit order:

1. `ReplaceWallet` to a non-payment-channel wallet (and re-check actor code).
2. `SubmitShares` for the pending quarter (now that every wallet in the map is admissible).
3. `RemoveOrchestrator` if the Orchestrator is being exited (only after shares are no longer pending).

</details>

<details>
<summary><strong>2.3.4 — Quarterly FPV verification & correction (CorrectVolume)</strong></summary>

> **Requires:** both SRA Safes (jointly) · **Hold:** none (the verification window is the hold) · **Enforced by:** SRA · **Cancel:** not cancellable

1. Recompute each posted FPV_i(Q) from public settlement events + registry state, using the versioned reference indexer.
2. Publish the recomputation.
3. During the verification window, both Safes jointly call `CorrectVolume(orch, Q, value)` to replace a misreport or supply a figure for a non-poster. A value neither posted nor supplied binds at zero (conservative under-count). Open the task-register issue (calldata + `taskId`) before the first approval.
4. **Internal dual-approval deadline.** The window check runs in the function body after approvals are stored (`ServiceRewardsActor.correctVolume`), so a second approval after the window closes reverts while the first approval stays. Aim for both approvals by day 5 of the 7-day mainnet window (scale the same fraction on calibnet). If the window closes without the second approval, the first approver `veto`s the stale task.
5. Values bind when the window closes; there is no separate cancellation of a completed correction.
6. A contested correction is appealed via the [dispute process](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings); the outcome affects only later quarters.

FIP-0118 fixes the binding behavior:

> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter."
>
> "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."

</details>

<details>
<summary><strong>2.3.5 — Dispute resolution for contested bindings</strong></summary>

1. The contesting party opens an issue identifying the pair and its claim.
2. SRA Governance requests evidence from both parties (increasing strength: business records; client confirmation via a verifiable channel; a signed payer-wallet attestation naming the orchestrator).
3. Decide the outcome. Removal executes on-chain via the standard registry-change flow (2.3.1–2.3.4).
4. Record the resolution in the issue.

Full procedure and evidence standards: [§4.4.2](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings).

</details>

<details>
<summary><strong>2.3.6 — Exception-based verification (removal)</strong></summary>

1. Trigger: an anomaly in public settlement data, and/or monitoring dashboards, and/or a community report. Audits are never initiated as pre-verification.
2. Investigate: recompute FPV and examine the (payer, operator) relationships for wash trading, self-dealing, misreported FPV, or binding fraud.
3. Decide grounds.
4. Act: `RemoveOrchestrator)` (2.3.2) via the standard registry-change flow, subject to approval/cancellation.
5. Record the finding and rationale.

</details>

<details>
<summary><strong>2.3.7 — Maintain admitted lists & fee-auction parameters (SetAdmittedLists, pricing settings)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

Standard registry-change flow for the admitted-stablecoin whitelist, the admitted Filecoin Pay contract addresses, and the fee-auction pricing parameters (`MIN_LOT_FLOOR`, `MIN_LOT_ALPHA`, `PRICE_BAND`, `REGISTRATION_CUTOFF`). These are gate-consequential. Registry actions bind at once when the second Safe approves — not held, not cancellable. Either Safe can `veto(taskId)` on a half-approved task before the second approval. Never a silent edit. For `SetAdmittedLists`, sort each address array ascending before encoding so both Safes share one `taskId` (see task-register canonical calldata).

</details>

<details>
<summary><strong>2.3.8 — Upgrade the SRA's code</strong></summary>

> **Requires:** both SRA Safes + accepted FIP · **Hold:** `SRA_UPGRADE_HOLD`, 7 days · **Enforced by:** SRA · **Cancel:** either SRA Safe via `veto(taskId)` during the hold

The one registry change that requires a FIP (e.g., the Phase 2 permissionless-admission transition). Otherwise follows the standard flow.

</details>

<details>
<summary><strong>2.3.9 — Replace an SRA Safe (cooperative)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable

1. Veto every open task in the SRA register first. `removeOwner` does not clear pending approvals; a freed owner bit can recycle to a future owner still carrying old approvals (`Owners.sol`).
2. Both Safes approve the replacement (`ReplaceOwner`), with the action’s calldata and `taskId` recorded in the issue before the first approval (see task register above). It binds at once when the second Safe approves and is announced with a post-mortem. Not held and not cancellable.

Hostile/deadlocked case: a SWA write re-points the service stream's writer to a redeployed SRA, always under a published FIP; registry state is reconstructible from public data. See §2.5, Safety and rotation playbook.

</details>

<details>
<summary><strong>2.3.10 — Monitor mechanism-executed updates (oversight, no approval)</strong></summary>

`SubmitShares(Q)` runs permissionlessly and is not held (FPV is posted already USD-denominated; there is no separate on-chain conversion pass). Duty: confirm `SubmitShares` has run and wrote the correct wallet-to-share map (integrity rests on the upstream verification window). It is not cancellable.

</details>

<details>
<summary><strong>2.3.11 — Continuous upkeep</strong></summary>

Maintain and version the reference indexer; monitor anomaly reports, contested bindings, and the cancellation queue; keep Safe rosters, disclosures, and the Orchestrator Registry current.

</details>

### 2.3.12 Orchestrator Registry (admitted Orchestrators)

This is the canonical, human-readable list of Orchestrators admitted to the Solstice program. It is maintained by SRA Governance and mirrors on-chain SRA registry state — **the on-chain registry is the source of truth**; this list is a convenience view and audit trail. Entries are added and updated only through the SRA registry actions above: an Orchestrator appears on Admit (2.3.1), has its wallet updated on Replace (2.3.3), and is marked *Removed* on Remove (2.3.2).

**Initial Orchestrator at activation.** FIP-0118 seats an Orchestrator at activation and requires it to “publish a disclosure on activation, following the requirements in the governance repository set for any other Orchestrator.” That Orchestrator is not admitted through an Admit action (2.3.1). On activation it publishes the same declaration file required of any other Orchestrator ([§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies)), and SRA Governance sets its registry row below to **Active** with admission = quarter 1 (identity and payout wallet `0x97A90f5696be5E3C8d3752C92Adac287c2b4484e`, matching `initialOrchestrator` / `initialOrchestratorWallet` in [`deployments.json`](https://github.com/filecoin-project/solstice/blob/main/deployments.json)).

**Status legend**

| Status | Meaning |
| :-- | :-- |
| **Active** | Admitted and operating; can `RegisterPairs`/`PostVolume`; FPV counts toward `SplitRule` and `AggregatedFPV`. |
| **Removed** | Permanently removed; (payer, operator) bindings released for re-registration (2.3.2). |

**Admitted Orchestrators**

> **Program status:** the activation seat is recorded below; further Orchestrators appear when SRA Governance executes an `Admit` action (2.3.1). The table shows the columns each entry must carry.

| # | Orchestrator | Identity (`orch`) | Payout wallet | Status | Admitted (quarter / epoch) | Declaration | Registered (payer, operator) pairs |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | Initial (activation seat) | 0x97A90f5696be5E3C8d3752C92Adac287c2b4484e | 0x97A90f5696be5E3C8d3752C92Adac287c2b4484e | Active | quarter 1 | TBD (Policy 6 declaration on activation) | — |

**Per-Orchestrator entry template**

When admitting an Orchestrator, add a row to the table above and a detailed entry using this template:

```markdown
### <Orchestrator name>

- **Identity (`orch`):** <f4… / 0x… address>
- **Payout wallet:** <f4… / 0x… address> (may differ from identity)
- **Status:** Active | Removed
- **Admitted:** <quarter, epoch, tx hash>
- **Declaration issue:** #<issue number> (link)
- **Contact:** <public contact per the §3.1 responsiveness rule>
- **Registered (payer, operator) pairs:** <pointer to on-chain registry / list>
- **Measurement rules:** <short description or link to service-contract metadata>
- **History:** <admit / replace / remove events, each with date + tx>
```

> Field definitions follow the Orchestrator guidelines: identity and payout wallet in [§3.2](03-orchestrator-operational-guidelines.md#32-orchestrators-tasks-and-actions), the declaration file in [§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies), and the (payer, operator) binding rules in [§3.1, Policy 3](03-orchestrator-operational-guidelines.md#31-policies) (FIP-fixed uniqueness).

---

## 2.4 Parameters

### 2.4.1 Safe Addresses

Each tier consists of two organization multisigs (Safes) registered in the contract it governs. The protocol sees only the two addresses.

| Tier | Contract governed | Organization 1 Safe (address) | Organisation 2 Safe (address) | Rule (fixed by the FIP) |
| :-- | :-- | :-- | :-- | :-- |
| SWA Governance (§2.2) | Stream Weights Actor (SWA) | 0x4d6db5600c908b3C9b9888cFef43f27F38b203e9 | 0x024a3c8CCA435db64D2dfa0f903E4823A5eBdf63 | Both approve; either alone cancels |
| SRA Governance (§2.3) | Service Rewards Actor (SRA) | 0x6c724FF811f51945d95872CF99FBd3fF63f7cFa2 | 0xFb1B58925947E52B3f75BAc3D9fB5325cfb36371 | Both approve; binds at once (not cancellable; SRA code upgrades held, either Safe may veto during the hold) |

Each organization runs a separate Safe per tier — four accounts in total — so approvals cannot be replayed across surfaces.

Addresses match [`filecoin-project/solstice` `deployments.json`](https://github.com/filecoin-project/solstice/blob/main/deployments.json) for Filecoin mainnet (`314`): Organization 1 = `*Owner2`, Organisation 2 = `*Owner1`. The SRA row’s cancel rule follows §2.1.2 (registry changes bind at once; only an SRA code upgrade is held and cancellable).

> The approval rule is fixed by FIP-0118:
>
> > "Both Safes approve, the change is published and held."

### 2.4.2 Governed Parameters

| Parameter | Where set | Value |
| :-- | :-- | :-- |
| `SWA_TIMELOCK` (objection window on SWA writes to f02) | FIP-fixed, enforced in f02 | 7 days |
| SWA-internal timelock (own state and code) | FIP-fixed, equal to the f02 window | 7 days |
| `POST_PERIOD` (FPV posting) | FIP-fixed at SRA deployment (code upgrade to change) | 8,640 epochs (3 days) |
| `VERIFICATION_WINDOW` (FPV verification) | FIP-fixed at SRA deployment (code upgrade to change) | 20,160 epochs (7 days) |
| `EPOCHS_PER_QUARTER` | FIP-fixed at SRA deployment (code upgrade to change) | 262,974 epochs |
| Quarter boundaries | Counted from `ACTIVATION_EPOCH` | Not calendar quarters |
| Dispute resolution target ([§4.4.2](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings)) | This repository | 7 days |
| Orchestrator response window ([§4.4.5](04-quarterly-review-and-runbook.md#445-escalation)) | This repository | 7 days |
| Admitted-stablecoin whitelist | This repository | TBD |
| Fee-auction pricing parameters (`MIN_LOT_FLOOR`, `MIN_LOT_ALPHA`, `PRICE_BAND`, `REGISTRATION_CUTOFF`) | SRA parameter events (next-quarter boundary) | See FIP initial values |
| Admission rubric | This repository | TBD, after the first application cycle |

> `SWA_TIMELOCK` is FIP-fixed and enforced in L1:
>
> > "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

---

## 2.5 Safety and rotation playbook (both tiers)

The threat model rests on the two-Safes rule: no single Safe can make a change bind, and either Safe can cancel a pending change during the hold. FIP-0118 fixes the L1 backstop that makes even a fully compromised Safe unable to bind early:

> "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

### 2.5.1 Key compromise inside a Safe (below the internal threshold)

1. The organization reports publicly in this repository.
2. Any pending change the Safe approved while the key was compromised is reviewed; if suspect, either Safe cancels it.
3. The organization rotates its internal membership (org-internal, immediate, no governance action) and announces the rotation here.

### 2.5.2 A whole Safe compromised (internal threshold reached by an attacker)

1. Nothing binds from one Safe alone. A hold (where one exists) starts only after both Safes approve. One Safe can only submit a task; the `Submitted` event carries the `taskId` (a hash), not the full content. The other Safe’s remedy is `veto(taskId)` on that half-approved or held task. Repeated resubmission is publicly visible; the attacker cannot bind a change silently. On the SWA side, a malicious discretionary write also lacks its required published FIP, making the objection case unambiguous.
2. **Task register (off-chain).** Pending tasks never expire on-chain (`UnanimousGovernance`), and `Submitted` names only the `taskId` hash. Before the first on-chain approval of any governance action, open an issue in this repository that records the exact calldata and the expected `taskId` (= `keccak256(msg.data)`). That issue is the human-readable register entry the hash alone cannot provide. **Canonical calldata:** both Safes copy the calldata from that issue verbatim. Address lists in the payload (e.g. `SetAdmittedLists`) are sorted ascending by address before encoding — the same list in a different order is a different `msg.data` and a different task that never reaches unanimity. The second Safe also records its pre-approval dry-run result on that issue (see the standard flows in §2.2 and §2.3).
3. The tier is treated as frozen (halted/suspended), and frozen consequences are bounded: registry frozen means payments and `SetShares` continue; SWA frozen means discretionary changes stop while the ramp and the gate continue through the permissionless crank.
4. Exit: replace the compromised Safe address. This requires both Safes, so if the compromised Safe obstructs, the FIP backstop applies (2.5.3).

### 2.5.3 Replacing a registered Safe

**Cooperative case:** both Safes approve the replacement; it binds at once and is not cancellable; announced here with a post-mortem. See 2.2.7 (SWA) and 2.3.9 (SRA).

**Hostile or deadlocked case, always under a published FIP:** for SRA Governance, a SWA write re-points the service stream's designated writer to a redeployed SRA; registry state is reconstructible from public data, and no network upgrade is needed. For SWA Governance, a coordinated network upgrade migrates the SWA address in f02.

> **Open question:** whether any fast path short of a FIP is acceptable, and what covers simultaneous compromise of both tiers.

---

← [Back to README](../README.md) · Next: [3. Orchestrator Operational Guidelines](03-orchestrator-operational-guidelines.md) →