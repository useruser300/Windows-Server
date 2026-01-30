## DHCP (Install + Scope + Options + Exclusions + Backup + Failover)

This section provides a corporate-style DHCP implementation for the domain network.  
It covers:

- DHCP base setup and why it is used in companies
- Scope design aligned with network planning
- Scope options (DNS + Default Gateway + Lease)
- Exclusion ranges to protect reserved IPs
- Operational maintenance (editing scope settings, updating gateway/DNS options)
- Backup and restore (disaster recovery + branch replication use cases)
- DHCP Failover (High Availability) using Hot Standby, including Windows Server Core considerations

All values are aligned with the Environment Variables:

- Subnet: `192.168.10.0/24`
- Gateway: `192.168.10.1`
- DNS Primary: `192.168.10.10`
- DNS Secondary: `192.168.10.12`
- Scope: `192.168.10.150 - 192.168.10.200`
- Exclusion: `192.168.10.90 - 192.168.10.95`

---

### Why DHCP is important in companies

Many companies prefer to run **DNS and DHCP together**, especially with Active Directory.  
DHCP automatically assigns IP addresses to computers, so you do not need to configure static IP settings on every device.

This makes network management easier because:
- Clients receive IP settings automatically
- DNS and gateway settings are standardized centrally
- IP conflicts are reduced through structured scope planning

---

## DHCP Base Setup (Install Role + Management Tools)

### Install DHCP Server role using PowerShell (Recommended)

On the server where DHCP will run (commonly `SRV-DC1-GUI`), run:

