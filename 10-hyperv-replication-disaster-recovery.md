# Hyper-V & Replication — Install + VM + Hyper-V Replica

This section installs Hyper-V and configures **Hyper-V Replica** using a production-style design between **dedicated Hyper-V hosts** (not Domain Controllers).

Recommended Hyper-V Replica design (Primary → Replica):
- **Primary Hyper-V Host:** `SRV-HV1` (`192.168.10.30`)
- **Replica Hyper-V Host:** `SRV-HV2` (`192.168.10.31`)

Note:
It is not recommended in production to run Hyper-V on Domain Controllers.  
This lab uses Hyper-V Replica as a portfolio demonstration for disaster recovery and virtualization concepts.

Hyper-V Replica is used in enterprise environments to:
- Protect virtual machines against host failure
- Maintain service continuity with minimal downtime
- Replicate VMs between two Hyper-V hosts inside the same domain

All values align with the Lab Profile (Environment Variables):
- Domain: `iq.networks`
- Subnet: `192.168.10.0/24`
- Gateway: `192.168.10.1`
- DNS Primary (DC1): `192.168.10.10`
- DNS Secondary (DC2): `192.168.10.12`
- Hyper-V Primary Host: `SRV-HV1` (`192.168.10.30`)
- Hyper-V Replica Host: `SRV-HV2` (`192.168.10.31`)

---

## Hyper-V Hosts Setup (SRV-HV1 + SRV-HV2) — Static IP + DNS + Domain Join (Reference)

This section prepares the dedicated Hyper-V hosts before enabling Hyper-V Replica.

Hosts used in this lab:
- `SRV-HV1` (Primary Hyper-V Host): `192.168.10.30`
- `SRV-HV2` (Replica Hyper-V Host): `192.168.10.31`

These are Hyper-V servers/hosts (not Domain Controllers). They run virtualization workloads and replicate VMs between each other.

### Configure Static IP + DNS on SRV-HV1

On `SRV-HV1`, identify the adapter:

```powershell
Get-NetAdapter
```

Set static IP (adjust InterfaceAlias if needed):

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.30 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10,192.168.10.12
```

Verify:

```cmd
ipconfig /all
```

### Configure Static IP + DNS on SRV-HV2

On `SRV-HV2`:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.31 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10,192.168.10.12
```

Verify:

```cmd
ipconfig /all
```

### Rename the Servers (Recommended)

On `SRV-HV1`:

```powershell
Rename-Computer -NewName "SRV-HV1" -Restart
```

On `SRV-HV2`:

```powershell
Rename-Computer -NewName "SRV-HV2" -Restart
```

### Join SRV-HV1 and SRV-HV2 to the Domain (iq.networks)

On `SRV-HV1`:

```powershell
Add-Computer -DomainName "iq.networks" -Credential (Get-Credential)
Restart-Computer
```

On `SRV-HV2`:

```powershell
Add-Computer -DomainName "iq.networks" -Credential (Get-Credential)
Restart-Computer
```

### Basic Connectivity + DNS Validation (Must Pass)

From `SRV-HV1`:

```cmd
ping SRV-HV2
ping 192.168.10.31
nslookup SRV-HV2
```

From `SRV-HV2`:

```cmd
ping SRV-HV1
ping 192.168.10.30
nslookup SRV-HV1
```

Expected:

* Ping works by name and by IP
* `nslookup` resolves correctly using `192.168.10.10` and/or `192.168.10.12`

### Time Sync Note (Kerberos Requirement)

Hyper-V Replica with Kerberos depends on correct time sync.

Validate:

```cmd
w32tm /query /status
```

If hosts are domain-joined, time should sync automatically from the domain.

---

## Prerequisites

Before enabling replication, ensure:

Domain membership (Kerberos auth):

* `SRV-HV1 ∈ iq.networks`
* `SRV-HV2 ∈ iq.networks`

DNS name resolution works:

* `SRV-HV1 ↔ SRV-HV2`

Time synchronization is correct (Kerberos)

Storage is available on `SRV-HV2` for replica files (VMs can be large)

Network connectivity allows replication traffic (firewall + connectivity)

---

## Install Hyper-V Role on SRV-HV1

On `SRV-HV1`, run:

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

Result:

* Hyper-V role is installed
* Server restarts automatically

---

## Install Hyper-V Role on SRV-HV2 (Replica Host)

On `SRV-HV2`, run:

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

Result:

* Replica host is ready to receive and manage replicated VMs

---

## Create a Virtual Switch (Required for VM Networking)

Hyper-V VMs connect to a Virtual Switch.

### Check existing network adapters

On `SRV-HV1`:

```powershell
Get-NetAdapter
```

### Create an External Virtual Switch

Example using adapter name `Ethernet`:

