## DNS (Zones + Records + Secondary/Transfers + Load Balancing + MX)

This section delivers a corporate-style DNS setup for an Active Directory environment.  
It covers:

- Forward Lookup Zones (primary + secondary for redundancy)
- Reverse Lookup Zones (name-to-IP and IP-to-name consistency)
- Zone Transfers (controlled replication from primary DNS to secondary DNS)
- Host records (A records) for servers and services
- Web DNS naming (friendly DNS names like `www.iq.networks`)
- DNS “Load Balancing” using a multi-IP A record
- MX record design for Exchange mail flow

All values align with the Unified Lab Profile:

- DomainName: `iq.networks`
- Primary DNS / DC1: `SRV-DC1-GUI` (`192.168.10.10`)
- Secondary DNS / DC2: `SRV-DC2-CORE` (`192.168.10.12`)
- Subnet: `192.168.10.0/24`

---

### DNS Redundancy Design (Primary + Secondary DNS)

Goal:  
The core server (`SRV-DC2-CORE`) acts as a backup DNS server for the main server (`SRV-DC1-GUI`).  
If the primary DNS becomes unavailable, clients can still resolve names using the secondary DNS.

Corporate reasoning:
- DNS is critical for Active Directory logon, domain join, and service discovery
- A single DNS server is a single point of failure
- Secondary DNS improves resiliency

---

## Install the DNS Server Role on the Secondary Server (SRV-DC2-CORE)

We install the DNS role on the Core server from the main server using Server Manager.

Steps (on `SRV-DC1-GUI`):

1. Open **Server Manager**
2. Go to **Manage** → **Add Roles and Features**
3. Click **Next** → **Next** until you reach **Server Selection**
4. Select the target server:
   - `SRV-DC2-CORE`
5. Under **Server Roles**, select:
   - **DNS Server**
6. Click **Next** → **Next** → **Next**
7. Click **Install**

Result:  
The DNS server role is installed on `SRV-DC2-CORE`.

---

## Forward Lookup Zone Setup (Primary on SRV-DC1-GUI, Secondary on SRV-DC2-CORE)

### important:
In an Active Directory environment, the zone `iq.networks` is normally created as an **AD-integrated zone**.

- In that case, you **do NOT need** to uncheck  
  **Store the zone in Active Directory**  
  to achieve redundancy.

Because AD-integrated DNS already replicates zone data automatically to other domain controllers running DNS.

So in this lab, you have **two valid designs**:

---

### Option A (Recommended): AD-Integrated DNS on both Domain Controllers

This is the standard corporate design for internal AD domains.

What you do:
- Keep the primary zone `iq.networks` as **AD-integrated**
- Install DNS on `SRV-DC2-CORE`
- DNS data will replicate via Active Directory replication (no classic zone transfer needed)

Benefits:
- Cleaner AD design
- No manual zone transfers
- More secure by default

---

### Option B (Classic Secondary Zone): Use Zone Transfers (Manual DNS Replication)

This is a valid lab scenario for learning classic DNS behavior, but it is less common inside modern AD domains.

Important requirement:
- The primary zone must allow **zone transfers**
- The secondary server pulls the zone from the master DNS server

Note:
Unchecking **Store the zone in Active Directory** changes the zone type away from AD-integrated, which is not typical for DC-hosted AD zones.

---

### Create the Secondary Zone on SRV-DC2-CORE (Classic Secondary Zone Design)

Steps:

1. Open **DNS Manager**
2. Select the Core server:
   - `SRV-DC2-CORE`
3. Expand:
   - **Forward Lookup Zones**
4. Right-click **Forward Lookup Zones** → **New Zone**
5. Zone type:
   - **Secondary zone**
6. Zone name:
   - `iq.networks`
7. Master DNS Server IP:
   - `192.168.10.10`
8. Finish

Result:  
On `SRV-DC2-CORE`, the zone `iq.networks` appears under Forward Lookup Zones as a secondary zone.

---

## Zone Transfers (Allowing SRV-DC2-CORE to receive the zone)

Zone transfers must be explicitly allowed on the primary server.

Steps (on `SRV-DC1-GUI`):

1. Open **DNS Manager**
2. Go to:
   - **Forward Lookup Zones** → `iq.networks`
3. Right-click the zone → **Properties**
4. Go to the **Zone Transfers** tab
5. Enable **Allow zone transfers**
6. Select:
   - **Allow zone transfers only to the following servers**
7. Click **Edit**
8. Add the allowed servers (IP addresses):
   - `192.168.10.12` (secondary DNS / core)
9. Apply → OK

Note:
- You only need to add the **secondary server** (`192.168.10.12`) here.
- Adding the primary itself (`192.168.10.10`) is not needed.

Result:  
`SRV-DC2-CORE` is authorized to pull zone data from the primary DNS.

