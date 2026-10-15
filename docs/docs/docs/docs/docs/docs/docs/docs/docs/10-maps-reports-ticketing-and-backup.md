# 10. Maps, Reports, Ticketing and Backup


## Overview

By this point PRTG was collecting data from the switch, servers, firewall and NAS. But collecting data is not enough. A monitoring platform becomes genuinely valuable when it helps a team understand what is happening, communicate it, respond to incidents and recover if something goes wrong. This section covers four features that turned PRTG from a collection of sensors into a complete platform:

- **Network maps** for visual monitoring
- **Reports** for documentation and long-term planning
- **Ticketing** for incident management
- **Configuration backups** for disaster recovery

Network operations centres (NOCs), managed service providers (MSPs) and enterprise IT teams use these every day.

---

## Network Maps

As I added devices, the Devices tree became hard to picture. It does not answer: Which switch connects to which firewall? Which server is on which segment? Which device is actually failing?

A **PRTG map** is a live dashboard combining monitoring data with a diagram of the network. Green means healthy, yellow means warning and red means immediate attention. Colours update in real time, so I can spot a problem in seconds. Large organisations show these on wall-mounted screens in their NOC.

### Creating my first map

1. I clicked **Maps**, then **Add Map**.
2. I named it `Home Lab Network Topology`.
3. I opened the **Map Designer** (drag-and-drop) and dragged my monitored devices onto the canvas in a layout roughly following the physical network:

```text
Internet
   │
WatchGuard Firewall
   │
 ┌─┴──────────────┐
 │                │
Cisco Switch   Core Switch
 │                │
Windows Server  Synology NAS
```

Once a device is on the map its status is dynamic. If the firewall goes offline its icon turns red, and if CPU passes a threshold the icon changes colour automatically.

### Customising the dashboard

Maps can also include company logos, text labels, bandwidth graphs, sensor gauges, live traffic charts, status tables, uptime counters and custom HTML widgets. I can display key business information as well, for example: Firewall Online, VPN Connected, Domain Controller Healthy, DNS Available, and current WAN usage.

### How larger organisations use maps (not built in my lab)

| Map type | Examples |
| -------- | -------- |
| **Site maps** | One map per office, for example London, Sydney, Melbourne, Singapore |
| **Department maps** | Network Infrastructure, Servers, Security, Cloud Services |
| **Executive dashboards** | Services available, active incidents, SLA compliance, internet availability, without low-level technical detail |

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The "Home Lab Network Topology" map in PRTG with the firewall, switches, Windows Server and Synology NAS showing live status colours.

---

## Reports

Real-time monitoring shows what is happening now. **Reports** show what happened yesterday, last month or last year, giving objective evidence instead of memory.

For example, if someone asks *"Has the internet connection really been unstable this month?"*, an availability report answers with data (availability percentage, downtime and average latency) rather than opinion.

> [!NOTE]
> The figures PRTG could show in such a report (for example 99.98% availability, 11 minutes of downtime and 7 ms average latency) are illustrative examples, not measurements from my lab.

### Creating a report

1. I opened **Reports**.
2. I selected **Add Report**.
3. I chose a template: Sensor Overview, Availability Report, Historic Data or SLA Report.
4. I selected the devices or sensors to include, for example the firewall, core switch, domain controller and NAS.
5. I chose a reporting period: Last 24 Hours, Last Week, Last Month, Last 90 Days or a Custom Range.
6. I chose a schedule: Daily, Weekly or Monthly.

PRTG can email the finished PDF or HTML report to administrators automatically.

### Practical uses

| Use | How reports help |
| --- | ---------------- |
| **Availability reporting** | Organisations that promise a level of uptime (for example a 99.9% SLA) can show evidence the agreement was met |
| **Capacity planning** | Months of trends show storage filling, WAN utilisation increasing or memory rising, so capacity can be predicted before a disk becomes full |
| **Security auditing** | Historical records of missing Windows updates, repeated authentication failures or firewall interface downtime support compliance reviews and audits |

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The PRTG "Add Report" page with a template, reporting period and schedule selected, or a generated availability report.

---

## Ticketing

