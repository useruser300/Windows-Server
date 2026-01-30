# Printers

This section configures a **shared printer** in the domain and makes it available to:
- **All Domain Users**

Printer sharing is a real corporate requirement because:
- Users should not install printers manually
- IT manages drivers and printer access centrally
- Printing permissions can be controlled using Active Directory groups and GPO

Environment:
- Print Server: `SRV-DC1-GUI` (`192.168.10.10`)
- Domain: `iq.networks`

---

## Printer Deployment Models (Corporate overview)

There are two common enterprise models:

### Model A — Print Server model (Recommended)
- Printer is installed on the server
- Printer is shared to clients
- Clients connect to the shared printer from the server
- IT controls drivers and settings centrally

### Model B — Direct IP printer on each client (Not recommended for domain control)
- Printer installed on each client manually
- Harder to manage and standardize
- Drivers and settings vary per device

This lab follows **Model A**.

---

## Prerequisites

Before adding a shared printer, ensure:

Domain environment is working:
- AD DS + DNS ready

Client computers are domain-joined:
- Example: `WIN10-CL1`

You know the printer connection type:
- Network printer (TCP/IP address) example: `192.168.10.100`
- OR USB printer connected to the server

You have printer driver availability:
- Via Windows Update
- OR vendor driver package

---

## Install Print and Document Services (Print Server Role)

On `SRV-DC1-GUI`, install print server features:

```powershell
Install-WindowsFeature -Name Print-Server -IncludeManagementTools
```

Result:
* Print Management tools become available

Optional verification:
```powershell
Get-WindowsFeature Print-Server
```

---

## Add a Printer on SRV-DC1-GUI

There are two common scenarios:

### Scenario A — Network Printer (TCP/IP) (Most common in companies)

Recommended method: Print Management (more control than Devices and Printers)

1. Open:
    * Server Manager → Tools → Print Management
2. Expand:
    * Print Servers → SRV-DC1-GUI
3. Right-click Printers → Add Printer
4. Choose:
    * Add a TCP/IP or Web Services Printer by IP address or hostname
5. Enter printer IP:
    * Example: 192.168.10.100
6. Install/select the correct driver
7. Finish

Result:
* Printer is installed on the server.

### Scenario B — USB Printer (Connected to the Server)

1. Connect the printer to SRV-DC1-GUI
2. Windows detects it
3. Install driver if needed
4. Confirm it appears in:
    * Devices and Printers / Print Management

---

## Share the Printer for Domain Users

Now share the printer so clients can connect using the server share path.

1. On SRV-DC1-GUI, open:
    * Print Management (or Devices and Printers)
2. Right-click the printer → Printer properties
3. Open the Sharing tab
4. Enable:
    * Share this printer
5. Set share name example:
    * CORP-Printer
6. Apply → OK

Result:
* The printer is available via:
    * \\SRV-DC1-GUI\CORP-Printer

---

## Configure Printer Permissions (Domain Users can Print)

Goal:
* All Domain Users can print
* Only admins can manage printer settings

1. Right-click the printer → Printer properties
2. Go to Security
3. Add:
    * Domain Users
4. Allow:
    * Print
5. Keep admin roles:
    * Administrators = Manage printer, Manage documents

Result:
* Domain users can print, admins control configuration.

---

## Deploy the Printer Automatically to Domain Users (Recommended)

In corporate environments, printers should be deployed automatically.

### Method A — Group Policy Preferences (User-based deployment)

1. Open Group Policy Management on SRV-DC1-GUI
2. Create a new GPO:
    * Name: Deploy Printers
3. Link it to:
    * The OU that contains the target users
    * OR the domain (if you want it for all users)
4. Edit the GPO:
    * User Configuration  
      → Preferences  
      → Control Panel Settings  
      → Printers
5. Create a new Shared Printer:
    * Action: Create (or Update if you expect changes)
    * Path: \\SRV-DC1-GUI\CORP-Printer
6. Apply

Result:
* Users receive the printer automatically at logon / policy refresh.

Optional apply/verify on client:
```powershell
gpupdate /force
gpresult /r
```

---

## Monochrome Printing Policy (Black-and-White Only)

### Corporate reality note

Monochrome enforcement depends on:
* Printer capabilities
* Driver features
* Server-side defaults

Correct lab approach:  
Set default printing preference to Monochrome  
(Enforcement beyond defaults is driver-dependent.)

---

### Set Monochrome as default on the Print Server

1. On SRV-DC1-GUI
2. Right-click printer → Printing preferences
3. Set:
    * Color: Off
    * Monochrome / Black & White: On
4. Apply → OK

Result:
* Users print black-and-white by default.
