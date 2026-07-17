# Enterprise Acquisition Simulation Lab

![Windows Server](https://img.shields.io/badge/Windows_Server-2025-0078D4?style=flat&logo=windows&logoColor=white)
![Azure](https://img.shields.io/badge/Microsoft_Azure-VNet_Peering-0089D6?style=flat&logo=microsoftazure&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-Cross--Forest_Trust-0078D4?style=flat&logo=windows&logoColor=white)
![ADMT](https://img.shields.io/badge/ADMT-SID_History_Migration-5C2D91?style=flat)
![PowerShell](https://img.shields.io/badge/PowerShell-Validated-5391FE?style=flat&logo=powershell&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat)

A hands-on simulation of an enterprise acquisition — standing up a second, independent Active Directory forest to represent an acquired company, integrating it with an existing production-style domain via a hardened cross-forest trust, migrating identities with SID history preserved using ADMT, and cleanly decommissioning the trust once integration is complete.

## Overview

| # | Lab | Focus |
|---|-----|-------|
| 1 | [Acquired Environment & Network Integration](#lab-1-acquired-environment--network-integration) | Independent forest provisioning, VNet peering, DNS conditional forwarding |
| 2 | [Cross-Forest Trust & Access Delegation](#lab-2-cross-forest-trust--access-delegation) | Selective Authentication forest trust, layered cross-forest permission delegation |
| 3 | [Identity Migration with ADMT](#lab-3-identity-migration-with-admt) | SID history-preserving user migration, migration tooling deployment |
| 4 | [Migration Validation & Decommission](#lab-4-migration-validation--decommission) | Post-migration validation, trust teardown, security cleanup |

**Environment:** Azure resource groups `AD-Lab-RG` (acquiring company, `lab.local`) and `AcquiredCo-RG` (acquired company, `acquiredco.local`) · North Central US · DC02 (existing domain controller) · ACQ-DC01 (new domain controller) · ADMT01 (dedicated migration tooling server)

---

## Lab 1: Acquired Environment & Network Integration

### Objective
Provision a fully independent Active Directory forest representing an acquired company, then establish network-level connectivity to the existing production domain via VNet peering and bidirectional DNS conditional forwarding.

### What I Built
- Deployed a new, isolated VNet and resource group with a non-overlapping address space, and promoted a new domain controller into an entirely separate forest (`acquiredco.local`)
- Created a representative OU structure and test identity to migrate later in the series
- Established bidirectional VNet peering between the acquiring and acquired company's networks
- Configured conditional DNS forwarding in both directions and validated full domain controller locator functionality (A and SRV records) across the peered networks

### Key Concepts
- Non-overlapping address space planning as a prerequisite for VNet peering
- Bidirectional VNet peering as two independently-configured objects
- DNS conditional forwarding and SRV record dependency for cross-domain AD functionality

### Skills Demonstrated
`Active Directory forest provisioning` `Azure VNet peering` `DNS conditional forwarding` `Cross-network AD validation`

---

## Lab 2: Cross-Forest Trust & Access Delegation

### Objective
Establish a one-way forest trust reflecting the real security posture of two organizations mid-acquisition, then correctly delegate the layered permissions required for genuine cross-forest administrative access.

### What I Built
- Created a one-way, Selective Authentication forest trust (acquiring domain trusts the acquired domain), deliberately avoiding blanket forest-wide access before integration needs were validated
- Diagnosed and resolved a multi-layered access-denial chain when attempting cross-forest administration, identifying and correctly configuring three independent, non-substitutable permission layers: trust-level "Allowed to Authenticate," AD group membership, and GPO-based logon rights
- Delegated access to a security group rather than an individual account, avoiding a fragile single-account dependency
- Validated the trust with a live, authenticated cross-forest directory query rather than relying on GUI status alone

### Key Concepts
- Selective vs. Forest-wide authentication trade-offs
- The three distinct permission layers required for cross-forest interactive/remote access to a domain controller
- Group-based vs. individual-account permission delegation

### Skills Demonstrated
`Forest trust configuration` `Selective Authentication` `Cross-forest permission delegation` `Systematic access-denial troubleshooting`

---

## Lab 3: Identity Migration with ADMT

### Objective
Deploy a dedicated migration tooling server and use the Active Directory Migration Tool to migrate a user identity from the acquired domain into the acquiring domain with SID history preserved.

### What I Built
- Provisioned a dedicated, domain-joined migration server, working within real Azure subscription quota constraints by identifying and fully cleaning up (not just deallocating) retired lab infrastructure, including orphaned disks, NICs, and public IPs left behind by prior VM deletions
- Configured all SID-history migration prerequisites on the source domain: registry-level TCP/IP client support, audit policy for account management events, and delegated administrative rights for the migrating account
- Installed ADMT 3.2 and its SQL Server Express database dependency, and enabled SID history support on the trust itself
- Diagnosed a persistent "access denied" auditing verification failure that survived every documented prerequisite fix, tracing it to an additional, undocumented requirement for the migrating account's rights on the source domain
- Successfully executed a live user migration with zero errors, using ADMT's native in-wizard credential handling after ruling out external credential-switching approaches

### Key Concepts
- ADMT's registry, audit policy, and rights prerequisites for SID history migration
- SID history as a temporary migration bridge rather than a permanent state
- ADMT 3.2's dependency model (SQL Server backend, no auto-install) versus earlier tool versions

### Skills Demonstrated
`ADMT deployment and configuration` `SID history migration` `SQL Server Express` `Azure resource quota troubleshooting` `Root-cause prerequisite diagnosis`

---

## Lab 4: Migration Validation & Decommission

### Objective
Validate that the migrated identity retained continuity with its original account, then cleanly decommission the trust and its associated privileges once migration was complete.

### What I Built
- Validated the migrated account's SID history via PowerShell, confirming the original source-domain SID was preserved exactly rather than the account being freshly recreated
- Disabled the source account post-migration to eliminate a duplicate live identity
- Disabled SID history support on the trust once migration concluded, treating it as a time-bounded capability rather than a standing configuration, consistent with its privilege-escalation risk if left enabled indefinitely
- Fully removed the cross-forest trust and validated its removal from both domains
- Removed VNet peering as the final network-layer teardown step, while deliberately retaining the acquired domain's infrastructure for potential further use rather than fully decommissioning it

### Key Concepts
- Why SID history should be disabled once active migration concludes
- Full-lifecycle IT integration project structure: build → integrate → migrate → validate → decommission
- Trust removal and validation as a distinct step from disabling the capabilities it enabled

### Skills Demonstrated
`Post-migration identity validation` `Trust lifecycle management` `Security-conscious decommissioning` `PowerShell validation`

---

## Tech Stack

**Infrastructure:** Microsoft Azure (VNets, VNet Peering, Resource Groups) · Windows Server 2025
**Identity:** Active Directory Domain Services · Active Directory Domains and Trusts · Group Policy
**Migration Tooling:** Active Directory Migration Tool (ADMT) 3.2 · SQL Server Express
**Scripting & Validation:** PowerShell (`Get-ADTrust`, `Get-ADUser`, `netdom`, `nltest`, `Get-AzVirtualNetworkPeering`, `auditpol`)
**Documentation:** Notion (runbooks) · GitHub
