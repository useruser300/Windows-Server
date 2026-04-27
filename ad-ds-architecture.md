# Active Directory Domain Services (AD DS): Architecture, Components, and Core Relationships

## Introduction

This document explains the core architecture of **Active Directory Domain Services (AD DS)** and how its main components work together in a Windows Server environment.

The focus is not only on defining individual concepts such as Forests, Domains, Organizational Units, Domain Controllers, Sites, Subnets, Users, Groups, and Service Accounts, but also on understanding how these components connect to form one complete identity and access management system.

AD DS is presented as a distributed identity platform that stores directory objects in a replicated database, organizes them logically through Forests, Domains, and OUs, runs physically through Domain Controllers and Sites, and controls access and configuration through Groups, Group Policy Objects, and managed service accounts.

This document is designed as a structured learning reference for understanding AD DS from both a logical and physical architecture perspective.

---

# 1) The Big Picture: What Is AD DS Really?

**Active Directory Domain Services (AD DS)** is a central system in Windows Server for managing:

* **Identities**: Who is the user? What is the device?
* **Authentication**: Is this user/device really from our environment?
* **Authorization**: What is it allowed to do?
* **Organization**: Where do we place users, devices, and groups?
* **Policies**: What settings and rules are enforced on them?
* **Administration**: How do we manage all of this from a central place?

Think of it like this:

> AD DS = **identity database + organization system + policy engine + distributed replication system**

This is the foundation that enterprise networks running Windows systems rely on.

---

# 2) The Scenario That the Module Solves

A company like **Contoso** has:

* several offices around the world
* local Windows servers
* many users and devices
* a need for centralized management
* a need to move to Windows Server 2025

In this environment, it is not enough for every device to have local accounts. The company needs:

* unified sign-in
* centralized user management
* organized permissions
* unified policies
* server and device management at scale

This is where AD DS comes in.

---

# 3) The Basic Mental Structure: How Is the System Built?

## The General Logical Sequence

```text
Forest
└── Domain
└── OU
└── Objects (Users, Computers, Groups, Service Accounts...)
```

## The Actual Implementation on the Ground

```text
Domain Controllers
Sites
Subnets
Data Store
Global Catalog
RODC
```

So you have two levels that you must not confuse:

## Logical Components

They are the way an AD DS environment is **designed and organized**.

## Physical Components

They are the way this design is **run and implemented** in reality.

This is the most important relationship in the whole module.

---

# 4) AD DS as a Database: Ntds.dit and Partitions

## What Is the Idea?

Although AD DS looks to you like a tree structure of Users, OUs, and Domains, deep down it is a **database**. The main file is:

```text
Ntds.dit
```

It is stored by default in:

```text
C:\Windows\NTDS
```

Along with this file, there are **log files**.

This database uses **Microsoft Jet database technology**.

## But Is It One Database or Several Databases?

In reality, it is **one database**, but it is logically divided into **Partitions** or **Naming Contexts**.

## What Is a Partition?

It is a part of the database that contains a specific type of information.

### The Important Types

#### 1. Schema Partition

Contains the definitions of object types and their properties.

#### 2. Configuration Partition

Contains the settings of the Forest.

#### 3. Domain Partition

Contains the users, devices, groups, and objects specific to the domain.

## Why Is This Important?

Because not everything in AD is treated the same way. Some data belongs to the domain, some belongs to the whole forest, and some belongs to object definitions.

## How Does This Relate to the Other Concepts?

* **Schema** lives here
* **Forest-level settings** live here
* **Users/Computers/Groups** are usually in the Domain Partition
* **Replication** copies these parts between DCs depending on their type

## Architectural Thinking Questions

### What problem does it solve?

Organizing AD data logically inside the database instead of it being chaotic or unmanageable.

### What would break if it did not exist?

You would not be able to separate domain data from forest data from object definitions, and replication and administration would become complicated.

### What are the alternatives?

From the AD perspective itself, there is no real alternative; this is part of its internal architecture.

### When do I not use it?

It is not something you choose or do not choose. It is a core part of AD DS design.

### What are the trade-offs?

It forces you to have a deeper understanding of the structure, but it makes the system scalable and organized.

---

# 5) Schema

## What Is It?

The **Schema** is the dictionary or rulebook that defines:

* What object types exist?
* What properties can each type have?

Example:

