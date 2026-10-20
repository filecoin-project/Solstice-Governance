# 7. Orchestrator Registry

> Part of the [Solstice Governance Repository](../README.md). This page is the canonical, human-readable list of Orchestrators admitted to the Solstice program.

This list is maintained by SRA Governance and mirrors on-chain SRA registry state — **the on-chain registry is the source of truth**; this page is a convenience view and audit trail. Entries are added and updated only through the SRA registry actions in [§2.3](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions): an Orchestrator appears on Admit (2.3.1), has its wallet updated on Replace (2.3.3), and is marked *Removed* on Remove (2.3.2).

**Contents**

- [7.1 Admitted Orchestrators](#71-admitted-orchestrators)
- [7.2 Status legend](#72-status-legend)
- [7.3 Per-Orchestrator entry template](#73-per-orchestrator-entry-template)
- [7.4 The Orchestrator seated at activation](#74-the-orchestrator-seated-at-activation)

---

## 7.1 Admitted Orchestrators

> **Program status:** the activation seat is recorded below; further Orchestrators appear when SRA Governance executes an Admit action ([§2.3.1](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)). 

| # | Orchestrator | Identity (`orch`) | Payout wallet | Status | Admitted (quarter / epoch) | Declaration | Registered (payer, operator) pairs |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | Initial (activation seat) | 0x97A90f5696be5E3C8d3752C92Adac287c2b4484e | 0x97A90f5696be5E3C8d3752C92Adac287c2b4484e | Active | quarter 1 | [Activation Report](quarterly-reports/FIL%20GTM%20Activation%20Report.md) | (0x5B27DbC6efeFbb5Ba8106fB19433048D60D6878F, 0xefA4ddBCe66B86B821A5A4238552b6F94C61d3fe) |

Removing the sole admitted Orchestrator burns the whole service stream from the epoch the removal binds until a later `SubmitShares` with eligible volume (FIP-0118, Security Considerations). That is the launch configuration.

## 7.2 Status legend

| Status | Meaning |
| :-- | :-- |
| **Active** | Admitted and operating; can `RegisterPairs`/`PostVolume`; FPV counts toward `SplitRule` and `AggregatedFPV`. |
| **Removed** | Permanently removed; (payer, operator) bindings released for re-registration ([§2.3.2](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)). |

## 7.3 Per-Orchestrator entry template

This block is the definitive list of declaration fields. [§3.1 Policy 6](03-orchestrator-operational-guidelines.md#31-policies) and [§3.2.4](03-orchestrator-operational-guidelines.md#32-orchestrators-tasks-and-actions) describe the duty; this is the field list.

When admitting an Orchestrator, add a row to the table in [§7.1](#71-admitted-orchestrators) and a detailed entry using this template:

​```markdown
### <Orchestrator name>

- **Identity (`orch`):** <f4… / 0x… address>
- **Payout wallet:** <f4… / 0x… address> (may differ from identity)
- **Status:** Active | Removed
- **Admitted:** <quarter, epoch, tx hash>
- **Declaration issue:** #<issue number> (link)
- **Contact:** <role address or shared alias, not a personal address; monitored per the §3.1 responsiveness rule>
- **Recovery contact:** <a second role address, distinct from the identity key and from the contact above>
- **Registered (payer, operator) pairs:** <pointer to on-chain registry / list>
- **Measurement rules:** <short description or link to service-contract metadata>
- **History:** <admit / replace / remove events, each with date + tx>
​```

> Field definitions follow the Orchestrator guidelines: identity and payout wallet in [§3.2](03-orchestrator-operational-guidelines.md#32-orchestrators-tasks-and-actions), the declaration file in [§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies), and the (payer, operator) binding rules in [§3.1, Policy 3](03-orchestrator-operational-guidelines.md#31-policies) (FIP-fixed uniqueness).

## 7.4 The Orchestrator seated at activation

FIP-0118 seats an Orchestrator at activation and requires it to "publish a disclosure on activation, following the requirements in the governance repository set for any other Orchestrator." That disclosure is on activation, not within 7 days of an `AddOrchestrator` admission ([§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies)). That Orchestrator is not admitted through an Admit action. On activation it publishes the same declaration file required of any other Orchestrator, and SRA Governance sets its row in [§7.1](#71-admitted-orchestrators) to **Active** with admission = quarter 1 (identity and payout wallet `0x97A90f5696be5E3C8d3752C92Adac287c2b4484e`, matching `initialOrchestrator` / `initialOrchestratorWallet` in [`deployments.json`](https://github.com/filecoin-project/solstice/blob/main/deployments.json)).

The row in §7.1 is chain id **314** (Filecoin mainnet). As of solstice `deployments.json` commit `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`, the same address is `initialOrchestrator` and `initialOrchestratorWallet` on calibnet (chain id 314159). A mainnet action names chain id 314. The same address string is not a check that the signer is on mainnet.

---

← Previous: [6. Program Change Log](06-changelog.md) · [Back to README](../README.md)
