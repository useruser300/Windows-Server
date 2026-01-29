## Group Policy Catalog (Baseline + OU Targeted Security Controls)

Group Policy is used to enforce corporate security rules across the domain.  
This catalog is organized into:

- Domain-wide baseline policies
- OU targeted user policies
- OU targeted computer policies
- Administrative validation commands

---

### Domain-wide baseline: Allow Remote Desktop on all machines

This ensures IT administrators can manage machines remotely inside the domain.

Recommended approach:  
Create a dedicated GPO for Remote Desktop and link it to the correct Computer scope.

---

#### Step 1: Create a new GPO (Baseline - Remote Desktop)

1. Open **Group Policy Management**
2. Right-click the domain:
   - `iq.networks`
3. Click:
   - **Create a GPO in this domain, and Link it here...**
4. Name it:
   - `Baseline - Remote Desktop`

---

#### Step 2: Configure Remote Desktop setting

1. Right-click the new GPO:
   - `Baseline - Remote Desktop`
2. Click **Edit**
3. Navigate to:

- **Computer Configuration**
  → **Policies**
  → **Administrative Templates**
  → **Windows Components**
  → **Remote Desktop Services**
  → **Remote Desktop Session Host**
  → **Connections**

4. Enable:
- **Allow users to connect remotely using Remote Desktop Services**

---

#### Step 3: Link / Scope recommendation

Recommended:  
Link this GPO to the OU that contains your computers (workstations/servers).

Firewall note:  
Make sure Windows Firewall rule: Remote Desktop (TCP-In) is allowed

---

### User Security Policies (Access restrictions)

This section includes common user-based restrictions used in corporate environments.

---

#### Block external/removable storage devices

Steps:

1. Open **Group Policy Management**
2. Edit the correct policy (baseline or OU policy)
3. Navigate to:

- **User Configuration**
  → **Policies**
  → **Administrative Templates**
  → **System**
  → **Removable Storage Access**

4. Enable:
- **All Removable Storage classes: Deny all access**

Result:  
External storage devices are blocked for users under that policy scope.

---

#### Disable Task Manager

Steps:

- **User Configuration**
  → **Policies**
  → **Administrative Templates**
  → **System**
  → **Ctrl+Alt+Del Options**

Enable:
- **Remove Task Manager**

Result:  
Users cannot open Task Manager.

---

#### Remove Properties from right-click context menu

Steps:

- **User Configuration**
  → **Policies**
  → **Administrative Templates**
  → **Windows Components**
  → **File Explorer**

Enable:
- **Remove Properties from the context menu**

Result:  
Users cannot open file/folder Properties from the right-click menu.

---

### OU Targeted Policy: Restrict Add/Remove Programs (Example OU: Sales)

This is used to prevent users from installing or removing software.

Steps:

1. Open **Active Directory Users and Computers (ADUC)**
2. Right-click the domain `iq.networks`
3. Create an OU named:
   - `Sales`
4. Create a user and move the user into the `Sales` OU

Now apply a GPO:

1. Open **Group Policy Management**
2. Expand:
   - Forest → Domains → `iq.networks`
3. Find the OU:
   - `Sales`
4. Right-click the OU → **Create a GPO in this domain, and Link it here...**
5. Give the GPO a name
6. Right-click the new GPO → **Edit**
7. Navigate to:

- **User Configuration**
  → **Administrative Templates**
  → **Control Panel**
  → **Add or Remove Programs**

8. Enable the required restrictions

Result:  
Users in the `Sales` OU will be restricted from installing/removing programs.

---

### OU Targeted User Policy: Prevent access to CMD

This is used when you want to prevent users from using command-line tools.

Steps:

1. In **Group Policy Management**, find the target OU (example: `Sales`)
2. Edit the linked GPO
3. Navigate to:

- **User Configuration**
  → **Administrative Templates**
  → **System**

4. Find:
- **Prevent access to the command prompt**

5. Enable or disable based on your requirement