* User object
* Computer object
* Group object

And each one has attributes such as:

* name
* email
* description
* SID
* memberships

## Why Is It Important?

Because AD DS cannot store any object correctly unless it is **defined in the Schema**.

## How Does It Relate to the Other Concepts?

* Every User, Computer, and Group depends on the Schema
* The Schema is shared at the **Forest** level
* Therefore, the Forest is the **schema boundary**

## Architectural Thinking Questions

### What problem does it solve?

Standardizing the shape of data across the entire system.

### What would break if it did not exist?

There would be no fixed definition for objects or their properties.

### What are the alternatives?

There is no alternative inside AD DS; every Directory Service needs some form of schema.

### When do I not use it?

You do not “use it” as an optional feature; it is always present. But you usually **do not modify it much**.

### What are the trade-offs?

A unified schema is excellent for stability, but applications that need a different schema may force you to create an **additional Forest**.

---

# 6) Logical Components

Now we enter the logical components themselves.

## 6.1 Domain

## What Is It?

The **Domain** is a logical administrative container for managing:

* users
* devices
* groups
* other objects

It is also connected to its own **Domain Partition**.

## How Should You Think About It?

The Domain is the **basic daily management unit** in AD.

It is not just a DNS name. It is also:

* an authentication scope
* a replication scope for domain data
* a management scope

## What Does It Provide?

### Authentication

When a domain-joined device starts or a user signs in, AD DS verifies the identity.

### Authorization

After verifying the identity, Windows determines what this user/device is allowed to access.

## Why Is It Important?

Because it gives you one unified logical place to manage all identities and objects.

## What Exists Inside It?

* User accounts
* Computer accounts
* Groups
* OUs
* Its own domain controllers
* Domain admins

## How Does It Relate to the Other Concepts?

* It lives inside a **Forest**
* It contains **OUs**
* Its data is stored in the **Domain Partition**
* It is served by **Domain Controllers**
* It may have trust relationships with other Domains

## Why Is It Called a Replication Boundary?

Any change inside the domain is replicated to all Domain Controllers in the same domain. But if you have several Domains, not all domain data is fully replicated to the other Domains.

## Why Is It Called an Administrative Unit?

Because it has:

* Administrator account
* Domain Admins group
* unified management capability at the level of this domain

## Architectural Thinking Questions

### What problem does it solve?

Grouping and managing the identity of resources and users in one logical unit.

### What would break if it did not exist?

You would not have a unified scope for authentication and management, and you would go back to local accounts or impractical structures.

### What are the alternatives?

* Workgroup
* Local accounts
* Cloud-only identity systems

### When do I not use multiple Domains?

If the organization does not need real administrative, organizational, or legal separation. Usually, **one Domain is enough**.

### What are the trade-offs?

* One Domain: simpler
* Multiple Domains: more flexibility but more complexity

---

## 6.2 Domain Tree

## What Is It?

The **Domain Tree** is a group of Domains that share:

* a common **Root domain**
* a **Contiguous DNS namespace**

Example:

```text
contoso.com
seattle.contoso.com
emea.contoso.com
```

## Why Is It Important?

Because it shows how Domains can branch within the same naming path.

## How Does It Relate to the Other Concepts?

* Several Trees may live inside the same **Forest**
* Each Tree has a root domain within the forest
* Trusts inside the forest make them work together

## Architectural Thinking Questions

### What problem does it solve?

Organizing branching Domains within the same namespace.

### What would break if it did not exist?

You would lose this hierarchical naming structure, but not every environment needs it.

### What are the alternatives?

Separate Domains inside different Trees within the same forest.

### When do I not use it?

If you use only one Domain, or if you have a different namespace for each entity.

### What are the trade-offs?

It gives you good hierarchical organization, but it may increase complexity if you do not need it.

---

## 6.3 Forest

## What Is It?

The **Forest** is the highest container in AD DS. It consists of:

* one or more Domain trees
* one or more Domains
* shared Schema
* shared Global Catalog

The **Forest root domain** is the first Domain you create in the forest.

## Why Is It Very Important?

Because it defines very important boundaries:

### 1. Security Boundary

By default, users outside the forest do not have access inside it.

### 2. Schema Boundary

All Domains in the forest share the same schema.

### 3. Replication Boundary

Some data such as configuration and schema is replicated at the forest level.

### 4. Trust Foundation