Monitoring only becomes useful when someone acts on the information. Ticketing adds that workflow: instead of just showing a red sensor, PRTG records the issue, tracks progress and documents the fix.

### Built-in ticketing

PRTG has a lightweight ticketing system. When a sensor goes **Down**, it can create a ticket automatically recording the affected device, sensor name, alert time, priority and current status. Engineers can acknowledge the issue, add notes, assign ownership and close the ticket after resolution. For a home lab, this is more than enough.

### Enterprise integrations

Most enterprises already run IT Service Management platforms, so PRTG integrates with them rather than replacing them: **ServiceNow, Jira Service Management, ConnectWise and HaloPSA**. A fully automated workflow looks like this:

```text
Firewall interface down
        ↓
PRTG detects failure
        ↓
Ticket automatically created
        ↓
Assigned to Network Team
        ↓
Engineer investigates
        ↓
Problem resolved
        ↓
Ticket closed
```

No manual step is needed to notify the helpdesk.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The PRTG Tickets page with a ticket generated by a sensor going Down (device, sensor, priority and status visible).

---

## Configuration Backup Strategy

Monitoring software becomes mission-critical quickly. After configuring dozens of devices, rebuilding from scratch would take hours, so protecting the monitoring system matters as much as monitoring everything else. I use two complementary backup methods.

### 1. PRTG configuration backup

This protects the application's configuration: devices, groups, sensors, notification templates, credentials, user accounts and monitoring settings.

1. I opened **Setup**.
2. I selected **Administrative Tools**.
3. I clicked **Save Configuration to File**.

PRTG generates a ZIP archive of the current configuration. I store copies on both my NAS and cloud storage for redundancy.

### 2. Virtual machine backup

A configuration backup alone is not enough. If Windows becomes corrupted or the virtual disk fails, restoring only the configuration would still mean reinstalling the operating system and PRTG. So I also back up the complete VMware VM with **Veeam Backup & Replication**. A VM backup captures the Windows Server, the PRTG installation, historical monitoring data, application settings and operating system configuration. If disaster strikes I restore the VM and resume monitoring with minimal downtime.

### My backup schedule

| Backup | Frequency | Purpose |
| ------ | --------- | ------- |
| PRTG configuration | Daily and before major changes | Protects the monitoring configuration |
| VMware VM backup | Every night | Protects the complete monitoring server |
| NAS replication | Continuous | Protects backup storage |
| Restore testing | Quarterly | Verifies backups can actually be restored |

This layered approach means no single failure can wipe out my monitoring platform.

### Testing the backups

A backup that has never been restored is only an assumption. At least once every quarter I restore the PRTG configuration to a separate test VM and check that:

- all devices reappear correctly,
- sensors resume polling,
- notification templates are intact,
- historical data is accessible where applicable,
- dashboards and maps load without errors.

Only after a successful restore test can I be confident the backup strategy will work in a real disaster.

> [!TIP]
> My Veeam backup and restore work is documented in a separate project: [Veeam Backup & Replication Home Lab](https://github.com/YOUR-USERNAME/veeam-backup-dr-homelab). Replace the link with your real repository URL.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The PRTG Save Configuration to File page and the resulting ZIP file, plus the Veeam job protecting the PRTG VM.

---

## Verification

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| Map created | Live map with devices and status colours | `Home Lab Network Topology` created | ✅ |
| Report configured | Template, period and schedule selectable | Configured as described | ✅ |
| Configuration backup | ZIP archive generated | ZIP archive generated | ✅ |
| Quarterly restore test | Configuration restored to a test VM with devices, sensors, templates and maps intact | Documented as my routine | ✅ |

---

## Visual Summary

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The "Section 10: Maps, Reports, Ticketing, and Backup" summary graphic from my original documentation (page 51 of the PDF).

---

## What I Took Away

Effective monitoring goes beyond collecting metrics. Maps give an intuitive view that speeds up fault finding, reports provide historical evidence for troubleshooting and capacity planning, ticketing turns alerts into tracked incidents, and layered backups protect the platform. The most important lesson: **a monitoring server is itself a critical service.** If it fails during a network outage, I lose visibility into every other system exactly when I need it most.

---
