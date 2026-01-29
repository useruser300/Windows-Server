## File Sharing (SMB + Permissions + Mapping Drives)

This section defines a corporate-style approach to file sharing in an Active Directory environment.  
It covers:

- Creating SMB shares using multiple methods (File Explorer, Server Manager, PowerShell)
- Applying permissions correctly (**Share vs NTFS**)
- Designing department-based access (Dev + HR example)
- Mapping network drives for users (**recommended: Group Policy Preferences**)

All naming and access design should align with the department model (OUs/Groups) and the lab variables:

- Domain: `iq.networks`
- NetBIOS: `iq`
- File Server (in this lab): `SRV-DC1-GUI` (`192.168.10.10`)
- Example client: `WIN10-CL1`

---

### Share Permissions vs NTFS Permissions (Critical Rule)

When a user accesses a shared folder, Windows evaluates:

1) **Share permissions** (network-level access over SMB)  
2) **NTFS permissions** (file system permissions on the folder itself)

Important rule:  
Windows uses the **most restrictive** permission result.

Example:

- NTFS = Modify  
- Share = Read  

Result:
- **Read only**

This means you must design both permission layers carefully to avoid mistakes.

---

### Method A — Create a Shared Folder using File Explorer (GUI)

This method is fast and commonly used for small shares or quick tests.

---

#### A1) Create the folder

1. Open **File Explorer**
2. Go to `C:\` (or any drive)
3. Create a new folder  
   Example: `HR-Share`

---

#### A2) Share the folder

1. Right-click the folder `HR-Share`
2. Click **Properties**
3. Go to the **Sharing** tab
4. Click **Share...**
5. Choose users  
   - `Everyone` should only be used for testing
6. Set permission level:
   - Read
   - Read/Write
7. Click **Share**

---

#### A3) Check the shared folder locally

1. In File Explorer, type:
   - `\\localhost`
2. Press Enter
3. Confirm the shared folder appears

You can also test via Run:

- Press `Windows + R`
- Type:
  - `\\localhost`

---

#### A4) Access the share from another computer

On a client (example: `WIN10-CL1`):

1. Press `Windows + R`
2. Type:
   - `\\SRV-DC1-GUI`
   or
   - `\\192.168.10.10`
3. Press Enter
4. Open `HR-Share`

---

#### A5) Change NTFS security permissions (optional but common)

1. Right-click the folder → **Properties**
2. Go to **Security**
3. Configure permissions such as:
   - Full Control
   - Modify
   - Read

This controls what the user can do inside the folder on the file system level.

---

### Method B — Create a Shared Folder using Server Manager (Enterprise-style GUI)

This is a more structured corporate method and works well for controlled environments.

This method is more enterprise-ready because it clearly separates:
- Share permissions
- NTFS permissions  
and allows additional options such as:
- Access-based Enumeration (ABE)

---

#### B1) Open the Shares console

1. Open **Server Manager**
2. Go to:
   - **File and Storage Services**
   - **Shares**

---

#### B2) Create a new share

1. Click:
   - **TASKS**
   - **New Share**

---

#### B3) Choose share type

- Select:
  - **SMB Share – Quick**

This works with:
- Windows
- Linux
- macOS

Click **Next**

---

#### B4) Choose folder path

1. Select:
   - **Custom Path**  
   (recommended to avoid sharing an entire drive by mistake)
2. Click **Browse**
3. Choose your drive (example: `C:\`)
4. Create a folder (example: `Data-Share`)
5. Click **Select Folder**
6. Click **Next**

---

#### B5) Share name

Share name:
- `Data-Share`

Remote path will look like:
- `\\SRV-DC1-GUI\Data-Share`

Click **Next**

---

#### B6) Share settings (optional)

You can enable:
- **Access-based Enumeration**

Meaning:  
Users only see folders they have permission to access.

Click **Next**

---

#### B7) Permissions (Important)

You will set **two types of permissions**:

A) **NTFS Permissions**
- These are permissions inside Windows for the folder itself

B) **Share Permissions**
- These control network access over SMB

Important rule:
Windows enforces the **most restrictive** permission across both layers.

Example:
- NTFS = Modify
- Share = Read
Result:
- Read only

---

#### B8) Remove Everyone (recommended in companies)

If you see `Everyone`, remove it.  
Then add principals such as:

- Department groups (recommended)
- Domain Users (only if needed)

Assign appropriate permissions such as:
- Read
- Change
- Full Control

---

#### B9) Create the share

Click **Create**

---

#### B10) Test the share

On the server:

1. Open File Explorer
2. Type:
   - `\\localhost`
3. You should see the share:
   - `Data-Share`

Quick check:
- Read only → user cannot create folders/files
- Modify / Full Control → user can create and edit files

---

### Method C — Create a Shared Folder using PowerShell (Automation-ready)

This method is used in automation, scripts, and standardized deployments.

---

#### C1) Create a shared folder for Dev and HR (department-based access)

On `SRV-DC1-GUI`, run PowerShell:

```powershell
New-Item -Path "C:\Shared\DevHR" -ItemType Directory

