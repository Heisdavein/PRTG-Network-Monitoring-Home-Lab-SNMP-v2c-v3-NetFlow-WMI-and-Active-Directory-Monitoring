# 9. Monitoring Windows Server Roles and Services

## Overview

So far I had monitored the health of the server itself (CPU, memory, disk, traffic, availability). But a healthy operating system does not mean the services running on it are healthy. A common mistake is building monitoring that says a server is *alive* but not whether the business applications on it are *working*.

Users do not care that a server's CPU is at 18%. They care whether they can log in, open shared folders, browse company websites, send email, reach databases and authenticate to applications. Those abilities come from **server roles and services**, not from the operating system itself.

---

## Why Role Monitoring Matters

Picture 8:47 on a Monday morning. PRTG shows a domain controller with CPU 12%, memory 48%, disk 74% free, a healthy network adapter and successful pings. Every infrastructure metric is green.

But the **Active Directory Domain Services (AD DS)** service has crashed. Within minutes employees cannot log in, Group Policy stops applying, file shares become inaccessible, dependent applications fail and the helpdesk phones ring non-stop. From the operating system's view nothing is wrong; from the users' view the whole organisation is offline.

- **Without service monitoring** IT finds out from users.
- **With service monitoring** PRTG detects the failure the moment the service stops and sends an alert.

That is the difference between reactive and proactive IT.

### Common Windows Server roles

| Server role | Purpose |
| ----------- | ------- |
| Active Directory Domain Services (AD DS) | Authenticates users and computers |
| DNS Server | Resolves computer names into IP addresses |
| DHCP Server | Automatically assigns IP addresses |
| File Services | Provides network file shares |
| Print Services | Hosts shared printers |
| IIS (Internet Information Services) | Hosts websites and web applications |
| Hyper-V | Runs virtual machines |
| Certificate Services | Issues digital certificates |
| Remote Desktop Services | Provides remote application access |
| Windows Deployment Services | Deploys Windows operating systems |

Each role is made of one or more Windows services. If those services stop, the role stops working even if Windows is fine. PRTG can monitor each service individually.

---

## Step 1: Add an LDAP Sensor for Active Directory

My server was a Domain Controller, so Active Directory was the first role I monitored. The **LDAP Directory Services** sensor does more than check that TCP port 389 is open: it runs an actual LDAP query against the directory, proving that AD can answer authentication-type requests.

1. I selected **DMT-AD-01** and clicked **Add Sensor**.
2. I searched for `LDAP` and selected **LDAP Directory Services**.
3. I configured it:

| Setting | Value |
| ------- | ----- |
| Distinguished Name (DN) | `DC=corp,DC=danieltraining,DC=com` |
| Username | `administrator@corp.danieltraining.com` |
| Password | Domain administrator password |
| Port | 389 (default LDAP; LDAPS would use 636) |
| Timeout | 60 seconds |

The **Distinguished Name** identifies the root of the Active Directory tree, the starting point where LDAP begins searching.

Every polling interval (typically once a minute) the sensor checks that:
- LDAP is accepting connections,
- the supplied credentials authenticate,
- Active Directory responds correctly,
- queries finish within the configured timeout.

If any check fails, the sensor changes to a warning or error state. That gives far more confidence than a ping.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The "Add Sensor: LDAP Directory Services" configuration page with the Distinguished Name, username, port 389 and timeout 60 seconds (password hidden).

---

## Step 2: Monitor Active Directory Replication

Most enterprises use several domain controllers across offices that must continuously replicate changes (password changes, new users, domain joins, Group Policy updates). If replication fails, controllers hold different data, which causes confusing problems: users able to log in at one office but not another, intermittent password changes, Group Policy not applying, vanishing new accounts and inconsistent DNS records. These can stay hidden for weeks.

1. I clicked **Add Sensor**.
2. I searched for `Active Directory`.
3. I selected **WMI Active Directory Replication Errors**.
4. I clicked **Create**.

> [!NOTE]
> My lab has only **one** domain controller, so there are no replication partners and this sensor has little to report. In production with multiple domain controllers it is one of the most valuable sensors available.