```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

Install using Server Manager (GUI alternative)
1. Open Server Manager
2. Go to:
    * Manage → Add Roles and Features
3. Select:
    * DHCP Server
4. Finish the wizard and install

This installs:
* The DHCP Server role
* The management tools needed to configure DHCP

Confirm installation success

Verification:
* Open Server Manager and confirm DHCP appears in the roles list

PowerShell verification:
```powershell
Get-WindowsFeature DHCP
```

Decide IPv4, IPv6, or both

You can configure DHCP for:
* IPv4 only
* IPv6 only
* both

In this lab design, the scope example is IPv4-focused.

---

## Scope Design (Corporate planning before configuration)

Define the scope IP range

A DHCP scope is the pool of IP addresses that DHCP distributes to clients.  
For this environment:
* Network: 192.168.10.0/24
* Subnet mask: 255.255.255.0
* DHCP range:
    * Start: 192.168.10.150
    * End: 192.168.10.200

Plan reserved IP addresses before choosing the range

Before defining the DHCP range, plan which IPs are reserved for servers and infrastructure.  
Do not include reserved IPs inside the DHCP range.

Examples of reserved static IPs:
* DNS Server / Domain Controllers
* Web servers
* Printers
* Infrastructure servers

Example reservation plan:
* Reserve 192.168.10.1 up to 192.168.10.20 for servers and important devices
* Then start the DHCP scope after that reserved block

This prevents conflicts and keeps the network organized.

---

## Create the DHCP Scope (IPv4)

Create a new DHCP scope with PowerShell
```powershell
Add-DhcpServerV4Scope -Name "DHCP Scope" -StartRange 192.168.10.150 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0
```

Result:  
The scope exists and can distribute addresses within 192.168.10.150 to 192.168.10.200.

---

## Configure Scope Options (DNS + Router/Gateway + Lease)

After creating the scope, configure options that clients must receive:
* DNS Server addresses
* Default Gateway (Router) address
* Lease duration

Set DNS Server and Router options  

Important note:  
Set-DhcpServerv4OptionValue should usually be applied to a specific scope using -ScopeId.  
Without -ScopeId, you are configuring server-level options, not scope-level options.

Recommended (scope-level) command:
```powershell
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -DnsServer 192.168.10.10,192.168.10.12 -Router 192.168.10.1
```

Why DNS option matters:
* Ensures all clients use the internal DNS servers:
    * Primary: 192.168.10.10
    * Secondary: 192.168.10.12

Why Router option matters:
* Ensures all clients get the correct default gateway:
    * 192.168.10.1

Note (server-level vs scope-level):
* If you set options at Server Options, they apply to all scopes on that DHCP server.
* If you set options at Scope Options, they apply only to that scope.

---

## Configure the lease duration

Example lease duration: 8 hours
```powershell
Set-DhcpServerV4Scope -ScopeId 192.168.10.0 -LeaseDuration 8.00:00:00
```

---

## Exclusion Range (Protect reserved/critical IPs)

Why exclusions exist

Exclusion ranges prevent DHCP from handing out IPs that must stay reserved.  
They protect:
* infrastructure IPs
* addresses that must never be assigned dynamically

Apply the exclusion range (GUI method)

Exclusion range in this lab:
* 192.168.10.90 - 192.168.10.95

Steps:
1. Open DHCP from:
    * Tools → DHCP
2. Expand:
    * DHCP Server → IPv4 → (your scope)
3. Right-click:
    * Address Pool
4. Choose:
    * New Exclusion Range
5. Add:
    * Start: 192.168.10.90
    * End: 192.168.10.95
6. Apply → OK

Result:  
DHCP will not assign any IP between 192.168.10.90 and 192.168.10.95.

Optional PowerShell method:
```powershell
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.10.0 -StartRange 192.168.10.90 -EndRange 192.168.10.95
```

---

## Verify DHCP configuration (Console validation)

Open the DHCP Management Console

Open DHCP from:
* Server Manager  
or
* Tools → DHCP

You will see:
* IPv4
* IPv6 sections

Verify the IPv4 scope configuration

Inside IPv4:
1. Confirm the scope exists
2. Confirm Address Pool range:
    * 192.168.10.150 to 192.168.10.200
3. Verify Scope Options:
    * Router: 192.168.10.1
    * DNS Servers: 192.168.10.10 and 192.168.10.12

PowerShell verification:
```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
```

Final result:  
DHCP is active and ready to automatically assign IP settings to clients.

---

## Operational Changes (Gateway + DNS Options Maintenance)

Change the Default Gateway (Router option)

Why this happens in companies:
* Multiple routers
* Router failover
* Gateway migration

Steps:
1. Open DHCP:
    * Tools → DHCP
2. Go to:
    * IPv4 → Server Options (or Scope Options)
3. Find:
    * Router (Default Gateway)

Method A: Configure Options
1. Right-click Server Options or Scope Options
2. Select Configure Options
3. Select Router
4. Add / remove gateway IPs
5. Apply → OK

Method B: Edit Router directly
1. Right-click Router
2. Choose Properties
3. Remove the old IP, add the new IP, Apply → OK

---

## Change or Add DNS Servers in DHCP options

Why this happens:
* DNS server replacement
* Adding a secondary DNS for redundancy

Steps:
1. Open DHCP:
    * Tools → DHCP
2. Go to:
    * IPv4 → Server Options (or Scope Options)
3. Find:
    * DNS Servers

Method A: Edit DNS Servers directly
1. Right-click DNS Servers
2. Properties
3. Add / remove DNS IPs
4. Apply → OK

Method B: Configure Options
1. Right-click Server Options
2. Configure Options
3. Select DNS Servers
4. Edit values
5. Apply → OK

lab target:
* Primary: 192.168.10.10
* Secondary: 192.168.10.12

---

## Edit Scope Settings (Range and Dynamic DNS update behavior)

Steps:
1. Open DHCP:
    * Tools → DHCP
2. Expand your server
3. Go to IPv4
4. Right-click the scope → Properties

Inside scope properties:
* You can change the IP range
* You can review DNS behavior (Dynamic DNS updates)
* You can adjust advanced options

Dynamic DNS update behavior:
* DHCP can automatically update DNS records when clients receive leases (depending on settings)

Save changes:
* Click OK, then re-open properties to confirm changes were applied.

---

## Backup and Restore DHCP Server Configuration

Why DHCP backup is important

DHCP can fail due to:
* server shutdown
* disk failure
* database corruption
* accidental deletion

Backup allows recovery of:
* scope configuration
* reservations (if used)
* options (DNS/router)
* DHCP database settings

Also useful for branch replication:
* clone the same DHCP configuration to a new branch

---

## Backup DHCP configuration (GUI)

Steps:
1. Open DHCP:
    * Tools → DHCP
2. Right-click the DHCP server → Backup
3. Choose the backup location
4. Confirm

Result:  
Backup is created in the selected location.

Optional PowerShell backup method:
```powershell
Backup-DhcpServer -ComputerName SRV-DC1-GUI -Path "C:\Backup\DHCP"
```

---

## Restore DHCP configuration (GUI)

Steps:
1. Right-click the DHCP server → Restore
2. Select the backup folder
3. Confirm restore:
    * Click Yes
4. Verify the scope appears again under IPv4

Optional PowerShell restore method:
```powershell
Restore-DhcpServer -ComputerName SRV-DC1-GUI -Path "C:\Backup\DHCP"
```

---

## DHCP Failover (High Availability) — Hot Standby (Core + GUI)

Why DHCP failover is used

DHCP is essential for client connectivity.  
If DHCP fails:
* new clients cannot get IP addresses
* existing clients may fail to renew leases

High Availability ensures service continuity.

---

## Core requirements and environment

* Primary DHCP: SRV-DC1-GUI
* Failover partner: SRV-DC2-CORE
* Mode: Hot Standby (one active, one standby)

Failover can be configured via:
* DHCP console:
    * DHCP → Action → Configure Failover  
or
* PowerShell

---

## Install DHCP on both servers

On SRV-DC1-GUI:
```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

