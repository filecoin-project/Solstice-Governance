# 6. Program Change Log

> Part of the [Solstice Governance Repository](../README.md). This page is the chronological record of every change to the Solstice program rules — what changed, why, the community issue that raised it, and the resulting FIP (for rule/code changes) or pull request (for repository changes).
>
> **The Change Log records every permanent change to this repository, starting from the Nv29 upgrade epoch**. Those changes are split in two categories: (a) governance-tier actions (SRA and SWA) and their associated FIPs, PRs, and network-upgrade announcements; and (b) amendments to the repository's own content — Quarterly Orchestrator Report template and examples, admission rubric, parameters, and any section text.
>
> Both categories are made by pull request with public review, and recorded here. This is the process referred to from the README, §2.3, §3.2 and §4.2.

> **This section is the single definition of what counts as a change, how each type is enacted, and what must be recorded.** Other pages point here rather than restating it.

## 6.1 How changes are classified

Per [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) §4.4, there are three types of change; every entry below is tagged with one:

| Type | What it covers | How it is enacted |
| :-- | :-- | :-- |
| **FIP + Network upgrade** | L1 itself — f02 code and its invariants (split logic, Σw ≤ 1, caps, the `SWA_TIMELOCK` value), or replacing the SWA address. | Accepted FIP **and** a coordinated network upgrade. |
| **FIP** | Contract code or a rule the threat model relies on (SWA/SRA code upgrade, gate parameters, tier powers, the two-multisig rule, and these hold durations: the SWA-internal timelock, `SRA_UPGRADE_HOLD`, `POST_PERIOD`, `VERIFICATION_WINDOW`, `EPOCHS_PER_QUARTER`). | Accepted FIP, and the contract action the FIP requires. A pull request cannot change a hold. |
| **Repository (PR)** | Operational data and procedures (rosters, the Quarterly Orchestrator Report, playbooks, mirrors of admitted lists, the Orchestrator Registry). | Pull request with public review. |

Each entry should link: (a) the **community issue / discussion** that raised the change, (b) the **FIP** if one was required, and/or (c) the **PR** that landed the repository edit.

## 6.2 Change log

> **Entries are ordered most-recent first.** The log begins at program establishment (Nv29); further entries are appended as the program evolves.