Result:  
Users in that OU cannot access CMD.

---

### OU Targeted Computer Policy: Disable camera usage

This is used as a device-level corporate security control.

Steps:

1. In ADUC, inside the OU (example: `Sales`)
2. Right-click → add a computer and give it a name
3. In **Group Policy Management**, edit the OU-linked GPO
4. Navigate to:

- **Computer Configuration**
  → **Administrative Templates**
  → **Windows Components**
  → **Camera**

5. Disable:
- **Allow Use of Camera**

Result:  
Computers in that OU cannot use the camera.

---

### Machine inactivity limit (Auto lock / session control)

This configuration forces session lock after a defined number of idle minutes.

Recommended approach:  
Create a dedicated GPO for session lock and link it to the correct Computer scope.

---

#### Step 1: Create a new GPO (Baseline - Session Lock)

1. Open **Group Policy Management**
2. Right-click the domain:
   - `iq.networks`
3. Click:
   - **Create a GPO in this domain, and Link it here...**
4. Name it:
   - `Baseline - Session Lock`

---

#### Step 2: Configure Machine Inactivity Limit

1. Right-click the new GPO:
   - `Baseline - Session Lock`
2. Click **Edit**
3. Navigate to:

- **Computer Configuration**
  → **Policies**
  → **Windows Settings**
  → **Security Settings**
  → **Local Policies**
  → **Security Options**

4. Configure:
- **Interactive logon: Machine inactivity limit**  
Right-click → **Edit** and set the required time

---

#### Step 3: Apply policy update (test)

Run on a target machine (CMD as Administrator):

```cmd
gpupdate /force
```

---

## Backup and Restore Group Policy Objects (GPOs)

GPO backup is important because:
- If a GPO is deleted accidentally, it can be restored
- You can re-use the same policies in another environment

### Backup all GPOs

1. Open **Group Policy Management**
2. Expand:
   - Forest → Domains → `iq.networks`
3. Go to:
   - **Group Policy Objects**
4. Right-click → **Back Up All**
5. Select a backup location
6. Click **Back Up**

---

### Restore GPOs

1. Right-click **Group Policy Objects**
2. Choose **Manage Backups**
3. Select a backup version
4. Click **Restore**

Result:  
Deleted or corrupted GPOs can be restored.

---

## Force a specific screen saver using Group Policy

Steps:

1. Open **Group Policy Management**
2. Go to the domain:
   - `iq.networks`
3. Create a new GPO in this domain
4. Name it:
   - `Screen Saver GP`
5. Right-click it → **Edit**
6. Navigate to:

- **User Configuration**
  → **Administrative Templates**
  → **Control Panel**
  → **Personalization**

7. Enable:
- **Enable screen saver**
- **Screen saver timeout**
- **Force specific screen saver**

Apply policy on the server:
```cmd
gpupdate /force
```

Verify policy results on Windows 10:

On `WIN10-CL1`, run:
```cmd
gpresult /r
```

Result:  
The client receives the screen saver settings enforced by the domain.

---

## Windows Update active hours (Operational maintenance)

Sometimes you want to control when updates happen.

On Windows Server:
1. Open **Update & Security**
2. Select **Change active hours**
3. Under **Advanced options**, configure update behavior

This helps avoid unexpected reboots during working hours.

---

## Reset Domain Controller DSRM Password (Server Recovery Procedure)

Sometimes the DSRM password is lost or unknown.  
It can be reset without reinstalling the system.

This is useful when:
- The system administrator changed
- The previous admin left the company
- The password was lost

Steps:

1. Open **Command Prompt** as Administrator
2. Start the Active Directory maintenance tool:
```cmd
ntdsutil
```

3. Enter DSRM password reset mode:
```text
set dsrm password
```

4. Reset the password on the server:
```text
reset password on server SRV-DC1-GUI
```

If unsure about the server name, run:
```cmd
hostname
```

5. Enter and confirm the new password

Final result:  
The Domain Controller DSRM password is successfully reset.