On SRV-DC2-CORE:
```powershell
Install-WindowsFeature -Name DHCP
```

Result:  
Both servers now have the DHCP role installed.

---

## Configure DHCP Failover on the primary server (Hot Standby)

Important note:  
For failover to work correctly, the DHCP server must be authorized in Active Directory.

GUI authorization:
* DHCP console → right-click server → Authorize

PowerShell method:
```powershell
Add-DhcpServerInDC -DnsName "SRV-DC1-GUI.iq.networks" -IpAddress 192.168.10.10
Add-DhcpServerInDC -DnsName "SRV-DC2-CORE.iq.networks" -IpAddress 192.168.10.12
```

Create the failover relationship:
```powershell
Add-DhcpServerv4Failover `
  -ComputerName "SRV-DC1-GUI" `
  -PartnerServer "SRV-DC2-CORE" `
  -Name "Failover-HotStandby" `
  -SharedSecret "YourSecretKey" `
  -Mode HotStandby `
  -ScopeId 192.168.10.0
```

Replace:
* YourSecretKey with a real shared secret.

Result:  
Failover is configured and standby server can take over if the primary fails.

---

## Failover Verification (Recommended)

On the primary server:
```powershell
Get-DhcpServerv4Failover
Get-DhcpServerv4Failover -Name "Failover-HotStandby"
```

From a client:
* Confirm IP range:
    * 192.168.10.150 - 192.168.10.200
* Confirm DNS and gateway:
    * DNS: 192.168.10.10, 192.168.10.12
    * Gateway: 192.168.10.1

Optional test:
* Stop DHCP service on the primary and verify renew works.

---

## Operational best practice summary

* Use scope-level options (recommended) to avoid unintended server-wide changes
* Always include -ScopeId in PowerShell
* Plan scope ranges and reserved IPs before deployment
* Use exclusion ranges to protect static infrastructure IPs
* Backup DHCP configuration regularly
* Implement DHCP Failover (Hot Standby) for High Availability
* Verify using console checks and real client testing
