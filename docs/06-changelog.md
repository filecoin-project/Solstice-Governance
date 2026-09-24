# 6. Program Change Log

## 2026-09-24 — Second-Safe dry-run and zero-volume quarter

- Standard SWA (§2.2) and SRA (§2.3) flows: the second Safe signs only after a successful dry-run of its own approval; note the result on the task-register issue; on revert, veto and resubmit (`UnanimousGovernance`; SRA header).
- Policy 5 / reporting: post FPV only when above zero; zero is not posted and binds at zero. Template §2.5 and runbook §4.1–4.2 accept `no message (zero quarter)` with indexer evidence.



## 2026-09-23 — Canonical calldata and payment-channel wallet

- Task register: action issue carries the calldata both Safes copy; address lists sorted ascending before encoding (`taskId` = `keccak256(msg.data)`).
- Admit (§2.3.1): checklist step — resolve payout wallet to actor ID; actor code must not be a payment channel (solstice #78 known gap).
- ReplaceWallet (§2.3.3): payment-channel recovery order — `ReplaceWallet` → `SubmitShares` → `RemoveOrchestrator`.


## 2026-09-23 — FIP-0118 leftovers (ReplaceWallet)

- Doc 02/03: rename leftover “controlling wallet” / `Replace(old, new)` to `ReplaceWallet` / payout wallet.
- Doc 03: `ReplaceWallet` is an immediate payout-wallet swap (FIP §3.2), not deferred to the next share-map push.
- Dropped non-FIP asides: payment-channel rule on identity; identity rotation as remove/re-admit.
- Doc 02 §2.4.1: Org-1 Safe addresses aligned to solstice `deployments.json` (`swaOwner2` / `sraOwner2`); SRA rule no longer says “either alone cancels” (binds at once per §2.1.2).
- Doc 02 §2.2.1 / §2.2.5: pairing duty retuned — FIP §3.1.1 sequencing (no `QuarterlyGateCheck` between effective epochs; SWA enforces only when holds overlap) plus cancel-recovery (re-send a paired write).
- Doc 02 §2.3.12: initial Orchestrator activation disclosure (Policy 6 declaration + Active / quarter 1 registry row for `0x97A9…484e`).
- Doc 02: task-register issue (calldata + `taskId`) before first approval; ReplaceOwner (§2.2.7 / §2.3.9) starts with veto-every-open-task (Owners.sol bit recycle).
- Doc 02 §2.3.4: CorrectVolume dual-approval internal deadline (day 5 / scaled on calibnet); first approver vetoes if the window closes without the second.
- Doc 03 §3.2.2 / §3.2.4 + quarterly report template: name an f1 claim keeper when payout is Safe/f410/EVM; “rewards received” = claimed (with txs) or accrued in f02.



## 2026-09-22 — Align operational docs with FIP-0118

Doc-only correction (Rod review §1): registry actions and cooperative `ReplaceOwner` bind without a hold; `POST_PERIOD` / `VERIFICATION_WINDOW` / `EPOCHS_PER_QUARTER` are FIP deployment-fixed; remove `FinalizeConversion`; clarify share-map no-op, filing-after-bind, pricing/gate parameter names, upgrade hold + `veto(taskId)`, SWA cancel paths, and Orchestrator identity vs payout wallet. No FIP or contract change.


> Part of the [Solstice Governance Repository](../README.md). This page is the chronological record of every change to the Solstice program rules — what changed, why, the community issue that raised it, and the resulting FIP (for rule/code changes) or pull request (for repository changes).
> 
> The Change Log records every permanent change to this repository, in two categories: (a) governance-tier actions (SRA and SWA) and their associated FIPs, PRs, and network-upgrade announcements; and (b) amendments to the repository's own content — Quarterly Orchestrator Report template and examples, admission rubric, parameters, and any section text.
>
> Both categories are made by pull request and recorded here.

## 6.1 How changes are classified

Per [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md), there are three tiers of change; every entry below is tagged with one:

| Type | What it covers | How it is enacted |
| :-- | :-- | :-- |
| **FIP + Network upgrade** | L1 itself — f02 code and its invariants (split logic, Σw ≤ 1, caps, the `SWA_TIMELOCK` value), or replacing the SWA address. | Accepted FIP **and** a coordinated network upgrade. |
| **FIP** | Contract code or a rule the threat model relies on (SWA/SRA code upgrade, gate parameters, tier powers, the two-Safes rule). | Accepted FIP + the standard held contract action. |
| **Repository (PR)** | Operational data and procedures (rosters, hold durations, Quarterlyy Orchestrator Report, playbooks, admitted lists, the Orchestrator Registry). | Pull request with public review. |

Each entry should link: (a) the **community issue / discussion** that raised the change, (b) the **FIP** if one was required, and/or (c) the **PR** that landed the repository edit.

## 6.2 Change log

> **Entries are ordered most-recent first.** The log begins at program establishment (Nv29); further entries are appended as the program evolves.

| Date | Change | Type | Community issue / discussion | FIP | PR / commit | Status |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 2026 (pending) | Establishment of the Solstice program and this governance repository; deprecation of Filecoin Plus. | FIP + Network upgrade | [FIP-0118 discussion](https://github.com/filecoin-project/FIPs/discussions/1249) | [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) | Initial repository | Draft |

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

> **Why this log exists:** Solstice iterates in the open. Parameters and procedures are meant to change without a FIP (by PR), while rules and code change only through a FIP. This log makes the full history auditable in one place — from the community issue that surfaced a problem, through the FIP or PR that resolved it, to the section it changed.

---

← Previous: [5. Quarterly Reports](05-quarterly-reports.md) · [Back to README](../README.md)