## Build the Lab using Hyper-V (VM Creation Plan + Network Layout)

This optional section explains how the entire lab environment can be built **from zero** using **Hyper-V virtual machines**.

If you already have your servers installed and ready (physical or virtual), you can skip this section and start directly from:

- **Domain Setup (DC1 GUI + DC2 CORE + Join WIN10)**

This section is included because many real corporate labs and test environments are built on virtualization platforms such as:

- Hyper-V
- VMware
- Proxmox

In this lab, Hyper-V is used to create and run:

- `SRV-DC1-GUI` (Windows Server 2019 GUI) — Primary Domain Controller
- `SRV-DC2-CORE` (Windows Server 2019 Core) — Secondary Domain Controller
- `SRV-HV1` (Windows Server 2019) — Primary Hyper-V Host (Replica Source)
- `SRV-HV2` (Windows Server 2019) — Replica Hyper-V Host (Replica Target)
- `WIN10-CL1` (Windows 10) — Domain Client

All values align with the Unified Lab Profile:

- Domain: `iq.networks`
- Subnet: `192.168.10.0/24`
- Gateway: `192.168.10.1`
- DNS Primary (DC1): `192.168.10.10`
- DNS Secondary (DC2): `192.168.10.12`

---

### A.1 Host Machine Requirements (Hyper-V Host)

To run multiple virtual machines smoothly, the host system should have:

Recommended minimum for a full lab:

- CPU: 4 cores (8 threads preferred)
- RAM: 16 GB (minimum), 32 GB (better)
- Disk: SSD recommended for performance
- Windows: Hyper-V supported edition (Windows Server or Windows 10/11 Pro/Enterprise)

---

### A.2 Unified Lab Network Layout (Virtual Network Design)

All machines must be inside the same subnet:

- Subnet: `192.168.10.0/24`
- Gateway: `192.168.10.1`

Static infrastructure IPs:

- `SRV-DC1-GUI` (DNS Primary / PDC) → `192.168.10.10`
- `SRV-DC2-CORE` (DNS Secondary / ADC) → `192.168.10.12`
- `SRV-HV1` (Primary Hyper-V Host) → `192.168.10.30`
- `SRV-HV2` (Replica Hyper-V Host) → `192.168.10.31`

Client:

- `WIN10-CL1` → DHCP (from scope `192.168.10.150 - 192.168.10.200`)

Important rule:  
All VMs must connect to the same Virtual Switch so they can communicate.

---

### A.3 Create the Virtual Switch (vSwitch-LAN)

A Virtual Switch is required so VMs can connect to the network.

#### A.3.1 List available adapters (host)

On the Hyper-V Host:

```powershell
Get-NetAdapter
```

Example output may include:
- Ethernet
- Wi-Fi

#### A.3.2 Create an External Virtual Switch

External switch connects VMs to the physical network.

```powershell
New-VMSwitch -Name "vSwitch-LAN" -NetAdapterName "Ethernet" -AllowManagementOS $true
```

Explanation:
- `-Name "vSwitch-LAN"` → the virtual switch name
- `-NetAdapterName "Ethernet"` → must match your real adapter name
- `-AllowManagementOS $true` → host keeps network connectivity

Verify:

```powershell
Get-VMSwitch
```

Result:  
vSwitch-LAN exists and is ready for VM networking.

---

### A.4 VM Specs (Recommended Corporate Lab Specs)

These specs are balanced for:
- performance
- realism
- stability on a normal laptop/PC host

#### A.4.1 SRV-DC1-GUI (Primary Domain Controller)
- OS: Windows Server 2019 (GUI)
- Name: SRV-DC1-GUI
- RAM: 4 GB (recommended)
- CPU: 2 vCPU
- Disk: 60 GB
- Network: vSwitch-LAN

#### A.4.2 SRV-DC2-CORE (Secondary Domain Controller)
- OS: Windows Server 2019 (Core)
- Name: SRV-DC2-CORE
- RAM: 2 GB
- CPU: 2 vCPU
- Disk: 40 GB
- Network: vSwitch-LAN

#### A.4.3 SRV-HV1 (Primary Hyper-V Host)
- OS: Windows Server 2019 (GUI or Core)
- Name: SRV-HV1
- RAM: 4 GB (minimum), 8 GB (better)
- CPU: 2 vCPU (minimum), 4 vCPU (better)
- Disk: 80 GB (recommended for VM storage)
- Network: vSwitch-LAN

