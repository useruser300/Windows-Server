## Corporate Structure (OUs + Groups + Users)

This section builds a corporate-style Active Directory structure using:
- **Organizational Units (OUs)** per department
- **Security Groups** per OU
- **Users** inside each OU
- **User → Group mapping** as the standard permission model

All objects are created under the domain:
- `iq.networks`

Server used for administration:
- `SRV-DC1-GUI`

---

### Why we build an OU-based structure

In corporate environments, you do not keep everything inside the default containers only.  
A proper OU structure allows you to:

- Apply different security rules for each department
- Separate user and computer objects
- Manage groups and permissions cleanly
- Scale Active Directory without confusion

The goal is to create a department-based structure, then map:

- Users → Department OU  
- Users → Department Group  
- Policies → OU level (targeted)

---

### Create Organizational Units (OUs) for departments

On `SRV-DC1-GUI`:

1. Open **Server Manager**
2. Go to **Tools**
3. Open **Active Directory Users and Computers (ADUC)**
4. Expand the domain:
   - `iq.networks`
5. Right-click the domain → **New** → **Organizational Unit**

Create these OUs:

- `HR`
- `Sales`
- `Finance`
- `Dev`
- `IT`

Result:  
You now have clean department OUs that match how companies separate teams.

---

### Create department security groups (one group per OU)

Groups are used to:
- Assign permissions on shared folders
- Assign permissions for applications
- Apply access control rules more efficiently
- Avoid assigning permissions directly to individual users

Steps:

1. In ADUC, right-click the OU `HR`
2. Select **New** → **Group**
3. Name the group:
   - `HR Group`
4. Repeat the same for the remaining OUs:

- `Sales Group`
- `Finance Group`
- `Dev Group`
- `IT Group`

Result:  
Each department has its own group, which is a common corporate access design.

---

### Create users inside each department OU

Steps:

1. In ADUC, right-click the department OU  
   Example: `HR`
2. Select **New** → **User**
3. Create users for the department

Example naming approach:
- `HR User1`
- `HR User2`

Repeat the same for other departments.

When creating the user, fill in:
- First name / Last name
- User logon name (username)

Then:

1. Click **Next**
2. Set a temporary password
3. Enable:
   - **User must change password at next logon**

Why this is best practice:  
This ensures the admin does not know the final password of the user.  
The user will set their own password on first login.

4. Click **Finish**

---

### Add users to their department group (User → Group mapping)

After creating users, add them to the correct group.

Steps:

1. Right-click the user
2. Select **Add to a group**
3. Choose the department group  
   Example: add HR users to `HR Group`

Result:  
Department users are linked to their department group, which makes access management scalable.

---

### Optional: Add a user to Administrators group

Sometimes you may create a user and then add them to administrators.

Steps:

1. Double-click the user account
2. Open **Member Of**
3. Click **Add**
4. Choose a group such as:
   - `Administrators`

This is optional and should only be used when needed, because Administrator-level access should be limited.
