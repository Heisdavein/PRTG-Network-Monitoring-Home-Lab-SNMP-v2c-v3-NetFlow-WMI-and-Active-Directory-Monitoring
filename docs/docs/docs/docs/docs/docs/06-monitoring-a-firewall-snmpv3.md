# 6. Monitoring a Firewall with SNMP Version 3

## Overview

Every device so far used SNMP v2c. It is simple and fine for a home lab, but a production firewall protects an entire network, so it needs something stronger. **SNMP v3** adds real authentication *and* encryption. It does not just check that the monitoring server is allowed to ask; it also stops anyone on the network from reading the monitoring traffic.

That matters for firewalls because they expose interface details, VPN information, CPU and memory use and traffic patterns that could help an attacker. Protecting that information is as important as collecting it.

---

## SNMP v2c vs SNMP v3

| Feature | SNMP v2c | SNMP v3 |
| ------- | -------- | ------- |
| Authentication | Community string (plaintext) | Username with authentication password |
| Encryption | None | DES, AES-128 or AES-256 (device dependent) |
| Security | Low | High |
| Credentials | Single community string | Username, authentication password and encryption password |
| Best use | Home labs and trusted internal networks | Production and security-critical devices |

With v2c, anyone who discovers the community string can query the device. With v3 an attacker would need the username, the authentication password *and* the encryption password, and would still be unable to read the traffic.

> [!IMPORTANT]
> SNMP v1 should never be used. SNMP v2c is only acceptable inside trusted internal environments. For routers, firewalls, VPN concentrators and other security-sensitive devices, SNMP v3 should be the first choice.

---

## Step 1: Configure SNMP v3 on the WatchGuard Firewall

I logged into my **WatchGuard Firebox** management interface and went to **System → SNMP**. I enabled SNMP, selected **Version 3**, and created a dedicated monitoring account:

| Setting | Configuration |
| ------- | ------------- |
| Username | `snmpv3user` |
| Authentication protocol | SHA-1 |
| Authentication password | Strong unique password |
| Privacy protocol | DES |
| Privacy password | A different strong password |

DES was the only encryption option available on my appliance, but many enterprise devices now support **AES-128 or AES-256**, which are significantly stronger and should always be chosen when available.

SNMP v3 separates authentication from encryption: the **authentication password** proves who I am, and the **privacy password** encrypts the conversation. Using different passwords for these two jobs improves security.

I then configured **SNMP traps**. Instead of waiting for PRTG's sixty-second poll, traps let the firewall send important events straight away. I set the trap destination to my PRTG server's IP address and saved.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The WatchGuard Firebox System > SNMP page with Version 3 selected, the snmpv3user account, SHA-1 and DES, and the trap receiver IP (do not show passwords).

**Concepts carry across vendors.** Whether Cisco, Palo Alto, Fortinet or WatchGuard, every platform asks for an SNMP username, authentication protocol, authentication password, encryption protocol, encryption password and the monitoring server's IP address. Those concepts are worth more than memorising any one interface.

---

## Step 2: Configure SNMP v3 at the Group Level in PRTG

1. I right-clicked **Local Probe**, selected **Add Group** and named it `Firewalls` (I covered the group in [document 3](03-organising-devices-with-groups.md)).
2. In the group's settings I scrolled to **Credentials for SNMP Devices**.
3. I disabled **Inherit from Parent**, because the default credentials were set for SNMP v2.
4. I entered the v3 settings:

| Setting | Value |
| ------- | ----- |
| Version | SNMP v3 |
| Authentication | SHA |
| Username | `snmpv3user` |
| Authentication password | My configured password |
| Encryption | DES |
| Encryption key | Privacy password |

Every firewall I add to this group now inherits these credentials. If I monitored twenty firewalls and the SNMP password changed, I would update it once at the group level instead of editing twenty devices.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The Firewalls group settings > Credentials for SNMP Devices with Inherit from Parent unticked and SNMP v3 settings entered (passwords hidden).

---