#### A.4.4 SRV-HV2 (Replica Hyper-V Host)
- OS: Windows Server 2019 (GUI or Core)
- Name: SRV-HV2
- RAM: 4 GB (minimum), 8 GB (better)
- CPU: 2 vCPU (minimum), 4 vCPU (better)
- Disk: 80 GB (recommended for replica storage)
- Network: vSwitch-LAN

#### A.4.5 WIN10-CL1 (Domain Client)
- OS: Windows 10
- Name: WIN10-CL1
- RAM: 4 GB
- CPU: 2 vCPU
- Disk: 60 GB
- Network: vSwitch-LAN

---

### A.5 Create Virtual Machines (PowerShell)

You can create the VMs quickly using PowerShell on the Hyper-V Host.

#### A.5.1 Create folders for VM storage (recommended)

```powershell
New-Item -Path "C:\VMs" -ItemType Directory
```

#### A.5.2 Create SRV-DC1-GUI VM

```powershell
New-VM -Name "SRV-DC1-GUI" `
  -Generation 2 `
  -MemoryStartupBytes 4GB `
  -NewVHDPath "C:\VMs\SRV-DC1-GUI\SRV-DC1-GUI.vhdx" `
  -NewVHDSizeBytes 60GB `
  -SwitchName "vSwitch-LAN"
```

#### A.5.3 Create SRV-DC2-CORE VM

```powershell
New-VM -Name "SRV-DC2-CORE" `
  -Generation 2 `
  -MemoryStartupBytes 2GB `
  -NewVHDPath "C:\VMs\SRV-DC2-CORE\SRV-DC2-CORE.vhdx" `
  -NewVHDSizeBytes 40GB `
  -SwitchName "vSwitch-LAN"
```

#### A.5.4 Create SRV-HV1 VM

```powershell
New-VM -Name "SRV-HV1" `
  -Generation 2 `
  -MemoryStartupBytes 4GB `
  -NewVHDPath "C:\VMs\SRV-HV1\SRV-HV1.vhdx" `
  -NewVHDSizeBytes 80GB `
  -SwitchName "vSwitch-LAN"
```

#### A.5.5 Create SRV-HV2 VM

```powershell
New-VM -Name "SRV-HV2" `
  -Generation 2 `
  -MemoryStartupBytes 4GB `
  -NewVHDPath "C:\VMs\SRV-HV2\SRV-HV2.vhdx" `
  -NewVHDSizeBytes 80GB `
  -SwitchName "vSwitch-LAN"
```

#### A.5.6 Create WIN10-CL1 VM

```powershell
New-VM -Name "WIN10-CL1" `
  -Generation 2 `
  -MemoryStartupBytes 4GB `
  -NewVHDPath "C:\VMs\WIN10-CL1\WIN10-CL1.vhdx" `
  -NewVHDSizeBytes 60GB `
  -SwitchName "vSwitch-LAN"
