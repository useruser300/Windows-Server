## Account Operations + Password/Lockout Policies

This section covers the most common account operations that system administrators perform daily, including:
- User onboarding workflow (create → group membership → restrictions)
- Account restrictions (Log On To / Logon Hours)
- Password reset operations
- Unlock vs Disable/Enable logic
- Domain-level Password Policy and Account Lockout Policy (security baseline)

All examples are based on the domain:
- `iq.networks`

---

### Add and manage new user accounts (Operations workflow)

A common workflow in real companies is:

1. Create the user account
2. Assign group membership
3. Configure restrictions (if required)
4. Confirm logon behavior
5. Provide user with a temporary password (force change at first login)

This ensures:
- Accounts are ready before users start working
- Permissions are correct
- Security controls are applied properly

---

### Restrict a user to log on only from specific computers (Log On To)

Sometimes you want to control where a user can log in from.

Examples:
- Restrict an administrator account to only one machine
- Restrict an employee account to their assigned work PC

Steps:

1. Open **Active Directory Users and Computers (ADUC)**
2. Go to the **Users** container or the OU where the user exists
3. Select the user account  
   - By default, the user can log on from any computer in the domain
4. Right-click the user → **Properties**
5. Go to the **Account** tab
6. Click **Log On To**

Default behavior:
- The user can log in from any computer

Why administrators often need access from any computer:  
Administrator accounts may require access to multiple machines, such as:
- Formatting a computer
- Fixing user devices
- Applying settings and configurations

Why regular users might be restricted:  
Some companies limit employees to log in only from their assigned computer.  
Other environments allow them to use multiple computers, so the restriction may be left open.

To restrict access:

1. Select the option to allow logon only from specific computers
2. Click **Add**
3. Add the allowed computer names (example):
   - `WIN10-CL1`
4. Click **OK** to apply

Result:  
If the user tries to log in from another machine such as `SRV-DC2-CORE` or another client, access will be denied because the account is only allowed to log in from the computers listed.

Final result:  
You have successfully limited this user account to log on only from a specific computer or set of computers.

---

### Control user logon hours (Logon Hours)

This setting controls when the user is allowed to sign in.

Steps:

1. In ADUC, right-click the user account → **Properties**
2. Go to the **Account** tab
3. Click **Logon Hours**

Default schedule:
- All time blocks are usually blue
- This means the user can log in any time, Sunday to Saturday, 24 hours a day

Set allowed working hours:  
If you want to limit the user to working hours, allow logon only during specific times.

Example:
- Allow logon from **06:00 AM until 05:00 PM**

Deny logon outside working hours:

1. Select the blocks you want to restrict
2. Set them to **Logon Denied**
3. Click **OK**

Examples:
- Private companies: work hours early morning until late afternoon
- Government environments: work hours might end earlier like 2 PM, 3 PM, or 4 PM

Benefit of logon hour restrictions:  
If the environment has internet access or VPN access from outside the company, this helps prevent users from logging in outside allowed working hours.

Allow 24-hour access if needed:  
If the user must work 24 hours a day, keep all blocks allowed.

Weekend restrictions if required:  
You can deny logon on specific days such as Friday and Saturday if they are weekends in your country.

Example:
- In Iraq, Friday and Saturday may be official holidays, so you can block logon on those days.

Some roles may need weekend access:  
Some engineers and special roles may need weekend access, so you keep those days allowed.

Final result:  
You have successfully applied logon hour restrictions based on company rules.

---

### Reset a user password in Active Directory

This is used when a user forgets their password.

Steps:

1. In ADUC, find the user account
2. Right-click the user
3. Choose **Reset Password**
4. Enter the new password

Password strength requirement:  
If the password is not strong enough, the system rejects it.  
This depends on the domain password policy.

If the password is rejected:
- Create a stronger password
- Enter it again
- Confirm it

Important:  
Keep this option enabled:
- **User must change password at next logon**

Why this matters:  
The user should choose their own password after logging in.

Result:  
After resetting, you will see a message that the password was changed successfully.

---

### Unlock vs Disable/Enable account

There is a major difference between unlocking and enabling an account:

- **Unlock account**
  - The account is locked because of failed login attempts
- **Disable account**
  - The account was manually disabled by an administrator

---

#### Unlock an account

Steps:

1. In ADUC, right-click the user → **Properties**
2. Go to the **Account** tab
3. Find **Unlock account**
4. Check **Unlock account**
5. Click **Apply**
6. Click **OK**

Now the user can log in again.

Why accounts get locked:
- User enters the wrong password many times
- A policy triggers lockout after failed attempts

---

#### Disable an account

Steps:

1. Right-click the user
2. Click **Disable account**

After disabling:
- The account shows a small down arrow icon
- The user cannot log in

Why disabling accounts is useful:  
Sometimes you create accounts early but want to configure:
- Group membership
- Permissions
- Security options  
before giving access to employees.

Example:  
HR department needs four new employee accounts. You:
- Create accounts first
- Keep them disabled
- Configure their permissions
- Enable accounts when everything is ready

---

#### Enable an account again

Steps:

1. Right-click the user
2. Choose **Enable account**

Final result:  
You can unlock accounts after failed logon attempts, and you can disable/enable accounts as part of an account provisioning workflow.

---

## Password Policy + Account Lockout Policy (Domain Security Baseline)

Password and lockout policies are configured using **Group Policy** at the **domain level**.

---

### Open Default Domain Policy

Steps:

1. Open **Group Policy Management**
2. Expand:
   - Forest
   - Domains
   - `iq.networks`
3. Right-click:
   - **Default Domain Policy**
4. Click **Edit**

Navigate to:

- **Computer Configuration**
  → **Policies**
  → **Windows Settings**
  → **Security Settings**
  → **Account Policies**
  → **Password Policy**

---

### Configure Password Policy values

Set the following:

- Maximum password age: **60 days**
- Minimum password length: **8 characters**
- Password complexity requirements: **Enabled**
- Enforce password history: **3 passwords**

These settings ensure that passwords follow a basic corporate security baseline.

---

### Configure Account Lockout Policy values

In the same policy editor, navigate to:

- **Computer Configuration**
  → **Policies**
  → **Windows Settings**
  → **Security Settings**
  → **Account Policies**
  → **Account Lockout Policy**

Set the following:

- Account lockout duration: **60 minutes**
- Account lockout threshold: **4 invalid logon attempts**
- Reset account lockout counter after: **15 minutes**

Final result:  
This enforces account lockout after 4 invalid attempts as part of domain security.
