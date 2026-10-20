# 3. Orchestrator Operational Guidelines

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) and referenced throughout. Quotations of FIP-0118 are copied from the commit pinned in the [README](../README.md).

This section consolidates what is expected of Orchestrators: the operational guidelines, policies, tasks, and monitoring tools. Declaration, verification and escalation are covered in [§4.4](04-quarterly-review-and-runbook.md#44-declaration-verification-and-escalation).

**Contents**

- [3.1 Policies](#31-policies)
- [3.2 Orchestrators Tasks and Actions](#32-orchestrators-tasks-and-actions)
- [3.3 Monitoring tools & references](#33-monitoring-tools--references)
- [3.4 Solstice Orchestrator Task Checklist](#34-solstice-orchestrator-task-checklist)

The service stream pays for measured, paid storage service. These rules give Orchestrators predictability about what is expected and what the consequences of a breach are, and give the network confidence that the number driving the gate and the shares reflects reality. The standing test for any action in the program: **does this volume represent a client paying for storage service on Filecoin?** If the answer is yes, the Orchestrator is operating within both the rules and their intent.

Orchestrators are free to run their businesses autonomously. The program does not review their pricing, their client selection, or their operations. To maintain trust in the measurement layer, all Orchestrators are expected to adhere to the criteria below. These may be revised over time through the standard PR process (and logged in the [Program Change Log](06-changelog.md)).

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
| 2 | Qualifying volume is settlement through admitted Filecoin Pay contracts, in admitted stablecoins or FIL converted off-chain via the reference indexer using public fee-auction prints (`MIN_LOT`, `PRICE_BAND`), attributable to the Orchestrator's registered (payer, operator) pairs. Nothing else counts. | Contract |
| 3 | A (payer, operator) pair binds to exactly one Orchestrator. Registering a pair already bound elsewhere reverts. Clients are free to work with multiple Orchestrators across different operator relationships; the pair, not the client, is the unit of attribution. | FIP |
| 4 | A registration applies from its execution epoch and also covers the pair's unbound past inside the same quarter, back to the later of the quarter's first epoch and the epoch the pair last became unbound (back-cover). A registration in `[Start(Q+1) - REGISTRATION_CUTOFF, Start(Q+1))` applies from `Start(Q+1)` and covers nothing of quarter Q. `REGISTRATION_CUTOFF` is a governance parameter (initial mainnet value 20,160 epochs). A change to it applies from the first epoch of the next quarter. | FIP |
| 5 | Each quarter, Orchestrators post FPV within `POST_PERIOD` when it is above zero. `PostVolume` rejects zero (a zero total is equivalent to not posting); zero is not posted and binds at zero. A value neither posted nor corrected during `VERIFICATION_WINDOW` likewise binds at zero. Bound values are final; a successful appeal affects later quarters only. | FIP and contract |
| 6 | Each Orchestrator publishes its declaration in this repository: a short description of its service, contact information, a recovery contact, its registered pairs, and any pointer to service-contract metadata.  An Orchestrator admitted by `AddOrchestrator` publishes it within 7 days of admission. The Orchestrator seated at activation publishes it on activation (FIP-0118 §4.2), not on that 7-day clock. | Repository, and FIP for the activation disclosure |
| 7 | An Orchestrator with no settled volume for a number of consecutive quarters enters registry review and may be removed. Removal on inactivity is registry hygiene: an idle registration adds attack surface without adding measurement value. | Repository |
| 8 | No registry address participates in either governance tier, as a governance multisig or as a key holder within one. | FIP |
| 9 | Quarterly Orchestrator Report on claims and reward reception. Failure to file is a removal trigger and forfeits re-admission for the following quarter (FIP-0118 §4.2). The template, required disclosures, and filing cadence are set in this repository. A later return is a fresh `AddOrchestrator` and a fresh application, and it is not available in the forfeited quarter. | FIP and Repository |

Policies 5 and 8 restate FIP-0118 text. Policy 3 restates the uniqueness sentence. Policy 4 restates the application rules in FIP-0118 §2.2; it is not a repository rule.

> "Indeed, the SRA enforces that each (payer, operator) pair is bound to at most one Orchestrator, so a registration that duplicates an existing binding reverts."
>
> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter." … "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."
>
> "No address in the registry may participate in either contract governance tier, as a governance multisig or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

## 3.2 Orchestrators Tasks and Actions

> These are operational tasks and actions; each is a collapsible dropdown. Changes to them follow the process in [§6.1](06-changelog.md#61-how-changes-are-classified).
> <img width="1600" height="897" alt="image" src="https://github.com/user-attachments/assets/c1750749-9809-452d-9024-7fc904896188" />



<details>
<summary><strong>3.2.1 — Deal making</strong></summary>

> **Cadence:** continuous · **On-chain call:** none

Orchestrators set their own pricing, choose their own clients, and structure their deals, subsidies, and operations as they see fit. The program does not review or approve commercial terms, client selection, or operations. The only constraint is measurability: paid storage-service revenue must settle through admitted Filecoin Pay rails on registered (payer, operator) pairs so it can be counted.

</details>

<details>
<summary><strong>3.2.2 — Orchestrator identity and payout wallet</strong></summary>

> **Cadence:** at admission, and as needed (`ReplaceWallet`) · **On-chain call:** `ReplaceWallet(old, new)` (payout-wallet rotation)
Each Orchestrator has **two** on-chain addresses in the SRA registry (`AddOrchestrator(orch, wallet)`):

1. **Orchestrator identity (`orch`)** — calls `RegisterPairs` and `PostVolume`.
2. **Payout wallet (`wallet`)** — receives the service-stream share from f02. Must not be a payment-channel actor; f02 rejects payment channels as share recipients. Prefer a multisig (e.g., a Safe) with hardware-key signers. If the payout wallet is a Safe or other f410 / EVM-contract address, also name an **f1 claim keeper** (see §3.2.4).

They can differ. As part of admission (see [§2.3 SRA Governance Tier](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)), record both addresses in your GitHub declaration. Your entry appears in the [Orchestrator Registry](02-solstice-program-governance.md#2312-orchestrator-registry-admitted-orchestrators) with both columns.

Rotate a compromised **payout** wallet by filing the public request in [§2.3.3](02-solstice-program-governance.md#233--replace--rotate-an-orchestrator-address-replacewallet). SRA Governance then calls `ReplaceWallet(old, new)`. Per FIP-0118 §3.2, that call runs only on a public request from the Orchestrator recorded in this repository, and it swaps the payout wallet through an immediate `ReplaceAddress`, so f02 pays the new wallet at once. The Orchestrator does not send the governance call. If the identity key is lost, the request cannot be filed and `ReplaceWallet` does not run.

</details>


<details>
<summary><strong>3.2.3 — Linking deals to Filecoin Pay services</strong></summary>

> **Cadence:** per deal · **On-chain call:** `RegisterPairs`

Volume counts when it settles on an admitted Filecoin Pay contract, in an admitted stablecoin or FIL, on a (payer, operator) pair bound to you under the FIP application rules ([Policy 4](#31-policies)). A registration applies from its execution epoch and covers the pair's unbound past inside the same quarter (back-cover). A registration in `[Start(Q+1) - REGISTRATION_CUTOFF, Start(Q+1))` applies from `Start(Q+1)` and covers nothing of quarter Q. Admitted lists and pricing parameters in force are those that apply to the quarter, not a list changed mid-quarter ([§2.3.7 Maintain admitted lists & fee-auction parameters](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)). To link a deal:

1. Set up the client's payment on an admitted Filecoin Pay contract (the latest `SetAdmittedLists` event that applies to the quarter).
2. Create the rail with the operator you will register.
3. Register the (payer, operator) pair via `RegisterPairs`. A pair already bound elsewhere reverts (uniqueness invariant). Registering promptly still matters: back-cover does not cross a previous binding, a removal's release epoch, or the cutoff into the previous quarter.
4. The client settles on-rail (`RailSettled` or one-time payments); those settlements accrue to your FPV when the binding covers their epoch.

> **Note:** volume off admitted rails, or in non-admitted assets, is not counted. Back-cover does not reach it.

</details>

<details>
<summary><strong>3.2.4 — Measurement and receiving funds</strong></summary>

> **Cadence:** measurement quarterly · payout each epoch · **On-chain call:** `PostVolume` (measurement)

1. Understand your figure: FPV_i(Q) is the value settled on your registered (payer, operator) rails during a given quarter; amounts are denominated in USD — admitted stablecoins at face value, FIL converted off-chain via the reference indexer using public fee-auction prints (`MIN_LOT`, `PRICE_BAND`).
2. Declare how your volume is measured (your pairs, optional service-contract metadata, booked-revenue basis).
3. Receiving service-stream payouts: f02 credits your **payout wallet** with its share of the service stream (w2) every epoch. Entitlements are withdrawn via the permissionless native `Claim` method. An f410 (including a Safe) or an EVM-contract wallet cannot send that call; an **f1 keeper account** must. Name the keeper in your declaration, using the field list in the [per-Orchestrator entry template](02-solstice-program-governance.md#2312-orchestrator-registry-admitted-orchestrators). Prefer keeping the Safe as the payout wallet for custody and using the f1 only to `Claim`.

The following rule establishes how the share is set: `SplitRule` = your bound FPV_i(Q) ÷ AggregatedFPV(Q), written into f02 once per quarter via `SetShares`.

If your bound FPV for the quarter is zero while another Orchestrator has bound volume, your share for that quarter is zero and your wallet is never debited. If no Orchestrator has bound volume (eligible FPV sums to zero), `SubmitShares` is a benign no-op: the existing share map stands.

</details>

<details>
<summary><strong>3.2.5 — Treasury function and wallet-safety best practices</strong></summary>

> **Cadence:** continuous · **On-chain call:** none (off-protocol)

Rewards land in your payout wallet each epoch; deploying them against your mandate (client acquisition, subsidies, integrations, migrations, infrastructure, etc.) is your responsibility and lies outside the protocol. Recommended practices:

- **Custody:** hold reserves in a multisig (e.g., Safe) with hardware-key signers; never keep the treasury on a single hot key.
- **Segregation:** keep a cold reserve (offline/hardware, high signing threshold) separate from a small hot operating wallet; sweep excess from hot to cold.
- **Key hygiene:** never expose or share private keys or seed phrases, and never place them in code, configs, or shared documents; if exposure of the payout wallet is even suspected, file a `ReplaceWallet` request immediately (§3.2.2).
- **Least privilege:** where you use exchanges or custodial APIs, prefer read-only keys and disable withdrawal/trading scopes you don't need.
- **Monitoring & recovery:** alert on inflows/outflows, name an accountable key-custody owner, and keep a documented, tested recovery path.

</details>

<details>
<summary><strong>3.2.6 — Reporting and reconciliation</strong></summary>

> **Cadence:** quarterly (`POST_PERIOD`, then `VERIFICATION_WINDOW`) · **On-chain call:** `PostVolume`

1. Within `POST_PERIOD`, if your FPV is above zero, post it via `PostVolume` as a single USD-denominated total, with FIL volume converted off-chain by the indexer per the fee-auction pricing rule. If the quarter’s FPV is zero, do **not** call `PostVolume` (the contract rejects zero); the value binds at zero the same as a non-post.
2. During `VERIFICATION_WINDOW`, recompute your own FPV from public settlement events using the reference indexer and reconcile it against what you posted (or against zero if you did not post). Flag any discrepancy proactively on the quarter's issue, with the events and blocks you used. Only SRA Governance can act on it: a correction is `CorrectVolume`, called by both Registry Safes jointly and only while the window is open. If the window closes without a correction, the posted figure — or zero — binds and is final for that quarter.
3. Keep your declaration file (pairs, contact, measurement rules) current.
4. Remember the failure mode: a value neither posted nor corrected binds at zero (a conservative under-count); bound values are final, and a successful appeal affects later quarters only.

FIP-0118 fixes the posting period and the no-clawback rule:

> "Posting: each Orchestrator posts `FPV_i(Q)` to the SRA during the posting period [`Start(Q+1)`, `Start(Q+1)` + `POST_PERIOD`) (`Start` as defined above)."
>
> "There is no retroactive clawback: a misreport discovered after binding is grounds for audit or removal (Section 4)."

</details>

<details>
<summary><strong>3.2.7 — Declarations, verification, and appeal</strong></summary>

> **Cadence:** as triggered · **On-chain call:** none (governance-side `CorrectVolume` / `ReassignBinding`)

Declarations of client relationships are accepted by default. SRA Governance still recomputes every posted figure each quarter ([§2.3.4](02-solstice-program-governance.md#234--quarterly-fpv-verification--correction-correctvolume)); that duty is not dispute-driven. An audit of a business relationship is exception-based ([§4.4](04-quarterly-review-and-runbook.md#44-declaration-verification-and-escalation)). If a binding is contested or a figure disputed, the process runs through [§4.4.2](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings). A contested binding that should move uses `ReassignBinding` ([§2.3.11](02-solstice-program-governance.md#2311--reassign-a-binding-reassignbinding)). Appeals of a `CorrectVolume` follow the same evidence process and can affect only later quarters, since bound values are final.

</details>

<details>
<summary><strong>3.2.8 — Release a binding (CancelBinding)</strong></summary>

> **Cadence:** when a client relationship ends, or a client moves to another Orchestrator · **On-chain call:** `CancelBinding(pair)`, by the bound Orchestrator, for itself

`CancelBinding` is not a governance call. It releases a pair the caller holds. The release applies at the execution epoch. The Orchestrator keeps the span it served. Any admitted Orchestrator may register the pair again, and that registration's back-cover reaches no further back than the release (FIP-0118 §2.2 and §3.2).

Use it for a cooperative exit: a client relationship that ended, or a client moving to another Orchestrator (FIP-0118 §3.2). The contract does not know whether a dispute is open. A contested binding is resolved by SRA Governance with `ReassignBinding` or removal ([§2.3.11](02-solstice-program-governance.md#2311--reassign-a-binding-reassignbinding) and [§2.3.2](02-solstice-program-governance.md#232--remove-an-orchestrator-removeorchestrator)).

The call reverts unless the caller holds the binding. State the chain id in any issue that records the release. Mainnet and calibnet can show the same contract address (§2.4.1 of the governance page).

</details>

## 3.3 Monitoring tools & references

*(links to be updated)*

These tools are meant to ease the monitoring of the program's activity. Orchestrators use them to check their recomputed FPV against what they posted; SRA Governance and any community observer run the same tools for oversight. Every verification action and its conclusion must be reproducible from these public sources.

- Versioned reference indexer: [fidlabs/fpv-indexer](https://github.com/fidlabs/fpv-indexer).
- Governance & signing interface: *(links to be added)* 
- Settlement data and dashboards: see [fidlabs/fpv-indexer public instances](https://github.com/fidlabs/fpv-indexer#public-instances) for calibnet/mainnet URLs
- Registry state and admitted Orchestrators: see the [Orchestrator Registry](02-solstice-program-governance.md#2312-orchestrator-registry-admitted-orchestrators). The admitted-stablecoin whitelist and the admitted Filecoin Pay addresses are the latest `SetAdmittedLists` event that applies to the quarter, not the registry table. *(on-chain links to be added)*
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

### 3.4.1 — Every New Client / Deal

Complete when taking on a new payer/operator relationship. Counting follows [Policy 4](#31-policies), not a rule that volume before registration is lost.

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
| [ ]  | Register payer/operator pair using `RegisterPairs`         | When the client relationship starts. Back-cover counts the pair's unbound on-rail past inside the quarter. A registration inside `REGISTRATION_CUTOFF` of the next quarter covers nothing of the quarter that is ending. | [Pair Registration Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)  |
| [ ]  | Confirm registration succeeded on-chain                    | Immediately after registration | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Confirm payments are settling through the registered rail  | After setup                    | [Settlement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)         |
| [ ]  | Retain evidence supporting the client relationship         | Ongoing                        | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)            |

> **Important:** Back-cover attributes the pair's unbound on-rail past inside the quarter. It does not cross an earlier binding, a release epoch, the `REGISTRATION_CUTOFF` into the previous quarter, or volume that never settled on an admitted rail.

---

### 3.4.2 — Treasury & Wallet Management

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

### 3.4.3 — Quarter-End FPV Calculation

| Done | Task                                                        | Timing              | Reference                                                                                                                                    |
| ---- | ----------------------------------------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Confirm the reporting quarter has ended                     | Quarter close       | [Timing Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters)  |
| [ ]  | Identify active registered payer/operator pairs             | Quarter close       | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Retrieve settlement activity for those pairs                | Quarter close       | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)               |
| [ ]  | Apply the FIP application rules (back-cover, `REGISTRATION_CUTOFF`, releases, and `inherit`) | Quarter close       | [Measurement Rules](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)        |
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

### 3.4.4 — Post Quarterly FPV

| Done | Task                                      | Timing                    | Reference                                                                                                                                   |
| ---- | ----------------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]  | Confirm `POST_PERIOD` is open             | Post period               | [Timing Parameters](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/02-solstice-program-governance.md#24-parameters) |
| [ ]  | Confirm final FPV amount                  | Before posting            | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)              |
| [ ]  | Submit FPV using `PostVolume` (skip if FPV is zero — contract rejects zero; binds at zero) | Post period               | [PostVolume Requirements](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md) |
| [ ]  | Confirm transaction succeeded             | Immediately after posting | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)              |
| [ ]  | Confirm posted FPV matches calculated FPV | Immediately after posting | [Guidelines](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/03-orchestrator-operational-guidelines.md)              |
| [ ]  | Save posting transaction reference        | Immediately after posting | —                                                                                                                                           |

---

### 3.4.5 — Verification Window

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
| [ ]  | Provide evidence supporting any correction request. `CorrectVolume` is called by both Registry Safes jointly — an Orchestrator cannot correct its own bound figure | Before window closes | [§2.3.4 CorrectVolume](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions) |
| [ ]  | Confirm final FPV                                        | Before window closes | [Verification Runbook](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/04-quarterly-review-and-runbook.md)           |

---

### 3.4.6 — Quarterly Community Report

| Done | Task                                             | Timing                   | Reference                                                                                                                      |
| ---- | ------------------------------------------------ | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| [ ]  | Open the current Quarterly Orchestrator Report template | Each quarter | [Report Template](https://github.com/filecoin-project/Solstice-Governance/blob/main/docs/quarterly-reports/_TEMPLATE.md) |
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

### 3.4.7 — Quarter Closeout

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
| [ ]  | Release ended client pairs with `CancelBinding`            |
| [ ]  | Carry unresolved operational actions into the next quarter |

### 3.4.8 — Operational Tooling Still Pending

| Tool / Resource                       | Status  |
| ------------------------------------- | ------- |
| Reference FPV indexer                 | [fidlabs/fpv-indexer](https://github.com/fidlabs/fpv-indexer) |
| Settlement data / dashboard           | see [fidlabs/fpv-indexer public instances](https://github.com/fidlabs/fpv-indexer#public-instances) |
| Filecoin Pay contract reference links | **TBD** |
| Final admitted stablecoin whitelist   | axlUSDC and USDFC — the authoritative record is the latest `SetAdmittedLists` event, applied from the next quarter |
---

← Previous: [2. Solstice Program Governance](02-solstice-program-governance.md) · [Back to README](../README.md) · Next: [4. Quarterly Review and Runbook](04-quarterly-review-and-runbook.md) →