```

Verify all VMs:

```powershell
Get-VM
```

Result:  
All virtual machines exist and are connected to vSwitch-LAN.

---

### A.6 Enable Nested Virtualization (Required for SRV-HV1 / SRV-HV2)

Because SRV-HV1 and SRV-HV2 are Hyper-V hosts running as virtual machines, you must enable Nested Virtualization on the physical Hyper-V host.

Run these commands on the Physical Hyper-V Host (not inside the VM):

```powershell
Set-VMProcessor -VMName "SRV-HV1" -ExposeVirtualizationExtensions $true
Set-VMProcessor -VMName "SRV-HV2" -ExposeVirtualizationExtensions $true
```

Restart both VMs:

```powershell
Restart-VM "SRV-HV1"
Restart-VM "SRV-HV2"
```

Result:
SRV-HV1 and SRV-HV2 can now install and run Hyper-V workloads inside the VM.

Note: Nested Virtualization must be enabled while the VM is powered off. Shut down SRV-HV1/SRV-HV2 first, apply the setting, then start them again.


---

### A.7 ISO Mounting (Install OS on each VM)

To install Windows inside each VM, attach an ISO file.

You need ISO files such as:
- Windows Server 2019 ISO (for all Windows Server VMs)
- Windows 10 ISO (optional if you later install Windows 10 via WDS PXE)

ISO requirement for Windows Server VMs:
- SRV-DC1-GUI
- SRV-DC2-CORE
- SRV-HV1
- SRV-HV2

#### A.7.1 Attach Windows Server 2019 ISO to SRV-DC1-GUI

```powershell
Set-VMDvdDrive -VMName "SRV-DC1-GUI" -Path "C:\ISO\Windows_Server_2019.iso"
```

#### A.7.2 Attach Windows Server 2019 ISO to SRV-DC2-CORE

```powershell
Set-VMDvdDrive -VMName "SRV-DC2-CORE" -Path "C:\ISO\Windows_Server_2019.iso"
```

#### A.7.3 Attach Windows Server 2019 ISO to SRV-HV1

```powershell
Set-VMDvdDrive -VMName "SRV-HV1" -Path "C:\ISO\Windows_Server_2019.iso"
```

#### A.7.4 Attach Windows Server 2019 ISO to SRV-HV2

```powershell
Set-VMDvdDrive -VMName "SRV-HV2" -Path "C:\ISO\Windows_Server_2019.iso"
```

#### A.7.5 Attach Windows 10 ISO to WIN10-CL1 (optional)

```powershell
Set-VMDvdDrive -VMName "WIN10-CL1" -Path "C:\ISO\Windows_10.iso"
```

Note:  
If you plan to deploy Windows 10 using WDS later, you can skip mounting the Windows 10 ISO to the client VM and deploy it using PXE.

---

### A.8 Boot Order (UEFI) and PXE Readiness

To support PXE boot later for WIN10-CL1, ensure:
- Network boot is enabled
- VM firmware boot order allows network boot

On Hyper-V Manager:
- VM Settings → Firmware → Boot order
- Ensure Network Adapter is available as a boot option

---

### A.9 Start VMs and Install Operating Systems

Start each VM:

```powershell
Start-VM -Name "SRV-DC1-GUI"
Start-VM -Name "SRV-DC2-CORE"
Start-VM -Name "SRV-HV1"
Start-VM -Name "SRV-HV2"
Start-VM -Name "WIN10-CL1"
```

Then open VM console using Hyper-V Manager and install:
- Windows Server 2019 GUI on SRV-DC1-GUI
- Windows Server 2019 Core on SRV-DC2-CORE
- Windows Server 2019 on SRV-HV1
- Windows Server 2019 on SRV-HV2
- Windows 10 on WIN10-CL1 (only if not using PXE yet)

---

### A.10 Initial OS Configuration (Step-by-Step)

This section performs the required **initial operating system configuration** before starting the domain setup.
It ensures all machines have:

- Correct hostname
- Correct static IP plan (for servers)
- Correct DNS configuration (mandatory for Active Directory)
- Basic connectivity verification

All values must match the Unified Lab Profile:

- Domain: `iq.networks`
- Subnet: `192.168.10.0/24`
- Gateway: `192.168.10.1`
- DNS Primary: `192.168.10.10`
- DNS Secondary: `192.168.10.12`

---

#### A.10.1 Windows Server GUI — Static IP + DNS (SRV-DC1-GUI / SRV-HV1 / SRV-HV2)

This method is used for **Windows Server GUI** machines where you can run PowerShell easily.

First: Identify the network adapter

```powershell
Get-NetAdapter
Get-NetIPAddress
```

You will need one of these values:
- `InterfaceIndex`
or
- `InterfaceAlias`

---

##### A.10.1.1 SRV-DC1-GUI (Primary Domain Controller) — Static IP + DNS

Target configuration:
- Hostname: `SRV-DC1-GUI`
- IP: `192.168.10.10/24`
- Gateway: `192.168.10.1`
- DNS: `192.168.10.10` (and optionally `192.168.10.12` after DC2 is ready)

Rename (recommended):
```powershell
Rename-Computer -NewName "SRV-DC1-GUI" -Restart
```

After reboot, configure IP + DNS (replace `<IF_INDEX>` with your real value):
```powershell
New-NetIPAddress -InterfaceIndex <IF_INDEX> -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceIndex <IF_INDEX> -ServerAddresses 192.168.10.10
```

Verify:
```cmd
ipconfig /all
ping 192.168.10.1
```

Important note:
- On the Domain Controller, DNS must always point to the DC DNS service.
- Do NOT set the DNS to public DNS like `8.8.8.8`.

---

##### A.10.1.2 SRV-HV1 (Hyper-V Host 1) — Static IP + DNS

Target configuration:
- Hostname: `SRV-HV1`
- IP: `192.168.10.30/24`
- Gateway: `192.168.10.1`
- DNS: `192.168.10.10`, `192.168.10.12`

Rename:
```powershell
Rename-Computer -NewName "SRV-HV1" -Restart
```

After reboot:
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.30 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10,192.168.10.12
```

Verify:
```cmd
ipconfig /all
ping 192.168.10.10
nslookup SRV-DC1-GUI
```

---

##### A.10.1.3 SRV-HV2 (Hyper-V Host 2) — Static IP + DNS