All Domains inside the forest trust each other automatically.

## What Exists in the Forest Root Domain?

* Schema Master role
* Domain Naming Master role
* Enterprise Admins group
* Schema Admins group

## What Exists in Each Domain?

* RID Master
* Infrastructure Master
* PDC Emulator
* Domain Admins

## How Does It Relate to the Other Concepts?

* It contains Domains
* It enforces a shared schema
* It uses the Global Catalog
* It forms the trust framework between Domains

## Why Usually Only One Forest?

Because an additional Forest means:

* separate trust
* separate Schema
* more complex administration

Usually, the company needs only **one Forest**.

## Architectural Thinking Questions

### What problem does it solve?

Grouping the whole identity environment inside one security and logical boundary.

### What would break if it did not exist?

You would not have a unified framework for schema, trust, and enterprise-level administration.

### What are the alternatives?

Multiple forests, but this is not a simple alternative; it is a different separation model.

### When do I not use one Forest?

When you need:

* strong security separation
* a different Schema
* strong organizational independence

### What are the trade-offs?

* One Forest: simpler administration, easier sharing
* Multiple Forests: stronger isolation, but much higher complexity

---

## 6.4 OU (Organizational Unit)

## What Is It?

The **OU** is a Container object inside a Domain used to group:

* Users
* Computers
* Groups
* and other objects

**GPOs** can be linked directly to it.

It can also:

* assign an OU manager
* associate a COM+ partition with it

## Why Is It Important?

For two main reasons:

### 1. Applying Policies: GPOs

When you link a GPO to an OU, the settings are applied to the objects inside it.

### 2. Delegation

You can delegate management of an OU to a user or group without giving them Domain Admin.

## How Should You Think About It?

The OU is not just a visual folder. It is an **organization, management, and policy unit**.

## How Does It Relate to the Other Concepts?

* It lives inside a Domain
* It contains Users/Computers/Groups
* It is used to link GPOs
* It is used in delegation

## What Is the Difference Between It and a Container?

The Container organizes objects, but:

* **you cannot link a GPO to it**
* its administrative capabilities are lower

## Hierarchical Design

OUs can be created inside OUs.

Example:

```text
Domain
└── Berlin
├── IT
├── HR
└── Sales
```

It is usually recommended that the depth does not exceed 10 levels, and most organizations use 5 or fewer.

## Architectural Thinking Questions

### What problem does it solve?

Organizing objects in a manageable way while applying GPOs and delegating administration.

### What would break if it did not exist?

You would have to manage everything at the domain level or through weak containers, and administration and policies would become chaotic.

### What are the alternatives?

Containers, but with fewer capabilities.

### When do I not use a complex OU design?

When there is no real need for that depth. Excessive complexity does more harm than good.

### What are the trade-offs?

* Good OUs: flexible organization, clear policies, delegation
* Too many OUs: difficulty in understanding and administration

---

## 6.5 Container

## What Is It?

A Container is an object for organization inside AD DS, but it is not an OU.

## Built-in Container Examples

* Users
* Computers
* Builtin
* Foreign Security Principals
* Managed Service Accounts

## What Is the Problem?

Many people confuse Container and OU.

The most important difference:

* **You cannot link a GPO directly to a Container**
* You cannot build the same administrative flexibility that OUs provide

## Architectural Thinking Questions

### What problem does it solve?

It provides default or system locations for some objects.

### What would break if it did not exist?

You would lose some convenient built-in/default structures.

### What are the alternatives?

Creating custom OUs when better administration is needed.

### When do I not use it as the main place for managing devices and users?

When you need GPOs, hierarchical organization, or delegation.

### What are the trade-offs?

Useful as a default container, but administratively weaker than an OU.

---

# 7) Physical Components

Now we move to the real implementation.

## 7.1 Domain Controller (DC)

## What Is It?

The **Domain Controller** is the server that contains a copy of the AD DS database.

Every DC can, in most operations:

* process changes
* replicate them to the other DCs in the domain

This means AD DS works using a **Multi-master replication** model.

## Why Is It Important?

Because it is the actual node that performs:

* Authentication
* Directory lookups
* Replication
* the domain service itself

## How Does It Relate to the Other Concepts?

* It serves a specific Domain
* It holds a copy of the Data Store
* It may work as a Global Catalog
* It exists within a specific Site

