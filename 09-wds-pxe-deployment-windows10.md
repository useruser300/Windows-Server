## Deployment (WDS) — Windows Deployment Services (PXE)

This section adds **Windows Deployment Services (WDS)** to the lab so you can deploy Windows 10 clients using **PXE boot** instead of installing manually.

WDS is commonly used in corporate environments to:
- Deploy Windows to many clients quickly
- Standardize OS installation and configuration
- Reduce manual installation time for IT teams

In this Unified Lab Profile, WDS will be installed on:
- **WDS Server:** `SRV-DC1-GUI` (`192.168.10.10`)

---

## Prerequisites (Required before WDS works)

Before deploying clients using PXE, ensure:

### Active Directory + DNS are ready
- Domain: `iq.networks`
- DNS Primary: `192.168.10.10`

### DHCP is working in the lab
- Scope: `192.168.10.150 - 192.168.10.200`
- Gateway: `192.168.10.1`

### Windows 10 ISO is available
You need a Windows 10 ISO file to add:
- Boot image (WinPE): `sources\boot.wim`
- Install image: `sources\install.wim` (or `sources\install.esd`)

### PXE client VM or physical client
- Example client: `WIN10-CL1`
- Must support Network Boot / PXE

---

## Install WDS Role on SRV-DC1-GUI

### Method A — PowerShell (Recommended)

Run PowerShell as Administrator on `SRV-DC1-GUI`:

```powershell
Install-WindowsFeature -Name WDS -IncludeManagementTools
```

### Method B — Server Manager (GUI Alternative)

1. Open **Server Manager**
2. Go to **Manage** → **Add Roles and Features**
3. Select:
   * **Windows Deployment Services**
4. Next → Install

Result:
WDS is installed and ready for initialization.

---

## Initialize the WDS Server

WDS must be initialized before it can serve PXE clients.

### Method A — WDSUTIL (CMD / PowerShell)

Run as Administrator on `SRV-DC1-GUI`:

```cmd
WDSUTIL /Initialize-Server /Server:SRV-DC1-GUI /RemInst:"C:\RemoteInstall"
```

Explanation:
- `C:\RemoteInstall` is the default storage folder used by WDS for images and PXE boot files.

Result:
- The WDS server is initialized
- The **RemoteInstall** folder is created

Technical note (best practice):
- Use a data disk for `RemoteInstall` if available (large images), but for the lab `C:\RemoteInstall` is fine.

---

## Add Windows 10 Boot Image (WinPE)

WDS requires a **Boot Image** so clients can boot into Windows Setup over the network.

Boot image path inside the Windows ISO:
- `sources\boot.wim`

### Add Boot Image using WDS Console (Recommended)

1. Open **Server Manager**
2. Go to **Tools** → **Windows Deployment Services**
3. Expand:
   * **Servers → SRV-DC1-GUI**
4. Right-click:
   * **Boot Images → Add Boot Image**
5. Browse to the Windows 10 ISO:
   * `sources\boot.wim`
6. Name it:
   * `Windows 10 Boot (PXE)`
7. Finish

Result:
- PXE clients can boot into Windows Setup.

---

## Add Windows 10 Install Image

The **Install Image** is what actually installs Windows 10 on the client.

Inside the Windows ISO:
- `sources\install.wim`
  or
- `sources\install.esd`

Important:
- WDS works best with **install.wim**
- If your ISO only has **install.esd**, you may need to convert ESD → WIM first

### Step 1 — Create an Image Group

1. In WDS Console, right-click:
   * **Install Images**
2. Click:
   * **Add Image Group**
3. Name it:
   * `Windows 10 Images`

### Step 2 — Add the Install Image

1. Right-click:
   * `Windows 10 Images`
2. Click:
   * **Add Install Image**
3. Browse to:
   * `sources\install.wim`
4. Finish

Result:
- WDS is now ready to deploy Windows 10.

---

## Configure PXE Response Settings

You must define how WDS responds to PXE requests.

1. In WDS Console, right-click:
   * `SRV-DC1-GUI`
2. Click:
   * **Properties**
3. Open the **PXE Response** tab
4. Recommended lab mode:
   * **Respond to all client computers (known and unknown)**

Result:
- Any PXE client in the lab can boot and install.

Note:
- In real companies, it is common to select **known clients only** for security.

---

## Deploy Windows 10 via PXE Boot

Steps on the client VM / machine (`WIN10-CL1`):

1. Start `WIN10-CL1`
2. Enter BIOS/UEFI Boot Menu
3. Select:
   * **Network Boot (PXE)**
4. The client should receive a DHCP IP address from:
   * `SRV-DC1-GUI`
5. WDS provides the boot image:
   * `Windows 10 Boot (PXE)`
6. Windows Setup starts
7. Select edition and disk settings
8. Install Windows

Result:
Windows 10 is installed using PXE deployment from the lab infrastructure.

---

## Post-Deployment: Join Windows 10 to the Domain

After installation completes, join the client to the domain.

### Method A — GUI Domain Join

Use the same approach from the client-join section:
* Domain: iq.networks
* DNS: 192.168.10.10

### Method B — PowerShell Domain Join

Run PowerShell as Administrator on WIN10-CL1:
```powershell
Add-Computer -DomainName "iq.networks" -Credential (Get-Credential)
Restart-Computer
```

---

## Verification (Confirm WDS deployment works)

### Verify WDS role is installed on SRV-DC1-GUI
```powershell
Get-WindowsFeature WDS
```

### Verify WDS service exists and is running
```powershell
Get-Service WDSServer
```

### Verify DHCP delivered an IP to the PXE client

On the PXE client after boot, confirm it received an IP in:
* 192.168.10.150 - 192.168.10.200

### Verify domain join after deployment

On SRV-DC1-GUI:
* Tools → Active Directory Users and Computers (ADUC)
* Domain: iq.networks
* Confirm WIN10-CL1 exists under Computers (or the OU you moved it to)

---

## Operational Notes (Lab Reality vs Corporate Networks)

In real production networks, PXE deployment often requires additional network configuration such as:
* IP Helper Address on routers (to forward PXE requests across VLANs/subnets)
* DHCP options (if DHCP and WDS are not on the same server)
* Secure deployment restrictions (known clients only)

In this lab:
* WDS + DHCP + DNS are aligned under SRV-DC1-GUI
* Same subnet 192.168.10.0/24

So PXE deployment should work smoothly inside the same network.

---

## Troubleshooting Quick Notes (Most common lab issues)

### PXE client gets IP but does not see a boot image
* Ensure Boot Image is added under Boot Images
* Ensure WDS is responding to PXE requests (PXE Response tab)
* Restart WDS service:
```powershell
Restart-Service WDSServer
```

### PXE client does not get an IP address
* Check DHCP scope is active
* Verify client is connected to the correct network/vSwitch
* Confirm DHCP exclusions and scope range are correct

### WDS fails to initialize / RemoteInstall issues
* Confirm disk space
* Confirm the path exists and permissions are OK
* Try initializing again with WDSUTIL

