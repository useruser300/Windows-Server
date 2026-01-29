# Domain Setup (DC1 GUI + DC2 Core + Join WIN10)

This section builds the Active Directory domain foundation using:
- **Primary Domain Controller (GUI):** `SRV-DC1-GUI` (`192.168.10.10`)
- **Secondary Domain Controller (Core):** `SRV-DC2-CORE` (`192.168.10.12`)
- **Windows 10 Client:** `WIN10-CL1`

All values must match the **Lab Environment Variables**.

---

## 1) Primary Domain Controller Setup (Windows Server 2019 GUI) — SRV-DC1-GUI

This section prepares the main Windows Server (GUI) as the **Primary Domain Controller (PDC)** and **Primary DNS server**.

---

### 1.1 Rename the Server (Recommended)

Rename the server to match the lab naming convention.

```powershell
Rename-Computer -NewName "SRV-DC1-GUI"
Restart-Computer
```

---

### 1.2 Verify Current Network Configuration

PowerShell:
```powershell
Get-NetIPAddress
```

```cmd
ipconfig /all
```

---

### 1.3 Configure a Static IP Address  
**SRV-DC1-GUI = 192.168.10.10**

A static IP is required for:
- Domain Controller stability
- DNS stability
- Domain join consistency for clients

```powershell
New-NetIPAddress -InterfaceIndex <YOUR_INTERFACE_INDEX> -IPAddress 192.168.10.10 -PrefixLength 24
```

Verify:
```cmd
ipconfig /all
```

Notes:
- Replace `<YOUR_INTERFACE_INDEX>` with the correct interface index
- Prefix length `24` = `255.255.255.0` (`192.168.10.0/24`)

---

## 2) Install Active Directory Domain Services (AD DS) on SRV-DC1-GUI

Two valid installation methods:
- GUI (Server Manager)
- PowerShell (recommended)

---

### 2.1 Install AD DS using Server Manager (GUI Method)

1. Open **Server Manager**
2. Go to **Manage**
3. Click **Add Roles and Features**
4. Click **Next**
5. Choose **Role-based or feature-based installation**
6. Select the local server: `SRV-DC1-GUI`
7. Select **Active Directory Domain Services**
8. Click **Next → Next**
9. Click **Install**

---

### 2.2 Install AD DS using PowerShell (Recommended)

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

---

## 3) Promote SRV-DC1-GUI to Domain Controller (Create New Forest)

---

### 3.1 Promote via GUI (Server Manager)

1. Open **Server Manager**
2. Click the notification flag (top-right)
3. Select **Promote this server to a domain controller**
4. Choose **Add a new forest**
5. Domain name:
   - `iq.networks`
6. Configure:
   - NetBIOS name: `iq`
   - Set a strong DSRM password
7. Click **Next** through the wizard
8. Click **Install**
9. Server restarts automatically

Result:  
After reboot, the server becomes the Domain Controller for `iq.networks`.

---

### 3.2 Promote via PowerShell (Alternative)

```powershell
Install-ADDSForest
```

Provide during setup:
- Domain name: `iq.networks`
- A secure DSRM password

---

## 4) Secondary Domain Controller Setup (Windows Server 2019 Core) — SRV-DC2-CORE

---

### 4.1 Open SCONFIG

```text
sconfig
```

---

### 4.2 Configure Network Settings (Static IP + DNS)

Inside **sconfig**:

1. Server Configuration  
2. `8) Network Settings`  
3. `1) Network Adapter Settings`

**Option 1: Set Network Adapter Address (Static)**  
- Choose `1)`
- Select `(S)` Static
- IP Address: `192.168.10.12`
- Subnet: `255.255.255.0`
- Gateway: `192.168.10.1`

**Option 2: Set DNS Servers**  
- Choose `2)`
- Primary DNS: `192.168.10.10`

---

## 5) Join SRV-DC2-CORE to the Domain (`iq.networks`)

1. Open `sconfig`
2. Select `1) Domain/Workgroup`
3. Choose **Domain**
4. Domain name: `iq.networks`
5. Enter credentials:
   - Username: `Administrator`
   - Password: domain admin password
6. Restart if prompted

Result:  
`SRV-DC2-CORE` is now a domain member.

---

## 6) Add SRV-DC2-CORE to Server Manager (SRV-DC1-GUI)

On `SRV-DC1-GUI`:

1. Open **Server Manager**
2. Go to **All Servers**
3. Right-click → **Add Servers**
4. Search and select `SRV-DC2-CORE`
5. Add → **OK**

---

## 7) Promote SRV-DC2-CORE to an Additional Domain Controller

---

### 7.1 Install AD DS on SRV-DC2-CORE

From **Server Manager** on `SRV-DC1-GUI`:
1. **Add Roles and Features**
2. Target server: `SRV-DC2-CORE`
3. Select **Active Directory Domain Services**
4. Install

---

### 7.2 Promote SRV-DC2-CORE as Additional Domain Controller

1. Promote server
2. Select **Add a domain controller to an existing domain**
3. Enter domain admin credentials
4. Domain: `iq.networks`
5. Domain Controller Options:
   - Uncheck **DNS**
   - Uncheck **Global Catalog**
6. Next → Install

Result:  
`SRV-DC2-CORE` is now an additional Domain Controller in `iq.networks`.

---

## 8) Join Windows 10 Client to Active Directory — WIN10-CL1

---

### 8.1 Configure Windows 10 Client

1. Rename computer to `WIN10-CL1`
2. Configure IP as needed
3. Set DNS server:
   - `192.168.10.10`

---

### 8.2 Join Windows 10 to the Domain

1. Open **Control Panel**
2. Go to **System**
3. Click **Change settings**
4. Click **Change…**
5. Select **Domain**
6. Enter `iq.networks`
7. Provide credentials:
   - Username: `Administrator`
   - Password: domain admin password
8. Restart when prompted

Result:  
Windows 10 client is joined to the domain.

---

### 8.3 Join the domain using PowerShell (Alternative method)

This method is useful for automation and faster deployments.

Run PowerShell as Administrator on Windows 10:

```powershell
Add-Computer -DomainName "iq.networks" -Credential (Get-Credential)
```
Enter domain admin credentials when prompted, then restart:

```text
Restart-Computer
```
---

## 9) Verify Windows 10 Computer in Active Directory

On `SRV-DC1-GUI`:

1. Open **Server Manager**
2. Go to **Tools**
3. Open **Active Directory Users and Computers**
4. Expand `iq.networks`
5. Open **Computers**
6. Confirm:
   - `WIN10-CL1` is listed