| Date | Change | Type | Community issue / discussion | FIP | PR / commit | Status |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 2026-10-14 | Procedures and FIP quotations aligned with FIP-0118 at `23b4296` (back-cover, `CorrectVolume` publication, `ReassignBinding` / `CancelBinding`, quarterly recomputation, chain id). See the dated subsection. | Repository (PR) | No GitHub issue. Review dated 2026-10-14. | n/a. Quotations are from [FIP-0118 @ `23b4296`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md). | `review-fixes-fip0118` | Accepted |
| 2026-10-09 | Upgrade-runbook link; one Safe-replacement section (§2.5.3) for both tiers. | Repository (PR) | n/a | n/a | [#17](https://github.com/filecoin-project/Solstice-Governance/pull/17) `142926f3ff38ed10166725a95a1a73500c35af3a` | Accepted |
| 2026 (pending) | Establishment of the Solstice program and this governance repository; deprecation of Filecoin Plus. | FIP + Network upgrade | [FIP-0118 discussion](https://github.com/filecoin-project/FIPs/discussions/1249) | [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) | Initial repository | Draft |


### 2026-10-21 — Reference FPV indexer published

- **Type:** Repository (PR)
- **Summary:** The versioned reference indexer is published at [fidlabs/fpv-indexer](https://github.com/fidlabs/fpv-indexer). Links replace the "not yet published" placeholders in §3.3 and §3.4.8, and the conditional fallback to the pinned FIP text is removed from §2.3.4, §2.3.10 and §4.4.1(a). The README's quotation pin is retained, narrowed to quotations only. §3.3 now states how an indexer version is cited.
- **Community issue / discussion:** [#22](https://github.com/filecoin-project/Solstice-Governance/pull/22)
- **FIP:** n/a. No FIP text is amended.
- **PR / commit:** n/a
- **Affected sections:** README §1.2 and §1.4, §2.3.4, §2.3.10, §3.3, §3.4.5, §3.4.8, §4.4.1
- **Status:** Accepted

### 2026-10-20 — Orchestrator Registry moved to its own page

- **Type:** Repository (PR)
- **Summary:** The Orchestrator Registry moved from §2.3.12 of the Program Governance page to its own page, [§7](07-orchestrator-registry.md), so that it can be linked to directly. No registry entry, status, or procedure changed. The per-Orchestrator entry template, which had been lost from §2.3.12, is restored in §7.3. §2.3 keeps pointers to the new page from the standard registry-change flow and from Admit (2.3.1). Actions 2.3.1–2.3.11 are unchanged and not renumbered.
- **Community issue / discussion:** Review feedback, 20 October 2026.
- **FIP:** n/a. No FIP text is amended.
- **PR / commit:** <fill in>
- **Affected sections:** README §1.2, §2.3, §2.3.12 (removed), §3.2.2, §3.2.4, §3.3, §6.2, new §7
- **Status:** Accepted

## 2026-10-14 — FIP-0118 procedure and quotation fixes

- **Type:** Repository (PR)
- **Summary:** Bring the operating procedures into line with FIP-0118 at `23b42964890590a49c9cbc6e19a7e9e361d1f8a0`. Policy 4 states back-cover and `REGISTRATION_CUTOFF`. The recomputation is published on the task-register issue before either Safe approves. A missed second signature is recorded as correction-blocked rather than treated as a cancel. `ReassignBinding` (including when `inherit = true`) and Orchestrator `CancelBinding` have procedures. Quotations that were not in the FIP are replaced or removed, including the claim that either Safe can cancel every discretionary change. The quarterly recomputation is an SRA duty every quarter, separate from exception audits. Task-register issues state the chain id, because the SRA and SWA proxies match on mainnet and calibnet. Admitted lists apply from the next quarter. The hostile-Safe path follows FIP §4.3 (SRA re-point by SWA write; SWA replacement by network upgrade), correcting the wording PR #17 had introduced.
  
- **Community issue / discussion:** No GitHub issue. Review dated 2026-10-14.
- **FIP:** n/a. No FIP text is amended.
- **PR / commit:** branch `review-fixes-fip0118`
- **Affected sections:** README, §2, §3, §4, templates, this log
- **Status:** Accepted

## 2026-10-09 — Upgrade runbook link and one Safe-replacement section (PR #17)

- **Type:** Repository (PR)
- **Summary:** README points at the solstice contract upgrade runbook. The per-tier cooperative `ReplaceOwner` sections are folded into §2.5.3 (veto every open task, then replace, record the new Safe, engineering follow-up in the owner-change FAQ). That pull request also said a hostile replacement of either contract needs a network upgrade because the proxies are hardwired. The 2026-10-14 entry restores the FIP §4.3 split; that claim from PR #17 does not describe the current procedure.
- **Community issue / discussion:** n/a
- **FIP:** n/a
- **PR / commit:** [#17](https://github.com/filecoin-project/Solstice-Governance/pull/17), merge `142926f3ff38ed10166725a95a1a73500c35af3a`
- **Affected sections:** README, §2.5.3 (and the removal of the old §2.2.7 / §2.3.9 replacement sections)
- **Status:** Accepted

## 2026-09-24 — Second-Safe dry-run and zero-volume quarter

- Standard SWA (§2.2) and SRA (§2.3) flows: the second Safe signs only after a successful dry-run of its own approval; note the result on the task-register issue; on revert, veto and resubmit (`UnanimousGovernance`; SRA header).
- Policy 5 / reporting: post FPV only when above zero; zero is not posted and binds at zero. Template §2.5 and runbook §4.1–4.2 accept `no message (zero quarter)` with indexer evidence.

## 2026-09-23 — Canonical calldata and payment-channel wallet

- Task register: action issue carries the calldata both Safes copy; address lists sorted ascending before encoding (`taskId` = `keccak256(msg.data)`).
- Admit (§2.3.1): checklist step — resolve payout wallet to actor ID; actor code must not be a payment channel ([solstice #78](https://github.com/filecoin-project/solstice/issues/78)).
- ReplaceWallet (§2.3.3): payment-channel recovery order — `ReplaceWallet` → `SubmitShares` → `RemoveOrchestrator`.

The §2.3.1 actor-code step recorded here was later dropped from the procedure in error. The 2026-10-14 entry restores it, on admission and on `ReplaceWallet`.
## 2026-09-23 — FIP-0118 leftovers (ReplaceWallet)

- Doc 02/03: rename leftover “controlling wallet” / `Replace(old, new)` to `ReplaceWallet` / payout wallet.
- Doc 03: `ReplaceWallet` is an immediate payout-wallet swap (FIP §3.2), not deferred to the next share-map push.
- Dropped non-FIP asides: payment-channel rule on identity; identity rotation as remove/re-admit.
- Doc 02 §2.4.1: Org-1 Safe addresses aligned to solstice `deployments.json` (`swaOwner2` / `sraOwner2`); SRA rule no longer says “either alone cancels” (binds at once per §2.1.2).
- Doc 02 §2.2.1 / §2.2.5: pairing duty retuned — FIP §3.1.1 sequencing (no `QuarterlyGateCheck` between effective epochs; SWA enforces only when holds overlap) plus cancel-recovery (re-send a paired write).
- Doc 02 §2.3.12: initial Orchestrator activation disclosure (Policy 6 declaration + Active / quarter 1 registry row for `0x97A9…484e`).
- Doc 02: task-register issue (calldata + `taskId`) before first approval. Cooperative `ReplaceOwner` starts with veto-every-open-task (`Owners.sol` bit recycle). PR #17 moved that procedure to §2.5.3. It is not §2.2.7 or §2.3.9.
- Doc 02 §2.3.4: CorrectVolume dual-approval internal deadline (day 5 / scaled on calibnet); first approver vetoes if the window closes without the second.
- Doc 03 §3.2.2 / §3.2.4 + quarterly report template: name an f1 claim keeper when payout is Safe/f410/EVM; “rewards received” = claimed (with txs) or accrued in f02.

## 2026-09-22 — Align operational docs with FIP-0118

Doc-only correction (engineering review, September 2026): registry actions and cooperative `ReplaceOwner` bind without a hold; `POST_PERIOD` / `VERIFICATION_WINDOW` are FIP deployment-fixed; remove `FinalizeConversion`; clarify share-map no-op, filing-after-bind, pricing/gate parameter names, upgrade hold + `veto(taskId)`, SWA cancel paths, and Orchestrator identity vs payout wallet. No FIP or contract change.

## 6.3 Entry template

Prepend new entries to the table above (most-recent first). For a fuller record, add a dated subsection below using this template:

```markdown
### YYYY-MM-DD — <short title>

- **Type:** FIP + Network upgrade | FIP | Repository (PR)
- **Summary:** <what changed and why>
- **Community issue / discussion:** #<issue> or discussion link
- **FIP:** <FIP number + link, or "n/a">
- **PR / commit:** #<PR> or commit hash
- **Affected sections:** <e.g., §2.3 SRA Governance Tier, §2.4 Parameters>
- **Status:** Proposed | Accepted | Live | Reverted
```

> **Why this log exists:** Solstice iterates in the open. Procedures in this repository change by pull request. Rules, code, and the hold durations change only through a FIP. This log makes the full history auditable in one place — from the community issue that surfaced a problem, through the FIP or PR that resolved it, to the section it changed.

---

← Previous: [5. Quarterly Reports](05-quarterly-reports.md) · [Back to README](../README.md) · Next: [7. Orchestrator Registry](07-orchestrator-registry.md) →