## Step 3: Add the WatchGuard Firewall

| Setting | Value |
| ------- | ----- |
| Parent group | Firewalls |
| Device name | `DMT-FW-01` |
| IP version | IPv4 |
| IP address | WatchGuard management IP |
| Device icon | WatchGuard firewall icon |

I did not need to enter any SNMP credentials, because they were inherited from the group. That confirmed the inheritance model works as intended.

---

## Step 4: Run Auto-Discovery

This time I let PRTG discover sensors, because WatchGuard devices are well supported. I right-clicked the firewall and selected **Run Auto-Discovery**. Over a few minutes PRTG read the firewall's MIB and added:

- Ping
- CPU Usage
- Memory Usage
- Interface Traffic
- Disk Free sensors
- System Uptime

The interface traffic sensors were immediately the most useful. Each interface showed separate graphs for inbound bandwidth, outbound bandwidth and utilisation over time, which would quickly show whether the WAN interface was saturated if I were troubleshooting a slow internet connection.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The Auto-Discovery results for DMT-FW-01 with the discovered sensors listed.

---

## Step 5: Review the Sensors

| Sensor | What it tells me |
| ------ | ---------------- |
| **Ping** | The foundation of everything. If it fails, every other sensor will eventually error because PRTG cannot reach the firewall |
| **CPU Usage** | A sudden spike might indicate heavy VPN traffic, denial-of-service attacks, excessive logging or overloaded firewall policies |
| **Memory Usage** | Memory that steadily falls over days or weeks without recovering could mean a memory leak or another software issue |
| **Disk Free** | See below |

PRTG created several **Disk Free** sensors. Because the WatchGuard operating system is Linux-based, each filesystem mount point appears separately, and some are system partitions that rarely change. Monitoring every one would waste sensors from my 100-sensor allowance, so I **deleted the partitions that were not useful** and kept only the relevant ones.

---

## Step 6: Sensor Priorities

PRTG gives each sensor a **priority rating shown as stars**. I first thought it was cosmetic, but the documentation showed it affects which sensors get the most visibility on the dashboard.

| Priority | Sensors |
| -------- | ------- |
| 5 stars | Ping, CPU Usage, Primary WAN interface, Trusted LAN interface |
| 3 stars | Less critical sensors, such as secondary filesystem usage |

During an incident the most important information now rises to the top instead of me searching through dozens of sensors.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The DMT-FW-01 sensor list with star priorities visible (five stars on Ping, CPU, WAN and LAN interfaces).

---

## Security Best Practices I Learned

- Use long, unique authentication and privacy passwords.
- Never reuse SNMP credentials across unrelated environments.
- Restrict SNMP so only the PRTG server can talk to monitored devices.
- Choose **SHA** instead of MD5 when authentication options exist.
- Choose **AES** instead of DES whenever the hardware supports it.
- Review SNMP users regularly and remove unused accounts.
- Keep monitoring software and firewall firmware up to date.

> [!WARNING]
> My lab uses **SHA-1 and DES** because that is what my appliance offered. DES is considered weak. In production I would use AES and the strongest authentication available.

---

## Verification

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| Device added with inherited SNMP v3 credentials | No credentials needed on the device | None needed | ✅ |
| Auto-Discovery | Sensors created from the firewall's MIB | Ping, CPU, Memory, Interface Traffic, Disk Free and Uptime sensors created | ✅ |
| Sensor review | Unneeded Disk Free sensors removed | Removed, relevant ones kept | ✅ |
| Priorities | Key sensors ranked highest | Five stars on Ping, CPU, WAN and LAN | ✅ |

---

## Visual Summary

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The "Section 6: Monitoring a Firewall with SNMP Version 3" summary graphic from my original documentation (page 28 of the PDF).

---

## What I Took Away

Monitoring is not just about visibility; it is also about **protecting the visibility itself**. SNMP v3 is more complex than v2c, but that complexity is justified for devices on the edge of the network. I also saw how credential inheritance at the group level saves enormous time as an environment grows.

---
