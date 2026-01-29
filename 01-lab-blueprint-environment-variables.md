# Lab Blueprint (Environment Variables)

These variables must stay consistent across the entire documentation.

## Core Network Settings
- **Domain Name (FQDN):** `iq.networks`
- **NetBIOS Name:** `iq`
- **Subnet:** `192.168.10.0/24`
- **Default Gateway:** `192.168.10.1`

---

## Domain Controllers (Active Directory)
- **Primary Domain Controller (GUI):** `SRV-DC1-GUI` — `192.168.10.10`
- **Secondary Domain Controller (Core):** `SRV-DC2-CORE` — `192.168.10.12`

---

## DNS Configuration
- **DNS Primary (DC1):** `192.168.10.10`
- **DNS Secondary (DC2):** `192.168.10.12`

---

## DHCP Configuration
- **DHCP Server:** `SRV-DC1-GUI`
- **DHCP Scope Range:** `192.168.10.150 - 192.168.10.200`
- **DHCP Exclusion Range:** `192.168.10.90 - 192.168.10.95`

---

## Client Machines
- **Windows 10 Client:** `WIN10-CL1`

---

## Hyper-V Replication Hosts (Production-Style Design)
- **Hyper-V Host 1 (Primary):** `SRV-HV1` — `192.168.10.30`
- **Hyper-V Host 2 (Replica):** `SRV-HV2` — `192.168.10.31`

---

## Rule
Any IP address, hostname, or domain name mentioned anywhere in this documentation must match the values defined in this file.