New-SmbShare -Name "DevHR" -Path "C:\Shared\DevHR" -FullAccess "iq\Dev Group","iq\HR Group"
```

Notes:

- `-FullAccess` gives full Share permissions.
- You can also use:
  - `-ChangeAccess`
  - `-ReadAccess`

Explanation (corporate model):

- Folder location: `C:\Shared\DevHR`
- SMB share name: `DevHR`
- Access granted to department groups:
  - `iq\Dev Group`
  - `iq\HR Group`

---

### NTFS permissions design note (Create/Edit but not Delete requirement)

Sometimes a company requires a rule such as:

- Users can create and edit files
- Users must NOT delete files

This is a special access design requirement.  
It requires careful NTFS permission planning (advanced permissions are usually required).

Example NTFS ACL setup (basic example, not a full "no delete" enforcement):

```powershell
$Acl = Get-Acl "C:\Shared\DevHR"

$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
  "iq\Dev Group",
  "Modify",
  "ContainerInherit, ObjectInherit",
  "None",
  "Allow"
)

$Acl.AddAccessRule($AccessRule)
Set-Acl "C:\Shared\DevHR" $Acl
```

Important clarification:

- The “create/edit but not delete” requirement needs advanced NTFS permissions.
- The core principle remains:
  - Align NTFS permissions with the business requirement
  - Ensure Share permissions do not override the plan

---

### Validate and access the share (Server + Client)

#### Local validation on the server

1. Open File Explorer
2. Type:
   - `\\localhost`
3. Confirm `DevHR` exists

---

#### Client access validation (WIN10-CL1)

1. Press `Windows + R`
2. Type:
   - `\\SRV-DC1-GUI\DevHR`
     or
   - `\\192.168.10.10\DevHR`
3. Press Enter
4. Confirm access is correct based on user group membership (Dev/HR)

---

### Mapping Drives (Corporate standard: Group Policy Preferences)

Mapping a network drive is a typical corporate requirement.  
Users should not manually browse to the share path every time.  
Instead, a drive letter is mapped automatically at logon.

---

#### Recommended approach — Group Policy Preferences (GPP)

Recommended corporate options:

- Use Group Policy Preferences to map drives based on:
  - OU membership
  - Group membership
  - Security filtering
  - Item-level targeting

This ensures:
- Dev users automatically get the DevHR drive
- HR users automatically get the DevHR drive (if required)
- Other departments do not see it

High-level steps:

1. Open **Group Policy Management**
2. Create or edit a GPO linked to the OU that contains the target users
3. Go to:
   - **User Configuration**
     → **Preferences**
     → **Windows Settings**
     → **Drive Maps**
4. Create a new drive map:
   - Action: Create/Update
   - Location: `\\SRV-DC1-GUI\DevHR`
   - Drive letter: `H:` (example)
5. Security filtering:
   - Apply to:
     - `iq\Dev Group`
     - `iq\HR Group`

---

#### Alternative mapping method — PowerShell (PSDrive)

You can map a network drive using PowerShell (useful for testing):

```powershell
New-PSDrive -Name "DevHR" -PSProvider FileSystem -Root "\\SRV-DC1-GUI\DevHR" -Persist
```

Notes:

- In corporate environments, mapping is typically centralized via GPO rather than manual commands.
- This method is still useful for testing or scripted setups.

---

### Operational best practice summary

- Use department groups for permissions (Dev/HR) instead of assigning rights to individual users
- Always plan both layers:
  - Share permissions (network access)
  - NTFS permissions (file system control)
- Remember the enforcement rule:
  - Most restrictive permission wins
- Prefer central mapping via Group Policy Preferences
- Validate from both:
  - Server (`\\localhost`)
  - Client (`\\SRV-DC1-GUI\ShareName`)