---

## Step 3: Monitor Critical Windows Services

Roles are built from services, and PRTG can alert me the instant an important one stops.

1. I clicked **Add Sensor**.
2. I searched for `Windows Service`.
3. I selected **WMI Service Monitor**.
4. PRTG listed the installed services, and I selected the essential ones.

| Service | What it does | What happens if it stops |
| ------- | ------------ | ------------------------ |
| **DNS Server** | Converts hostnames to IP addresses | Websites fail, applications cannot find servers and AD authentication has problems. The server is online but users see the network as down |
| **Netlogon** | Handles communication between computers and domain controllers: authenticating users, locating domain controllers, establishing secure channels, processing domain logons | Users cannot authenticate |
| **Windows Time (W32Time)** | Keeps clocks synchronised. Kerberos (AD's authentication protocol) needs clocks to stay close together, and even a five-minute difference can break authentication | Subtle but serious authentication failures |
| **DFS Replication (DFSR)** | Keeps the SYSVOL folder (Group Policies, login scripts, domain-wide configuration files) synchronised between domain controllers | Different domain controllers may deliver different Group Policies |

Each service becomes its own sensor and reports Running, Stopped, Warning or Error. If one stops, PRTG alerts me with the affected server, the exact service and the time of failure, so I know exactly where to start troubleshooting.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The WMI Service Monitor service selection list with DNS Server, Netlogon, Windows Time and DFS Replication ticked.

---

## Step 4: Role-Specific Sensors I Explored

I browsed PRTG's sensor library for other Microsoft workloads. These were **explored for learning and not deployed in my lab**.

| Sensor family | What it can monitor |
| ------------- | ------------------- |
| **DNS** | Actual DNS lookups: forward, reverse, response times, specific records |
| **Microsoft Exchange** | Mailbox databases, mount status, replication health, mail queues, Outlook Web Access, transport services |
| **IIS web server** | HTTP response codes, page response times, concurrent connections, application pools, website availability |
| **Microsoft SQL Server** | Database availability, backup status, replication, query performance, transaction log growth, storage use |
| **Hyper-V** | VM health, replication status, checkpoint age, CPU and memory allocation, storage use |
| **RDP** | Whether Remote Desktop connections can actually be established (a ping does not guarantee remote access) |
| **RADIUS** | Authentication response times, request failures and server availability (valuable for wireless and VPN) |

---

## Step 5: Custom Monitoring with PowerShell (Explored)

PRTG supports **custom PowerShell sensors** for anything no built-in sensor covers. The workflow:

1. Write a PowerShell script.
2. The script checks something specific.
3. It returns an exit value.
4. PRTG reads the result.
5. PRTG generates graphs and alerts automatically.

A simple script might return `0` = Healthy, `1` = Warning, `2` = Critical. Examples of what could be checked: Hyper-V replication status, VPN tunnel availability, Azure AD synchronisation, Microsoft 365 connectivity, SharePoint availability, SQL backup completion, application log errors, disk encryption status, certificate expiry and third-party application health. PRTG then treats it like any built-in sensor, with graphs, reports and alerts.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The example PowerShell custom sensor script and exit code meaning from the Section 9 summary graphic (page 45 of the PDF), if I built one; otherwise remove this placeholder.

---

## Verification

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| LDAP sensor | Connects, authenticates and queries AD within the timeout | Sensor created and polling every minute | ✅ |
| AD replication sensor | Created; little activity with one domain controller | Created; little activity expected | ✅ |
| Service sensors | DNS Server, Netlogon, W32Time and DFSR each report Running | Each became an independent sensor | ✅ |

---

## Visual Summary

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The "Section 9: Monitoring Windows Server Roles and Services" summary graphic from my original documentation (page 45 of the PDF).

---

## What I Took Away

This lab changed how I think about server monitoring. A server can have low CPU, plenty of memory and ample disk while still failing to deliver the services users depend on. By monitoring Active Directory, DNS, Windows services and replication health, I moved from monitoring *infrastructure* to monitoring *business functionality*, and I can detect problems at the service level before users feel them.

---
