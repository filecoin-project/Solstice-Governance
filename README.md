# Solstice Governance Repository
## 1.1 Purpose

This repository is the operational layer of the governance defined in the Solstice FIP. The FIP fixes what is enforced: the two governance tiers and their powers, the change lifecycle, and the invariants the threat model relies on. This repository holds how those tiers operate: addresses for each governance Safe, the procedures for verification, rotation, disputes, and the parameters designed to iterate without a FIP.

## 1.2 What lives here
### How this document maps to the repository

Each section below is a separate page, ordered by its number, so a reader interested in only one topic can open that page alone. File names carry the section number so they sort in reading order.

| Section | Repository file | Covers |
| :-- | :-- | :-- |
| 1. README | [`README.md`](README.md) | Purpose, references, section map |
| 2. Solstice Program Governance | [`docs/02-solstice-program-governance.md`](docs/02-solstice-program-governance.md) | Governance tiers & Safes · SWA tier · SRA tier · Parameters · Safety and rotation playbook |
| 3. Orchestrator Operational Guidelines | [`docs/03-orchestrator-operational-guidelines.md`](docs/03-orchestrator-operational-guidelines.md) | Orchestrator guidelines · policies · tasks · monitoring |
| 4. Quarterly Review and Runbook | [`docs/04-quarterly-review-and-runbook.md`](docs/04-quarterly-review-and-runbook.md) | Quarterly community reporting duty · report template · example · declaration, verification & escalation |
| 5. Reports | [`docs/05-quarterly-reports.md`](docs/05-quarterly-reports.md) | Archive/history of filed quarterly reports (one page per report) |
| 6. Program Change Log | [`docs/06-changelog.md`](docs/06-changelog.md) | Rule-change history with community issues & FIP links |
| 7. Orchestrator Registry | [`docs/07-orchestrator-registry.md`](docs/07-orchestrator-registry.md) | Canonical list of admitted Orchestrators · status legend · entry template · the activation seat |

### Contains key documentation including:
- Tasks and actions for each ecosystem actor, inclusive of the operational guidelines for each role, removal criteria for Orchestrators, and the admission rubric once agreed.
- Measurement-rule declarations for each Orchestrator, published here as Orchestrators are admitted.- The registry of admitted Orchestrators and the change log of program-rule updates.
- The dispute procedure for contested bindings, and the verification playbook.
- The versioned reference indexer that recomputes FPV from public settlement events. FIP-0118 §2.2 highlights that a versioned reference implementation is maintained here, and that the FIP text is the normative algorithm.
- Pointers to the FIP-fixed durations of `POST_PERIOD`, `VERIFICATION_WINDOW`, `SRA_UPGRADE_HOLD` and `EPOCHS_PER_QUARTER` (not edited here; changed only by an SRA code upgrade).
- [The contract upgrade runbook](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md) in the [solstice](https://github.com/filecoin-project/solstice) repository. Upgrades are driven by engineers at the request of governance (an accepted FIP, approved by both tiers' Safes), and the runbook stays next to the scripts and workflows it depends on so they are reviewed together.
- The wallet addresses for each tier's Safes.
- Issue templates, application forms, and workflow automation.
- The [registry of admitted Orchestrators](docs/07-orchestrator-registry.md) and the [change log of program-rule updates](docs/06-changelog.md).
  
## 1.3 Changing this repository

Changes happen by pull request with public review. What counts as a change, how each type is enacted, and what must be recorded are defined once in [§6.1 of the Program Change Log](docs/06-changelog.md#61-how-changes-are-classified). Every change to the program rules is recorded there.


## 1.4 Reference Links

- FIP text (latest): [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md)
- FIP text pinned for quotations in this repository: [FIP-0118 at `23b42964890590a49c9cbc6e19a7e9e361d1f8a0`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md). A quotation is updated in the same pull request that follows a change in that text. Until the reference indexer is published, this pinned text is the normative FPV algorithm.
- Contract upgrade runbook: [solstice `docs/UPGRADE.md`](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md). An upgrade issue pins this file by commit. The pending upgrade and its execution must be observable on chain (FIP-0118 §4.2).
- [FPV indexer](https://fpv-indexer.calibration.allocator.tech/#/Volume/VolumeController_getServiceOrchestratorQuarterlyVolume)
- [Solstice Watchtower](https://solsticewatchtower.eth.limo/)
- [Licence](LICENSE) and [Disclaimer](DISCLAIMER.md)


## 1.5 Repository Maintainers

Maintainers keep this repository current and administer pull requests. Authority over the Solstice program rests with two governance tiers (the SWA and SRA Safes) as fixed by FIP-0118.

---

*Solstice Governance Repository — operational layer of [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md).*