```powershell
New-VMSwitch -Name "vSwitch-LAN" -NetAdapterName "Ethernet" -AllowManagementOS $true
```

Explanation:

* External switch connects VMs to the real network
* `-AllowManagementOS $true` keeps the host connected through the same NIC

Verify:

```powershell
Get-VMSwitch
```

Best practice:
Create the same vSwitch name on `SRV-HV2` too if you plan to run/failover the VM there:

```powershell
New-VMSwitch -Name "vSwitch-LAN" -NetAdapterName "Ethernet" -AllowManagementOS $true
```

---

## Create a Virtual Machine (CoreVM)

Now we create a test VM that will be replicated.

On `SRV-HV1`:

```powershell
New-VM -Name "CoreVM" `
  -MemoryStartupBytes 2GB `
  -NewVHDPath "C:\VMs\CoreVM\CoreVM.vhdx" `
  -NewVHDSizeBytes 20GB `
  -SwitchName "vSwitch-LAN"
```

Optional verification:

```powershell
Get-VM
```

Result:

* A VM named `CoreVM` is created and attached to the network.

Technical note:
This creates an empty VM + disk. You still need to install an OS (ISO) if you want it bootable.
For replication testing, an installed OS gives a more realistic validation.

---

## Enable Hyper-V Replica (Configure the Replica Server)

Before replication, `SRV-HV2` must be configured to accept replicas.

### 12.6.1 Enable this computer as a Replica server (GUI)

On `SRV-HV2`:

1. Open **Hyper-V Manager**
2. Right-click the server → **Hyper-V Settings**
3. Open **Replication Configuration**
4. Enable:

   * **Enable this computer as a Replica server**
5. Authentication:

   * **Kerberos (HTTP)** (recommended inside a domain)
6. Allow replication from:

   * **Any authenticated server**
   * OR specify only `SRV-HV1` for stricter security
7. Default storage location (example):

   * `D:\HyperV\Replica`
8. Apply → OK

Result:
`SRV-HV2` is ready to accept Hyper-V replication.

---

## Configure VM Replication (Primary → Replica)

Now enable replication for VM `CoreVM` on `SRV-HV1`.

### 12.7.1 Enable replication for CoreVM (GUI)

On `SRV-HV1`:

1. Open **Hyper-V Manager**
2. Select VM: `CoreVM`
3. Right-click → **Enable Replication**
4. Replica server:

   * `SRV-HV2`
5. Authentication:

   * **Kerberos (HTTP)**
6. Replication frequency:

   * **5 minutes** (good for labs)
7. Initial replication method:

   * **Send initial copy over the network** (lab friendly)
8. Finish

Result:
Replication for `CoreVM` is enabled and initial replication begins.

If the wizard fails, it is usually one of these:

* DNS/time mismatch (Kerberos)
* Firewall rules not enabled
* Replica server not configured to accept replication
* Wrong virtual switch/network mismatch for failover testing

---

## Verification (Confirm Hyper-V Replica works)

### Check replication state from PowerShell

On `SRV-HV1`:

```powershell
Get-VMReplication -VMName "CoreVM"
```

Expected:

* `ReplicationState`: Normal / Replicating
* `Health`: Normal

### Confirm replica exists on SRV-HV2

On `SRV-HV2` (Hyper-V Manager):

* You should see `CoreVM` as a **Replica VM**

---

## Test Failover (Optional Lab Validation)

Hyper-V Replica allows a safe test failover to validate recovery.

On `SRV-HV2`:

1. Hyper-V Manager
2. Right-click `CoreVM` (Replica)
3. Choose:

   * **Test Failover**
4. Select a recovery point
5. Start the test VM

Result:

* Replica VM boots successfully for validation

Important:

* Test failover does **not** break production replication
* It creates a temporary test VM instance

---

## Firewall Note (Replication Traffic)

Hyper-V Replica requires firewall rules for replication traffic.

For Kerberos (HTTP), ensure the built-in Hyper-V Replica firewall rules are enabled.
If replication does not work, verify:

* Windows Firewall rules on both hosts
* Host-to-host connectivity (ping + DNS)
* Domain membership + time sync

hint:
If you want a quick baseline check on each host:

```powershell
Get-NetFirewallRule | Where-Object DisplayName -Like "*Hyper-V Replica*" | Select DisplayName, Enabled
```

---

## Operational Best Practice Notes

* Hyper-V Replica is a **disaster recovery** mechanism, not a backup replacement
* Replication does not protect against data corruption that also replicates to the replica
* In production, Hyper-V hosts are dedicated servers separated from AD Domain Controllers
* This lab demonstrates:

  * Virtualization resiliency
  * High availability mindset
  * Controlled failover validation (portfolio-ready skill)