## Architectural Thinking Questions

### What problem does it solve?

It provides an actual operating point for directory and identity services.

### What would break if it did not exist?

There would be no actual place where identities are verified or where the AD database is stored.

### What are the alternatives?

There is no real alternative inside AD DS. The DC is essential.

### When do I not use only one DC?

In real environments, you almost always need more than one DC for high availability.

### What are the trade-offs?

* Additional DC = higher availability
* but it increases administration, replication, and planning

---

## 7.2 Data Store

## What Is It?

A copy of the AD DS data store exists on every Domain Controller.

It includes:

* `Ntds.dit`
* log files

## Why Is It Important?

Because this is the actual storage of directory information.

## How Does It Relate to the Other Concepts?

* All Users/Computers/Groups/OUs are stored here
* Replication copies its data between DCs

## Architectural Thinking Questions

### What problem does it solve?

Storing directory information in an organized and reliable way on each DC.

### What would break if it did not exist?

There would be no actual directory.

### What are the alternatives?

There is none inside AD DS.

### When do I not interact with it directly?

Usually, as an administrator, you do not manually modify Ntds.dit; you manage AD through tools.

### What are the trade-offs?

Local copies on every DC provide flexibility and higher availability, but they require controlled replication.

---

## 7.3 Global Catalog Server

## What Is It?

It is a Domain Controller that hosts the **Global Catalog**, which is:

* a **partial** copy
* **read-only**
* of all objects in a multi-Domain Forest

## Why Is It Important?

It speeds up searching for objects that may exist in another Domain inside the same Forest.

## How Does It Relate to the Other Concepts?

* It works inside a Forest
* It helps in multi-domain environments
* It depends on partial replication

## Architectural Thinking Questions

### What problem does it solve?

It allows fast search and queries across multiple Domains.

### What would break if it did not exist?

Searching across the forest would become harder or slower, and some cross-Domain sign-in scenarios would be affected.

### What are the alternatives?

There is no direct alternative with the same role inside AD.

### When is its importance not very high?

In a very small single-Domain environment.

### What are the trade-offs?

Very useful in larger environments, but it adds extra partial replication load.

---

## 7.4 Read-Only Domain Controller (RODC)

## What Is It?

A special AD DS installation that runs in **read-only** mode.

## Why Does It Exist?

For branch offices that:

* have weaker physical security
* have weaker technical support
* need a local Domain Controller but without the full risks of a normal DC

## How Does It Relate to the Other Concepts?

* It is a type of DC
* It serves a specific location
* It fits with the concept of Sites and branches

## Architectural Thinking Questions

### What problem does it solve?

It provides AD services in remote locations while reducing risks.

### What would break if it did not exist?

You would either depend on a distant central DC, or deploy a full DC in a less secure place.

### What are the alternatives?

A normal DC or no local DC.

### When do I not use it?

In headquarters or places with good security and good administration.

### What are the trade-offs?

* Better security in branches
* but with fewer capabilities than a full DC

---

## 7.5 Site

## What Is It?

The **Site** is a Container that represents a **physical location** of the network.

It contains AD DS objects related to a specific location, such as:

* Computers
* Services

## What Is the Difference Between Site and Domain?

* **Domain**: logical administrative structure
* **Site**: physical representation of the network and location

## Why Is It Important?

Because AD needs to know:

* Where is the user?
* What is the closest DC to them?
* How should replication happen between locations?

## How Does It Relate to the Other Concepts?

* It contains Subnets
* It includes DCs within geographical/network locations
* It affects replication topology

## Architectural Thinking Questions

### What problem does it solve?

Improving performance, directing clients to the nearest DC, and managing replication between locations.

### What would break if it did not exist?

Users may connect to a distant DC, and replication would become suboptimal.

### What are the alternatives?

There is no direct alternative for the same function in AD.

### When are complex Sites not necessary?

In a small single-site environment.

### What are the trade-offs?

* Good Sites = better performance
* but they require correct network planning

---

## 7.6 Subnet

## What Is It?

A part of the organization’s IP addresses, assigned to devices inside a specific Site.

One Site may contain more than one Subnet.

## Why Is It Important?

Because AD links the client to the Site through the IP/Subnet.

## How Does It Relate to the Other Concepts?

* A Site consists of one or more Subnets
* Correct subnet definition helps direct the client to the correct DC

