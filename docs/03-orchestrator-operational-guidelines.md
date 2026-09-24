# 3. Orchestrator Operational Guidelines

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) and referenced throughout.

This section consolidates what is expected of Orchestrators: the operational guidelines, policies, tasks, and monitoring tools. Declaration, verification and escalation now lives in [§4.4](04-quarterly-review-and-runbook.md#44-declaration-verification-and-escalation).

**Contents**

- [3.1 Policies](#31-policies)
- [3.2 Orchestrators Tasks and Actions](#32-orchestrators-tasks-and-actions)
- [3.3 Monitoring tools & references](#33-monitoring-tools--references)
- [3.4 Solstice Orchestrator Task Checklist](#34-solstice-orchestrator-task-checklist)

The service stream pays for measured, paid storage service. These rules give orchestrators predictability about what is expected and what the consequences of a breach are, and give the network confidence that the number driving the gate and the shares reflects reality. The standing test for any action in the program: **does this volume represent a client paying for storage service on Filecoin?** If the answer is yes, the orchestrator is operating within both the rules and their intent.

Orchestrators are free to run their businesses autonomously. The program does not review their pricing, their client selection, or their operations. To maintain trust in the measurement layer, all orchestrators are expected to adhere to the criteria below. These may be revised over time through the standard PR process (and logged in the [Program Change Log](06-changelog.md)).

- **Measurable settlement.** Orchestrators route paid storage service revenue through admitted Filecoin Pay contracts in admitted stablecoins or FIL, and help their clients settle the same way. Volume that bypasses admitted rails is invisible to the mechanism and counts for nothing.
- **Responsiveness.** Orchestrators respond to dispute and verification requests within the response window (7 days) and keep their contact information in this repository current.
- **Good-faith declarations.** Declarations are accepted by default. The program extends trust upfront because every posted figure is recomputable from public events; misreporting is mechanically detectable and is grounds for removal.
- **Self-monitoring.** Orchestrators are expected to monitor their own recomputed FPV against posted figures during each verification window, and to flag discrepancies rather than wait for them to surface as findings.

## 3.1 Policies

Each policy notes where it is enforced:

1. **FIP** — protocol invariant, changes require a FIP;
2. **Contract** — enforced by SRA or f02 code;
3. **Repository** — program rule, changes by PR.

| # | Policy | Enforced |
| :-- | :-- | :-- |
| 1 | The program is open to any entity that routes paid storage service revenue through admitted settlement rails on behalf of clients. Admission is discretionary in Phase 1; a future FIP makes admission permissionless in Phase 2. | Repository |
| 2 | Qualifying volume is settlement through admitted Filecoin Pay contracts, in admitted stablecoins or FIL converted off-chain via the reference indexer using public fee-auction prints (`MIN_LOT`, `PRICE_BAND`), attributable to the orchestrator's registered (payer, operator) pairs. Nothing else counts. | Contract |
| 3 | A (payer, operator) pair binds to exactly one orchestrator. Registering a pair already bound elsewhere reverts. Clients are free to work with multiple orchestrators across different operator relationships; the pair, not the client, is the unit of attribution. | FIP |
| 4 | Volume counts only for pairs registered before the settlement occurs. Retroactive attribution is not accepted. | Repository |
| 5 | Each quarter, orchestrators post FPV within `POST_PERIOD` when it is above zero. `PostVolume` rejects zero (a zero total is equivalent to not posting); zero is not posted and binds at zero. A value neither posted nor corrected during `VERIFICATION_WINDOW` likewise binds at zero. Bound values are final; a successful appeal affects later quarters only. | FIP and contract |
| 6 | Within 7 days of admission, each orchestrator publishes its declaration file in this repository: a short description of its service, contact information, and its registered pairs, including any pointer to service-contract metadata. | Repository |
| 7 | An orchestrator with no settled volume for **[TBD timeline]** consecutive quarters enters registry review and may be removed. Removal on inactivity is registry hygiene: an idle registration adds attack surface without adding measurement value. | Repository |
| 8 | No registry address participates in either governance tier, as a Safe or as a key holder within one. | FIP |
| 9 | Quarterly Community Report on claims and reward reception, failure to disclose is a removal trigger and forfeits the next quarter re-admission | FIP and Repository |

Policies 3, 5, and 8 restate FIP-0118 invariants verbatim:

> "Each (payer, operator) pair is bound to at most one orchestrator, so a registration that duplicates an existing binding reverts."
>
> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter." … "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."
>
> "No address in the registry may participate in either contract governance tier, as a Safe or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

## 3.2 Orchestrators Tasks and Actions

> These are operational tasks and actions; each is a collapsible dropdown, and they may be updated by raising an issue in the present repository.
> <img width="1600" height="897" alt="image" src="https://github.com/user-attachments/assets/c1750749-9809-452d-9024-7fc904896188" />



<details>
<summary><strong>3.2.1 — Deal making</strong></summary>

> **Cadence:** continuous · **On-chain call:** none

Orchestrators set their own pricing, choose their own clients, and structure their deals, subsidies, and operations as they see fit. The program does not review or approve commercial terms, client selection, or operations. The only constraint is measurability: paid storage-service revenue must settle through admitted Filecoin Pay rails on registered (payer, operator) pairs so it can be counted.

</details>

<details>
<summary><strong>3.2.2 — Orchestrator identity and payout wallet</strong></summary>

> **Cadence:** at admission + on a need basis (`ReplaceWallet`) · **On-chain call:** `ReplaceWallet(old, new)` (payout-wallet rotation)

Each Orchestrator has **two** on-chain addresses in the SRA registry (`AddOrchestrator(orch, wallet)`):

1. **Orchestrator identity (`orch`)** — calls `RegisterPairs` and `PostVolume`.
2. **Payout wallet (`wallet`)** — receives the service-stream share from f02. Must not be a payment-channel actor; f02 rejects payment channels as share recipients. Prefer a multisig (e.g., a Safe) with hardware-key signers. If the payout wallet is a Safe or other f410 / EVM-contract address, also name an **f1 claim keeper** (see §3.2.4).

They can differ. As part of admission (see [§2.3 SRA Governance Tier](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)), record both addresses in your GitHub declaration. Your entry appears in the [Orchestrator Registry](02-solstice-program-governance.md#2312-orchestrator-registry-admitted-orchestrators) with both columns.

Rotate a compromised **payout** wallet via `ReplaceWallet(old, new)`. Per FIP-0118 §3.2, that call swaps the payout wallet through an immediate `ReplaceAddress`, so f02 pays the new wallet at once.

</details>


<details>
<summary><strong>3.2.3 — Linking deals to Filecoin Pay services</strong></summary>

> **Cadence:** per deal, before settlement · **On-chain call:** `RegisterPairs`

Volume counts only when it settles on an admitted Filecoin Pay contract, in an admitted stablecoin or FIL, on a rail whose (payer, operator) pair is registered to you *before* the settlement occurs. To link a deal:

1. Set up the client's payment on an admitted Filecoin Pay contract (admitted-contract list).
2. Create the rail with the operator you will register.
3. Register the (payer, operator) pair via `RegisterPairs` before the first settlement. Retroactive attribution is not accepted; a pair already bound elsewhere reverts (uniqueness invariant).
4. The client settles on-rail (`RailSettled` or one-time payments); those settlements accrue to your FPV.

> **Note:** volume on unregistered pairs, off admitted rails, or in non-admitted assets will not be accounted for in the FPV measurement.

</details>

<details>
<summary><strong>3.2.4 — Measurement and receiving funds</strong></summary>

> **Cadence:** measurement quarterly · payout each epoch · **On-chain call:** `PostVolume` (measurement)

1. Understand your figure: FPV_i(Q) is the value settled on your registered (payer, operator) rails during a given quarter; amounts are denominated in USD — admitted stablecoins at face value, FIL converted off-chain via the reference indexer using public fee-auction prints (`MIN_LOT`, `PRICE_BAND`).
2. Declare how your volume is measured (your pairs, optional service-contract metadata, booked-revenue basis).
3. Receiving service stream payouts: f02 accrues your **payout wallet** its share of the service stream (w2) every epoch. Entitlements are withdrawn via the permissionless native `Claim` method. An f410 (including a Safe) or an EVM-contract wallet cannot send that call; an **f1 keeper account** must. Name the keeper as a required part of setup in your declaration (alongside identity and payout wallet). Prefer keeping the Safe as the payout wallet for custody and using the f1 only to `Claim`.

The following rule establishes how the share is set: `SplitRule` = your bound FPV_i(Q) ÷ AggregatedFPV(Q), written into f02 once per quarter via `SetShares`.

If your bound FPV for the quarter is zero while another Orchestrator has bound volume, your share for that quarter is zero and your wallet is never debited. If no Orchestrator has bound volume (eligible FPV sums to zero), `SubmitShares` is a benign no-op: the existing share map stands.

</details>

<details>
<summary><strong>3.2.5 — Treasury function and wallet-safety best practices</strong></summary>

> **Cadence:** continuous · **On-chain call:** none (off-protocol)

Rewards land in your payout wallet each epoch; deploying them against your mandate (client acquisition, subsidies, integrations, migrations, infrastructure, etc.) is your responsibility and lies outside the protocol. Recommended practices:

- **Custody:** hold reserves in a multisig (e.g., Safe) with hardware-key signers; never keep the treasury on a single hot key.
- **Segregation:** keep a cold reserve (offline/hardware, high signing threshold) separate from a small hot operating wallet; sweep excess from hot to cold.
- **Key hygiene:** never expose or share private keys or seed phrases, and never place them in code, configs, or shared documents; if exposure is even suspected, rotate immediately (`Replace`).
- **Least privilege:** where you use exchanges or custodial APIs, prefer read-only keys and disable withdrawal/trading scopes you don't need.
- **Monitoring & recovery:** alert on inflows/outflows, name an accountable key-custody owner, and keep a documented, tested recovery path.
- **Conversion discipline:** convert only what runway requires; hold the reserve.

</details>

<details>
<summary><strong>3.2.6 — Reporting and reconciliation</strong></summary>

> **Cadence:** quarterly (`POST_PERIOD`, then `VERIFICATION_WINDOW`) · **On-chain call:** `PostVolume`

1. Within `POST_PERIOD`, if your FPV is above zero, post it via `PostVolume` as a single USD-denominated total, with FIL volume converted off-chain by the indexer per the fee-auction pricing rule. If the quarter’s FPV is zero, do **not** call `PostVolume` (the contract rejects zero); the value binds at zero the same as a non-post.
2. During `VERIFICATION_WINDOW`, recompute your own FPV from public settlement events using the reference indexer and reconcile it against what you posted (or against zero if you did not post); flag any discrepancy proactively.
3. Keep your declaration file (pairs, contact, measurement rules) current.
4. Remember the failure mode: a value neither posted nor corrected binds at zero (a conservative under-count); bound values are final, and a successful appeal affects later quarters only.

FIP-0118 fixes the posting period and the no-clawback rule:

> "Posting: each Orchestrator posts FPV_i(Q) to the SRA during the posting period (E, E + POST_PERIOD]."
>
> "There is no retroactive clawback: a misreport discovered after binding is grounds for audit or removal."

</details>

<details>
<summary><strong>3.2.7 — Declarations, verification, and appeal</strong></summary>

> **Cadence:** as triggered · **On-chain call:** none (governance-side `CorrectVolume` / `ReassignBinding`)

Declarations are accepted by default; verification is exception-based and dispute-driven, never pre-clearance. If a binding is contested or a figure disputed, the process runs through the dispute resolution process in [§4.4.2](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings). Appeals of a `CorrectVolume` follow the same evidence process and can affect only later quarters, since bound values are final.

</details>

## 3.3 Monitoring tools & references

*(links to be updated)*

These tools are meant to ease the monitoring of the program's activity. Orchestrators use them to check their recomputed FPV against what they posted; SRA Governance and any community observer runs the same tools for oversight. Every verification action and its conclusion must be reproducible from these public sources.

- Versioned reference indexer: *(links to be added)*
- Governance & signing interface: *(links to be added)*
- Settlement data and dashboards: *(links to be added)*
- Registry state, admitted-stablecoin whitelist, and admitted orchestrators: see the [Orchestrator Registry](02-solstice-program-governance.md#2312-orchestrator-registry-admitted-orchestrators) *(on-chain links to be added)*
- Filecoin Pay contracts: *(links to be added)*

## 3.4 Solstice Orchestrator Task Checklist
Use this checklist to track completion of the core Solstice Orchestrator responsibilities.


| Reference                           | Link                                                                                                                                |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Orchestrator Operational Guidelines | [View Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| Solstice Program Governance         | [View Governance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md)         |
| Quarterly Review & Runbook          | [View Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| Solstice Governance Repository      | [View Repository](https://github.com/filecoin-project/Solstice-Governance)                                                          |

---
<summary><strong>3.4.1 — Every New Client / Deal </strong></summary>

Complete before counting FPV from a new payer/operator relationship.

| Done | Task                                                       | Timing                         | Reference                                                                                                                                    |
| ---- | ---------------------------------------------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Confirm there is a genuine paying Filecoin storage client  | Before onboarding              | [Qualifying Volume Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)  |
| [ ]  | Confirm payer and operator identities                      | Before registration            | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Check payer/operator relationship for common control       | Before registration            | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Check for potential self-dealing                           | Before registration            | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Confirm payment will use an admitted Filecoin Pay contract | Before settlement              | [Settlement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)         |
| [ ]  | Confirm payment asset is admitted                          | Before settlement              | [Current Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters) |
| [ ]  | Establish payment rail between payer and operator          | Before settlement              | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Check whether payer/operator pair is already registered    | Before registration            | [Governance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md)                       |
| [ ]  | Register payer/operator pair using `RegisterPairs`         | **Before settlement**          | [Pair Registration Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)  |
| [ ]  | Confirm registration succeeded on-chain                    | Immediately after registration | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Confirm payments are settling through the registered rail  | After setup                    | [Settlement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)         |
| [ ]  | Retain evidence supporting the client relationship         | Ongoing                        | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)            |

> **Important:** Volume settled before the payer/operator pair is registered cannot be attributed retroactively.

---
<summary><strong>3.4.2 — Treasury & Wallet Management </strong></summary>

| Done | Task                                                                     | Cadence      | Reference                                                                                                                           |
| ---- | ------------------------------------------------------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Monitor service-stream reward accruals                                   | Ongoing      | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Claim rewards when operationally appropriate                             | As needed    | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)      |
| [ ]  | Track rewards received                                                   | Ongoing      | [Quarterly Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)      |
| [ ]  | Reconcile treasury inflows and outflows                                  | Regularly    | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Review wallet access and multisig signers                                | Regularly    | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Keep operational funds separated from protected reserves where practical | Ongoing      | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Maintain hardware-backed signing where practical                         | Ongoing      | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Maintain and test wallet recovery procedures                             | Periodically | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Monitor for unauthorized wallet activity                                 | Ongoing      | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Rotate exposed credentials immediately                                   | As needed    | [Wallet Guidance](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |

---
<summary><strong>3.4.3 — Quarter-End FPV Calculation </strong></summary>

| Done | Task                                                        | Timing              | Reference                                                                                                                                    |
| ---- | ----------------------------------------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Confirm the reporting quarter has ended                     | Quarter close       | [Timing Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters)  |
| [ ]  | Identify active registered payer/operator pairs             | Quarter close       | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Retrieve settlement activity for those pairs                | Quarter close       | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Exclude settlements occurring before pair registration      | Quarter close       | [Measurement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)        |
| [ ]  | Exclude settlements using non-admitted contracts            | Quarter close       | [Measurement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)        |
| [ ]  | Exclude settlements using non-admitted assets               | Quarter close       | [Current Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters) |
| [ ]  | Confirm remaining volume represents genuine client payments | Quarter close       | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Convert qualifying FIL volume using approved methodology    | Quarter close       | [Measurement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)        |
| [ ]  | Calculate total quarterly FPV                               | Quarter close       | [Measurement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)        |
| [ ]  | Reconcile FPV against internal records                      | Before posting      | [Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)                         |
| [ ]  | Reconcile FPV against public settlement activity            | Before posting      | [Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)                         |
| [ ]  | Resolve material discrepancies                              | Before posting      | [Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)                         |
| [ ]  | Finalize quarterly FPV                                      | Before `PostVolume` | [Measurement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)        |

---
<summary><strong>3.4.4 — Post Quarterly FPV </strong></summary>

| Done | Task                                      | Timing                    | Reference                                                                                                                                   |
| ---- | ----------------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Confirm `POST_PERIOD` is open             | Post period               | [Timing Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters) |
| [ ]  | Confirm final FPV amount                  | Before posting            | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)              |
| [ ]  | Submit FPV using `PostVolume` (skip if FPV is zero — contract rejects zero; binds at zero) | Post period               | [PostVolume Requirements](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Confirm transaction succeeded             | Immediately after posting | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)              |
| [ ]  | Confirm posted FPV matches calculated FPV | Immediately after posting | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)              |
| [ ]  | Save posting transaction reference        | Immediately after posting | —                                                                                                                                           |

---
<summary><strong>3.4.5 — Verification Window </strong></summary>

| Done | Task                                                     | Timing               | Reference                                                                                                                                   |
| ---- | -------------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Confirm verification window is open                      | Verification window  | [Timing Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters) |
| [ ]  | Recompute FPV using the reference indexer when available | Verification window  | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Compare recomputed FPV against posted FPV                | Verification window  | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Review registered-pair state used in calculation         | Verification window  | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Review FIL conversion calculations where applicable      | Verification window  | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Investigate discrepancies                                | Immediately          | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Proactively report material discrepancies                | Immediately          | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Monitor GitHub for verification requests                 | Verification window  | [GitHub Issues](https://github.com/filecoin-project/Solstice-Governance/issues)                                                             |
| [ ]  | Respond to verification requests                         | Within 7 days        | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Complete required corrections                            | Before window closes | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |
| [ ]  | Confirm final FPV                                        | Before window closes | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |

---
<summary><strong>3.4.6 — Quarterly Community Report </strong></summary>

| Done | Task                                             | Timing                   | Reference                                                                                                                      |
| ---- | ------------------------------------------------ | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| [ ]  | Open current Quarterly Community Report template | Each quarter             | [Report Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)       |
| [ ]  | Prepare quarterly report                         | Each quarter             | [Reporting Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md) |
| [ ]  | Include Orchestrator identification              | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include claimed FPV                              | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include rewards received                         | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include required financial information           | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Explain use of service rewards                   | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include what worked and what did not             | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include key lessons learned                      | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include next-quarter outlook                     | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Include required disclosures                     | Each quarter             | [Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md)              |
| [ ]  | Publish report in required repository location   | Each quarter             | [Quarterly Reports](https://github.com/filecoin-project/Solstice-Governance/tree/main/docs/quarterly-reports)                  |
| [ ]  | Respond to questions or challenges               | Required response window | [Reporting Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md) |

---
<summary><strong>3.4.7 — Quarter Closeout </strong></summary>

| Done | Task                                                       |
| ---- | ---------------------------------------------------------- |
| [ ]  | Confirm final bound FPV                                    |
| [ ]  | Confirm resulting service-stream share                     |
| [ ]  | Confirm expected rewards are accruing correctly            |
| [ ]  | Archive FPV calculations                                   |
| [ ]  | Archive client-relationship evidence                       |
| [ ]  | Archive payment and settlement records                     |
| [ ]  | Archive treasury records                                   |
| [ ]  | Archive relevant governance correspondence                 |
| [ ]  | Confirm quarterly report is publicly available             |
| [ ]  | Confirm no unresolved governance requests remain           |
| [ ]  | Carry unresolved operational actions into the next quarter |

<summary><strong>3.4.8 — Operational Tooling Still Pending </strong></summary>

| Tool / Resource                       | Status  |
| ------------------------------------- | ------- |
| Reference FPV indexer                 | **TBD** |
| Settlement data / dashboard           | **TBD** |
| Governance and signing interface      | **TBD** |
| Direct on-chain registry interface    | **TBD** |
| Filecoin Pay contract reference links | **TBD** |
| Final admitted stablecoin whitelist   | **TBD** |.
---

← Previous: [2. Solstice Program Governance](02-solstice-program-governance.md) · [Back to README](../README.md) · Next: [4. Quarterly Review and Runbook](04-quarterly-review-and-runbook.md) →