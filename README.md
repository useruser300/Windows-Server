# Windows Server Domain Lab

Practical documentation for building a corporate-style Active Directory domain environment using **Windows Server 2019**.

This lab was initially built using **VirtualBox**, then rebuilt using **Hyper-V** to support **Hyper-V Replica** for replication and disaster recovery validation.

---

## What's inside

- Windows Server 2019 concepts (AD / DNS / DHCP / GPO)
- Lab blueprint (environment variables and naming/IP rules)
- Build the full lab using Hyper-V (VM creation plan + network layout)
- Domain setup (DC1 GUI + DC2 Core + Windows 10 join)
- Corporate structure (OUs, Groups, Users)
- Account management and security policies
- Group Policy catalog (baseline + OU targeted controls)
- File sharing (SMB permissions + drive mapping)
- DNS setup (zones, records, transfers, MX)
- DHCP setup (scope, exclusions, backup, failover)
- WDS deployment (PXE for Windows 10)
- Hyper-V replication (disaster recovery)
- Printer sharing and deployment
- Full server backup (Windows Server Backup)

---

## How to follow the documentation

### 1) Start with the fundamentals
- `00-windows-server-concepts.md`

### 2) Lab blueprint (must stay consistent)
- `01-lab-blueprint-environment-variables.md`

### 3) Optional: Build the full lab using Hyper-V (from zero)
If you want to create the whole environment using virtual machines first, follow this file before starting the domain setup:

- `appendix-a-lab-build-hyperv.md`

### 4) Build the domain foundation
- `02-domain-setup-dc1-dc2-win10.md`

### 5) Build the corporate identity structure
- `03-corporate-structure-ous-groups-users.md`

### 6) Core operations and policies
- `04-account-management-operations-security-policies.md`
- `05-group-policy-catalog-baseline-ou-controls.md`

### 7) Core infrastructure services
- `06-file-sharing-smb-permissions-drive-mapping.md`
- `07-dns-corporate-setup-zones-records-mx.md`
- `08-dhcp-corporate-setup-scope-failover-backup.md`

### 8) Deployment and advanced components
- `09-wds-pxe-deployment-windows10.md`
- `10-hyperv-replication-disaster-recovery.md`
- `11-printers-shared-deployment-policies.md`
- `12-backup-full-server-windows-server-backup.md`