## Architectural Thinking Questions

### What problem does it solve?

Connecting the real network structure with AD Sites logic.

### What would break if it did not exist?

AD may not know where the client actually belongs.

### What are the alternatives?

There is no effective alternative; this is part of network design in AD.

### When is it not complex?

In a very simple network.

### What are the trade-offs?

It builds a precise connection between the network and AD, but it requires correct network management.

---

# 8) Users, Groups, Computers

## 8.1 User Objects

## What Are They?

Accounts that represent users who need access to network resources.

They contain:

* Username
* Password
* Group memberships
* other settings depending on the organization

## Why Are They Important?

Without user accounts, the user cannot:

* authenticate to the domain
* access resources

## How Do They Relate to the Other Concepts?

* They live inside a Domain
* They are usually placed inside OUs
* They are granted permissions through Groups
* They are managed through AD tools

## Management Tools

* Active Directory Administrative Center
* Active Directory Users and Computers
* Windows Admin Center
* Windows PowerShell
* dsadd

## Architectural Thinking Questions

### What problem does it solve?

Providing a centralized identity for the user.

### What would break if it did not exist?

The user would not be able to sign in or access resources in an organized way.

### What are the alternatives?

Local accounts, cloud identity, or a different LDAP.

### When do I not use an AD user account?

If the user is not part of the AD environment at all.

### What are the trade-offs?

Excellent centralized management, but it requires a correct AD structure.

---

## 8.2 Managed Service Accounts

## What Are They?

Many applications run services in the background. These services need a **Service Account** so that they can:

* start
* authenticate
* access resources

## The Problem with Traditional Accounts

* Password management is complex
* It is difficult to know where the account is used
* SPN management is difficult

## The Solution: Managed Service Account

An object type in AD that makes it easier to:

* manage the password
* manage the SPN

## Architectural Thinking Questions

### What problem does it solve?

Improving service account management compared to traditional accounts.

### What would break if it did not exist?

You would return to the problems of static passwords and manual administration.

### What are the alternatives?

Local Service / Network Service / Local System / domain-based normal account

### When do I not use it?

If the service is very simple or does not need this level of management.

### What are the trade-offs?

Better security and administration, but it requires correct understanding of the setup.

---

## 8.3 gMSA

## What Is It?

**Group Managed Service Account** extends the idea so it works on **more than one Server** in the domain.

Ideal in:

* NLB clusters
* IIS farms
* shared services across several servers

## The Important Requirement

You must create a **KDS root key** on a Domain Controller:

```powershell
Add-KdsRootKey –EffectiveImmediately
```

Then you can create a gMSA such as:

```powershell
New-ADServiceAccount -Name LondonSQLFarm -PrincipalsAllowedToRetrieveManagedPassword SEA-SQL1, SEA-SQL2, SEA-SQL3
```

## Why Is It Important?

Because a regular MSA is not enough when the same service runs on more than one server.

## Architectural Thinking Questions

### What problem does it solve?

Running the same service on several servers using the same account with automatic password management.

### What would break if it did not exist?

You would have to use a traditional shared domain account, with all its risks.

### What are the alternatives?

Traditional shared service account.

### When do I not use it?

If the service runs on only one Server.

### What are the trade-offs?

Better security and administration, but with additional setup requirements.

---

## 8.4 dMSA

## What Is It?

A new type in Windows Server 2025: **Delegated Managed Service Account**.

The basic idea:

* moving from traditional service accounts to machine-linked identity
* fully managed and randomized keys
* disabling old original passwords
* linking authentication to the device identity

## Why Is It Important?

Because it solves a serious problem in traditional accounts: **credential harvesting**.

## The Difference from gMSA

* gMSA: managed by AD, usually used on several Servers
* dMSA: managed by the Administrator, more closely linked to the identity of a specific device

Credential Guard can also improve dMSA security further.

## Architectural Thinking Questions

### What problem does it solve?

Reducing the risk of service account credential theft.

### What would break if it did not exist?

You would continue depending on weaker security models.

### What are the alternatives?

gMSA or traditional service accounts.

### When do I not use it?

If the environment does not support it or if the service does not need this model.

### What are the trade-offs?

Higher security, but understanding and configuring it is more complex.

---

## 8.5 Group Objects

## What Are They?

Groups are used to organize Users or Computers to make it easier to manage:

