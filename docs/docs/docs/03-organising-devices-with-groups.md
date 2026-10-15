# 3. Organising Devices with Groups


## Overview

Before adding my first monitored device, I stopped to plan how I wanted to organise the environment. It is tempting to start adding devices straight away, but a few extra minutes of planning saves a lot of time later.

PRTG organises everything in a hierarchy:

```text
Groups → Devices → Sensors
```

Every object **inherits settings** from the level above it:

- A **group** can contain multiple devices.
- A **device** can contain dozens or even hundreds of sensors.
- Credentials, scanning intervals, notification templates and SNMP settings can all be inherited automatically.

Instead of configuring every device, I can set something once at the right level and every child object picks it up. Even a modest network of fifty devices can have several hundred sensors, and without structure, finding a specific server or switch during an outage is slow and frustrating.

---

## Grouping Strategies

There is no single correct way to organise devices. These are the strategies I found in production-style deployments:

| Strategy | Description | Best for | Example |
| -------- | ----------- | -------- | ------- |
| **By function** | Group by what devices do, not where they are | Small to medium organisations, one IT team | Network Infrastructure, Servers, Storage, Firewalls, Wireless, Virtualisation |
| **By location** | Group by physical office or site | Large organisations with multiple sites | London, Manchester, Birmingham, Edinburgh (or New York, Chicago, Dallas) |
| **By technology** | Group by vendor or platform | Organisations with specialist teams | Cisco, VMware, Microsoft Windows, Linux, NetApp, Synology |
| **By criticality** | Group by business importance | Environments where alert prioritisation matters | Tier 1 (firewalls, domain controllers, core switches, storage arrays), Tier 2 (departmental apps, print servers, backup infrastructure), Tier 3 (lab, development, test) |
| **Hybrid** | Combine strategies | Large enterprises | London > Network, Servers, Storage; Manchester > Network, Servers, Storage |

Because my lab is a single site and I manage every device, I chose **organising by function**.

---

## My Group Structure

```text
Local Probe
│
├── Network Infrastructure
│   └── Switching
│
├── Servers
│
├── Storage
│
└── Firewalls
```

It looks simple, but leaves room to grow. I can add groups for wireless access points, routers, VMware hosts, backup servers or cloud infrastructure without reorganising anything.

---

## Creating the Groups

1. From **Devices**, I located the **Local Probe**, which is the root of the monitoring tree. Everything belongs beneath it.
2. I right-clicked **Local Probe** and selected **Add Group**.
3. I named the first group `Network Infrastructure`.
4. PRTG asked about automatic discovery. I **left it disabled**, because I wanted to add every device manually and understand each protocol.
5. I repeated the process for three more top-level groups: `Servers`, `Storage` and `Firewalls`.
6. I right-clicked **Network Infrastructure**, selected **Add Group**, and created a subgroup called `Switching`. This is where my Cisco Catalyst switch went later.

Subgroups I could add as the lab grows: Routing, Wireless, Load Balancers, WAN and VPN Appliances.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/0c32c765-40d2-4b25-aff2-aa25ba632e09" />
The "Add Group" dialog on Local Probe with Group Name "Network Infrastructure" and auto-discovery option.


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/06430796-a55a-434f-9bd8-e45d9d1b1a48" />
The finished Devices tree with Network Infrastructure (and its Switching subgroup), Servers, Storage and Firewalls beneath Local Probe.

---

## Understanding Inheritance

Inheritance lets child objects automatically receive settings from their parent group. For example, if every Cisco switch uses the same SNMP community string, I configure it once at the **Switching** group level and every switch placed inside inherits it. If I add ten more switches later, no extra configuration is needed.

Settings that can be inherited:

- scanning intervals
- credentials
- notification templates
- schedules
- dependency settings
- security options

| Question | Answer |
| -------- | ------ |
| **Why does it matter in production?** | In deployments with hundreds or thousands of devices, it removes repetitive work and keeps configuration consistent. |
| **What if I skipped it?** | I would be duplicating settings on every device, and one changed password would mean editing every device by hand. |

---

## Verification

| Check | Expected result | Actual result | Status |
| ----- | --------------- | ------------- | ------ |
| Top-level groups exist | Network Infrastructure, Servers, Storage, Firewalls | Created | ✅ |
| Subgroup exists | Switching under Network Infrastructure | Created | ✅ |
| Auto-discovery | Disabled for the new groups | Left disabled | ✅ |

---

## What I Took Away

Before this, I thought groups were just folders to keep the interface tidy. They are actually the foundation of the platform: a good structure improves navigation, saves configuration time through inheritance, makes expansion easy, and makes troubleshooting faster during an outage.