---

## Add the Secondary Server as an additional Name Server (NS record)

After configuring secondary DNS, the secondary server should be listed as an authoritative name server.

Steps:

1. Under **Forward Lookup Zones**, right-click:
   - `iq.networks`
2. Go to **Properties**
3. Go to the **Name Servers** tab
4. Click **Add**
5. Add:
   - Server name: `SRV-DC2-CORE`
   - IP address: `192.168.10.12`
6. Apply → OK

Result:  
The DNS zone lists both name servers correctly.

---

## Reverse Lookup Zone Setup (IP-to-Name Resolution)

Reverse zones allow you to resolve:
- IP → hostname

This is useful for:
- troubleshooting
- logging
- network consistency

Steps:

1. Open **DNS Manager**
2. Right-click:
   - **Reverse Lookup Zones**
3. Click **New Zone**
4. Zone type:
   - **Primary zone**
5. Follow Next → Next → Next
6. Enter the Network ID:
   - `192.168.10`
7. Finish

Result:  
A reverse lookup zone exists for the subnet `192.168.10.0/24`.

---

## Create DNS Host Records (A Records)

### Create a standard host record (A record)

Steps (on `SRV-DC1-GUI`):

1. Open **DNS Manager**
2. Expand:
   - **Forward Lookup Zones**
3. Expand the zone:
   - `iq.networks`
4. Right-click the zone → **New Host (A or AAAA)...**
5. Enter:
   - Host name: `app01` (example)
   - IP address: `192.168.10.20`
6. Click **Add Host**

Result:  
A new DNS record is created:
- `app01.iq.networks` → `192.168.10.20`

---

## Web DNS Naming (Friendly internal web name)

### Why companies use `www` names

In real companies, teams host internal web services and portals.  
Instead of accessing services by IP address, DNS gives a stable name.

Example:
- `www.iq.networks`
instead of:
- `http://192.168.10.x`

Benefits:
- Easier to remember
- IPs can change without impacting users
- AD-integrated DNS resolves names inside the network

---

### Create a DNS record for `www.iq.networks`

Steps:

1. Open **DNS Manager**
2. Expand:
   - **Forward Lookup Zones**
3. Open the zone:
   - `iq.networks`
4. Right-click the zone → **New Host (A or AAAA)...**
5. Host name:
   - `www`
6. IP address:
   - Choose the IP of the web server that hosts the service

Example used here:
- `192.168.10.12`

7. Click **Add Host**
8. If it does not appear immediately:
   - Right-click the zone → **Refresh**

Verify:
```cmd
ping www.iq.networks
```

Final result:  
Users can access internal services by name instead of IP.

---

## DNS “Load Balancing” using a Multi-IP A Record (`www.iq.networks`)

Goal:  
Create DNS “load balancing” by assigning multiple IP addresses to the same DNS name.

Concept:  
When a DNS name has multiple A records, clients may receive different IP answers, distributing requests across servers.

Steps:

1. Open **DNS Manager**
2. Expand:
   - **Forward Lookup Zones**
3. Open:
   - `iq.networks`
4. Create (or edit) the host record:
   - Host name: `www`
5. Add multiple IP addresses

Example:
- IP1: `192.168.10.10`
- IP2: `192.168.10.12`

Result:  
`www.iq.networks` resolves to multiple IP addresses, enabling basic DNS-level distribution.

---

## Mail (Exchange) DNS — Create an MX Record

### Why companies use Exchange and MX records

Many companies depend on email for business operations.  
An MX record routes incoming emails to the correct mail server.

This setup can support:
- Internal mail handling inside the network
- External mail delivery via DNS + firewall routing

---

### Create the mail server host record (A record)

Steps:

1. Open **DNS Manager**
2. Go to:
   - **Forward Lookup Zones**
3. Select your zone:
   - `iq.networks`
4. Create a new A record:
   - Right-click zone → **New Host (A or AAAA)...**
5. Name:
   - `mail`
6. IP address (example):
   - `192.168.10.11`
7. Click **Add Host**

Result:  
You created:
- `mail.iq.networks` → `192.168.10.11`

---

### Create the MX record (Mail Exchanger record)

Steps:

1. In **DNS Manager**, inside the zone `iq.networks`
2. Right-click the zone → **Other New Records...**
3. Choose:
   - **Mail Exchanger (MX)**
4. Set the mail server hostname to:
   - `mail.iq.networks`

Purpose:  
This tells mail systems that incoming email for `iq.networks` should be routed to `mail.iq.networks`.

Important production note:  
In real production environments, MX records typically point to a public IP, not a private internal IP.  
That public IP is configured on the firewall and forwarded to the internal Exchange server.

Final result:  
You created the DNS A record for the mail server and configured MX routing logic for email delivery.