* Permissions
* Group Policy
* administration

## Why Are They Essential?

In small networks, you may give permission directly to a user, but in large networks this is impractical.

The golden rule:

> Give the permission to the group, then put users inside it

## Group Types

### Security

Used for security and permissions, and included in ACLs.

### Distribution

Usually used for email, and not security-enabled.

## Group Scopes

### Local

* exists on a local device
* its permissions apply only to local resources
* members can be from anywhere in the forest

### Domain-local

* used to manage access to resources inside the same domain
* members can be from anywhere in the forest

### Global

* used to group users with similar characteristics
* members are from the local domain only
* it can be granted permissions anywhere in the forest

### Universal

* useful in multi-domain environments
* members can be from anywhere in the forest
* it can be granted permissions anywhere in the forest

## How Do They Relate to the Other Concepts?

* Users and Computers can be members of them
* permissions are granted through them
* they are used in authorization after authentication

## Architectural Thinking Questions

### What problem do they solve?

Managing permissions and memberships at scale in a scalable way.

### What would break if they did not exist?

Permissions would become directly assigned to users, and manageability would collapse.

### What are the alternatives?

Direct permissions, but that is much worse.

### When do I not use a Distribution group for security?

Always. For security, you need a Security group.

### What are the trade-offs?

Groups are excellent, but poor scope design causes chaos.

---

## 8.6 Computer Objects

## What Are They?

Computers, like users, are considered **Security Principals**.

This means they:

* have an account
* have a logon name and password that changes automatically and periodically
* authenticate with the domain
* can be members of Groups
* can have GPOs applied to them

## Their Lifecycle

* create computer object
* join to domain
* manage properties
* move between OUs
* rename
* reset
* disable
* enable
* delete

## Computers Container

When a device joins the domain, the default location is often:

```text
CN=Computers
```

But it:

* is not an OU
* cannot have an OU created inside it
* cannot have a GPO linked to it

So it is usually better to use **custom OUs for devices**.

## Architectural Thinking Questions

### What problem does it solve?

Giving the device a member identity inside the domain, not only the user.

### What would break if it did not exist?

You would not be able to manage devices centrally, or apply policies to them correctly.

### What are the alternatives?

Workgroup devices or local management.

### When do I not rely on the Computers container?

When I need real organization and GPO policies.

### What are the trade-offs?

Computer objects are very powerful for administration, but they need a correct OU design.

---

# 9) Forests and Domains and Trusts

This section connects everything above.

## What Is the Relationship Between Forest and Domain?

* Forest = highest container
* Domain = logical unit inside it
* Domains inside the same Forest:

* shared schema
* shared global catalog
* automatic trust

## Trust Relationships

In a multi-Domain or multi-Forest environment, trusts enable access to resources.

## Inside the Same Forest

**two-way transitive trusts** are created automatically between Domains.

This means:
If A trusts B, and B trusts C, then A trusts C.

## The Main Types

### Parent and child

* Transitive
* Two-way

### Tree-root

* Transitive
* Two-way

### External

* Nontransitive
* One-way or two-way
* for access with Windows NT 4.0 or a Domain in another Forest

### Realm

* Transitive or nontransitive
* One-way or two-way
* with non-AD Kerberos v5 realms

### Forest trust

* Transitive
* One-way or two-way
* between Forests

### Shortcut

* Nontransitive
* One-way or two-way
* to reduce authentication time between distant Domains in the same Forest

When creating a trust, Windows creates a **trusted domain object** inside the **System container**.

## Architectural Thinking Questions

### What problem do they solve?

Allowing organized access to resources between Domains and Forests.

### What would break if they did not exist?

Every Domain or Forest would become an isolated island.

### What are the alternatives?

Separate accounts or manual access processes, which are impractical.

### When do I not use an additional Trust?

When the environment is within one Forest and everything works automatically.

### What are the trade-offs?

Trusts are very powerful, but they increase security and administrative complexity if misused.

---

# 10) Generic Containers and OUs in Practice

After installing AD DS, default objects appear such as:

* Domain
* Builtin
* Computers
* Foreign Security Principals
* Managed Service Accounts
* Users
* Domain Controllers OU

With Advanced Features, the following also appear:

* LostAndFound
* Program Data
* System
* NTDS Quotas
* TPM Devices

## Why Is This Important?

Because you must know what is:

* a built-in/system object
* and what is a suitable place for your administrative design

The practical rule:

> Do not build your entire administrative design on the default containers, especially when you need GPOs and delegation. Use well-designed OUs.

---

# 11) Managing AD DS: The Tools

## 11.1 Active Directory Administrative Center

A GUI interface built on Windows PowerShell.

From it, you can:

* create and manage users/computers/groups
* create and manage OUs
* manage several domains
* search and query
* fine-grained password policies
* Active Directory Recycle Bin
* Dynamic Access Control objects

## 11.2 Windows Admin Center

A web-based interface for managing Windows servers and Windows 10.

It is often used to manage servers instead of RSAT in some cases.

Important notes:

* It is not recommended to install it on a Domain Controller
* The default port:

* 6516 on standalone Windows 10
* TCP 443 on Windows Server gateway mode
* On first launch, you may need to choose a Windows Admin Center Client certificate

## 11.3 RSAT

A set of tools for managing Windows Server roles/features remotely.

It is enabled from Settings > Manage optional features > Add a feature.

## 11.4 Other Tools

* Active Directory module for Windows PowerShell
* Active Directory Users and Computers
* Active Directory Sites and Services
* Active Directory Domains and Trusts
* Active Directory Schema snap-in

## Why Is This Important?

Because AD administration is not only a concept; it is also a practical daily workflow.

## Architectural Thinking Questions

### What problem do these tools solve?

They make managing an AD environment practically possible and efficient.

### What would break if they did not exist?

Administration would become manual or very weak.

### What are the alternatives?

PowerShell only in some cases, but usually you also need graphical tools.

### When do I not rely on only one tool?

In real environments, you need to know more than one tool.

### What are the trade-offs?

* GUI: easier and faster for some tasks
* PowerShell: more powerful for automation and precision

---

# 12) The Big Relationship — How Does Everything Connect Together?

Now we connect everything.

## Step 1: Forest

Represents the highest level. It enforces:

* schema
* general security boundaries
* global catalog
* trust framework

## Step 2: Domain

Inside the forest, it represents:

* authentication unit
* basic administration
* storage of users/computers/groups in the domain partition

## Step 3: OU

Inside the domain, it represents:

* organization
* policy targeting
* delegation

## Step 4: Users / Computers / Groups

These are the objects we actually manage:

* User = human identity or account
* Computer = device identity
* Group = grouping and permissions mechanism

## Step 5: DCs

These are the servers that actually run AD:

* store the database
* perform authentication
* replicate changes

## Step 6: Sites/Subnets

These connect the logical design to the real network:

* which client goes to which DC
* how replication happens between locations

## Step 7: GPOs

They are usually linked to OUs to apply settings to Users/Computers.

## Step 8: Service Accounts

They run services securely inside this system.

---

# 13) A Real Operating Scenario

You have:

```text
Forest: contoso.com forest
Domain: contoso.com
OU: Berlin\IT
User: Ali
Computer: BER-CL-01
Group: IT-Admins
DCs: BER-DC1, DUB-DC1
Site: Berlin / Dubai
```

What happens?

1. **Ali** exists as a User object inside the Domain
2. His device **BER-CL-01** also has a Computer object
3. Both are inside a suitable OU
4. The OU is linked to a GPO
5. Ali is a member of the IT-Admins group
6. When signing in:

* the device contacts the nearest DC according to Site/Subnet
* the DC verifies the identity
* Ali’s permissions are built using group memberships
* the GPOs linked to the OU are applied
7. If Ali needs a resource in another Domain inside the same Forest:

* internal trust and Global Catalog help

This is AD DS as an integrated system.

---

# 14) The Final Golden Summary

> **AD DS is a distributed identity management system in Windows. It stores objects in a replicated database, organizes them logically through Forest/Domain/OU, implements them physically through Domain Controllers/Sites/Subnets, controls access and policies through Groups and GPOs, and runs services securely through managed service accounts.**

And the most important relationships:

* **Forest**: highest level, schema + trust + global catalog
* **Domain**: administration and authentication unit
* **OU**: organization, policies, and delegation unit
* **Users/Computers**: security principals
* **Groups**: permission management
* **DC**: the node that runs AD
* **Site/Subnet**: connecting AD to the physical network
* **GPO**: enforcing settings
* **MSA/gMSA/dMSA**: running services securely