Target configuration:
- Hostname: `SRV-HV2`
- IP: `192.168.10.31/24`
- Gateway: `192.168.10.1`
- DNS: `192.168.10.10`, `192.168.10.12`

Rename:
```powershell
Rename-Computer -NewName "SRV-HV2" -Restart
```

After reboot:
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.31 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10,192.168.10.12
```

Verify:
```cmd
ipconfig /all
ping 192.168.10.12
nslookup SRV-DC2-CORE
```

---

#### A.10.2 Windows Server Core — Static IP + DNS (SCONFIG) (SRV-DC2-CORE)

This method is used for **Windows Server Core**, where the recommended setup is via `sconfig`.

Target configuration:
- Hostname: `SRV-DC2-CORE`
- IP: `192.168.10.12/24`
- Gateway: `192.168.10.1`
- DNS: `192.168.10.10` (primary)

---

##### A.10.2.1 Open Server Configuration Tool

```text
sconfig
```

---

##### A.10.2.2 Set Static IP Address

1. Select:
   - `8) Network Settings`
2. Select:
   - `1) Network Adapter Settings`
3. Select your adapter (example: `1`)
4. Select:
   - `1) Set Network Adapter Address`
5. Choose:
   - `(S) Static`

Enter:
- IP Address: `192.168.10.12`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.10.1`

---

##### A.10.2.3 Set DNS Server Address

1. Inside Network Adapter Settings, select:
   - `2) Set DNS Servers`
2. Enter:
   - Primary DNS: `192.168.10.10`
3. (Optional) Secondary DNS:
   - `192.168.10.12` (only after SRV-DC2-CORE becomes a DNS server)

---

##### A.10.2.4 Rename the Server (Recommended)

Inside `sconfig`:
1. Select:
   - `2) Computer Name`
2. Rename to:
   - `SRV-DC2-CORE`
3. Reboot

---

##### A.10.2.5 Verify Core Networking

Run:
```cmd
ipconfig /all
ping 192.168.10.10
nslookup iq.networks
hostname
```

Expected:
- Core can ping DC1
- DNS lookup works
- Hostname is correct

---

#### A.10.3 Windows 10 — DNS / DHCP Verification (WIN10-CL1)

The client should either:
- use DHCP (recommended after DHCP setup)
or
- use a manual static IP temporarily during early setup

Minimum requirement before domain join:
DNS must point to the Domain Controller DNS.

---

##### A.10.3.1 Rename the Client (Recommended)

On Windows 10:
- Rename computer name to: `WIN10-CL1`
- Restart the machine

---

##### A.10.3.2 Verify Current IP and DNS settings

Run CMD:

```cmd
ipconfig /all
```

Confirm:
- DNS Server = `192.168.10.10` (and optionally `192.168.10.12`)
- IP is correct based on your method:
  - static (temporary)
  - DHCP (192.168.10.150 - 192.168.10.200)

---

##### A.10.3.3 Test Connectivity + Name Resolution

```cmd
ping 192.168.10.10
ping SRV-DC1-GUI
nslookup iq.networks
```

Expected:
- Ping works
- DNS resolves names correctly

---

#### A.10.4 Verification Commands (Must Pass Before Domain Setup)

Run these checks before continuing to the Domain Setup section.

---

##### A.10.4.1 Check Hostnames

On each machine:
```cmd
hostname
```

Expected names:
- `SRV-DC1-GUI`
- `SRV-DC2-CORE`
- `SRV-HV1`
- `SRV-HV2`
- `WIN10-CL1`

---

##### A.10.4.2 Check IP Configuration

```cmd
ipconfig /all
```

Expected static IPs:
- SRV-DC1-GUI → `192.168.10.10`
- SRV-DC2-CORE → `192.168.10.12`
- SRV-HV1 → `192.168.10.30`
- SRV-HV2 → `192.168.10.31`

WIN10-CL1:
- either DHCP
- or static temporarily

---

##### A.10.4.3 Check DNS Resolution

On servers:
```cmd
nslookup iq.networks
nslookup SRV-DC1-GUI
```

---

##### A.10.4.4 Check Network Connectivity

```cmd
ping 192.168.10.1
ping 192.168.10.10
ping 192.168.10.12
```

If any ping fails:
- check vSwitch connection
- check Windows Firewall (temporary test)
- check DNS configuration
- check IP address correctness

---

### A.11 Final Result 

At this stage you have a complete virtual environment:
- VMs exist and are bootable
- VMs are connected to vSwitch-LAN
- IP plan matches the unified lab profile
- Domain infrastructure IPs are ready (DC + DNS)
- Hyper-V hosts are ready for Replica configuration



