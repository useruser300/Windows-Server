# Backup (Full Server) — Windows Server Backup + Daily Schedule (23:00)

This section configures a **Full Server Backup** schedule using:
- **Windows Server Backup**
- **wbadmin**
- **schtasks**

Backups are critical in real corporate environments because they protect against:
- server failure
- disk corruption
- accidental deletion
- misconfiguration
- ransomware incidents

Important concept:  
**Backup is not the same as High Availability**
- **High Availability** keeps services running when something fails
- **Backup** allows recovery when data is lost or damaged

This lab implements a basic daily backup schedule at:
- **23:00 (11:00 PM)**

Environment Reference:
- Primary server (backup source): `SRV-DC1-GUI` (`192.168.10.10`)
- Domain: `iq.networks`

---

## Backup Design

In this lab environment, the goal is to protect at least:
- System State / OS critical components
- Active Directory infrastructure (because DC1 is a Domain Controller)
- Critical volumes required for recovery

Realistic lab design:
- Backup runs daily at **23:00**
- Backup is stored on a **separate disk/target** (recommended: `D:`)

---

## Prerequisites

Before enabling scheduled backups:

Ensure you have a valid backup target:
- A second disk volume such as `D:`
- OR an external disk
- OR a network share (advanced option)

Ensure enough free disk space exists on the backup target.

Decide which server is backed up.  
In corporate labs you typically back up:
- `SRV-DC1-GUI` (Primary infrastructure server)

Optionally also:
- `SRV-DC2-CORE` (Secondary server)

This section focuses on a standard setup on:
- `SRV-DC1-GUI`

---

## Install Windows Server Backup Feature

On `SRV-DC1-GUI`, run PowerShell as Administrator:

```powershell
Install-WindowsFeature -Name Windows-Server-Backup -IncludeManagementTools
```

Result:
- Windows Server Backup tools are installed
- `wbadmin` becomes available

---

## Validate Backup Tools Installation

Check the feature status:

```powershell
Get-WindowsFeature Windows-Server-Backup
```

Expected:
- `Installed = True`

Confirm `wbadmin` exists:

```powershell
wbadmin /?
```

---

## Create a Daily Full Server Backup Schedule (23:00)

We will use `schtasks` to schedule a backup job that runs every day at **23:00**.

### Base Backup Command (wbadmin)

Corporate-style backup command example:
- Backup target: `D:`
- Include system critical volumes
- Quiet mode enabled

```powershell
wbadmin start backup -backupTarget:D: -include:C: -allCritical -quiet
```

Explanation:
- `-backupTarget:D:` → where backups are saved
- `-include:C:` → includes the C: drive data
- `-allCritical` → includes required critical volumes for system recovery
- `-quiet` → runs without interactive prompts

Technical notes:
- `-allCritical` is required for full system recovery readiness
- Backup target must not be the same volume being protected

---

### Create the Scheduled Task (Daily at 23:00)

Run on `SRV-DC1-GUI` (as Administrator):

```cmd
schtasks /create /tn "Full Server Backup - 23:00" /tr "wbadmin start backup -backupTarget:D: -include:C: -allCritical -quiet" /sc daily /st 23:00 /ru "SYSTEM"
```

Result:
- A scheduled backup task is created
- It runs daily at **23:00** under `SYSTEM`

Why run as SYSTEM:
- SYSTEM has required backup privileges
- No password required
- Works reliably in server environments

---

## Verify the Scheduled Task

List the task:

```cmd
schtasks /query /tn "Full Server Backup - 23:00"
```

Expected:
- TaskName: `Full Server Backup - 23:00`
- Schedule: `Daily`
- Time: `23:00`

---

## Run Backup Immediately (Test Validation)

Run the task manually: To test without waiting until 23:00.

```cmd
schtasks /run /tn "Full Server Backup - 23:00"
```

Monitor backup status:

```powershell
wbadmin get status
```
Result:
- Backup is running and completes successfully


Optional: list existing backups after completion:

```powershell
wbadmin get versions
```

---

## Restore Concept (Recovery Logic)

A backup is only valuable if it can be restored.

Common restore scenarios:
- File-level restore
- System State restore
- Full server recovery after failure

Windows Server Backup supports:
- file/folder recovery
- full server recovery

Important domain note:
- Domain Controllers require careful restore planning: authoritative or non-authoritative

---

## Backup Best Practices (Enterprise Mindset)

- Always store backups on a separate disk or target
- Never store backups on the same disk being protected
- Validate backups regularly (test restores)
- Document:
  - backup scope
  - schedule
  - retention behavior
- Backup is not a replacement for replication or failover

---

## Verification Checklist

Windows Server Backup installed:
```powershell
Get-WindowsFeature Windows-Server-Backup
```

Scheduled task exists:
```cmd
schtasks /query /tn "Full Server Backup - 23:00"
```

Backup can run successfully:
```powershell
wbadmin get status
```

Backup versions exist:
```powershell
wbadmin get versions
```

Backup target contains data:
- Check `D:` after backup completion
