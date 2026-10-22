# 8. Monitoring Windows Servers with WMI and SNMP

## Overview

Windows servers can expose performance data in more than one way. The two most common are:

- **WMI** (Windows Management Instrumentation): Microsoft's native management framework. It gives deep visibility into the operating system itself: Windows services, Active Directory, Event Logs, Windows Update status, memory, disk performance, page file usage and more.
- **SNMP**: platform independent. It gives standard statistics (uptime, CPU, interface traffic, memory) and lets a Windows server be monitored like any router, switch or storage device.

I decided to configure **both**. WMI gives rich operating system monitoring, and SNMP gives a vendor-neutral backup source. If one method has problems, the other keeps collecting data.

---

## WMI vs SNMP

| Feature | WMI | SNMP |
| ------- | --- | ---- |
| Authentication | Windows or Active Directory credentials | Community string (v2c) or username/password (v3) |
| Installation | Built into Windows | Requires the SNMP feature to be installed |
| Level of detail | Extremely detailed | General system statistics |
| Windows services | Yes | No |
| Event logs | Yes | No |
| Windows updates | Yes | No |
| CPU and memory | Yes | Yes |
| Network traffic | Yes | Yes |
| Best use | Windows servers | Network devices and baseline monitoring |

**Rule of thumb:** use WMI for Windows-specific functionality, and SNMP for standard hardware metrics or devices that do not support WMI. In production most administrators use WMI as the primary method and SNMP as a supplement.

---

## Step 1: Add the Windows Server to PRTG

I opened **Devices**, navigated to the **Servers** group, right-clicked and chose **Add Device**.

| Setting | Value |
| ------- | ----- |
| Device name | `DMT-AD-01` |
| IPv4 address | `192.168.1.222` (the server's static IP) |
| Device icon | Windows Server |

**Naming convention:** `DMT` (organisation) – `AD` (role: Active Directory) – `01` (first server of this type).

> [!TIP]
> Servers should always use **static IP addresses**, because monitoring relies on predictable connectivity.

### Windows credentials

In **Credentials for Windows Systems** I disabled inheritance, because I wanted to specify exactly which account PRTG uses:

| Setting | Value |
| ------- | ----- |
| Domain | `corp.danieltraining.com` |
| Username | `administrator` |
| Password | (entered, then masked) |

| Question | Answer |
| -------- | ------ |
| **Why does PRTG need Windows credentials?** | WMI is authenticated remote management. When PRTG asks for CPU use or Windows Update status, it is asking Windows itself, and Windows needs an account with enough permissions. |
| **Production difference** | I used the built-in Administrator for my lab, which is not best practice. In production I would create a dedicated service account such as `svc_prtg` with only the permissions needed for monitoring (the **Principle of Least Privilege**), reducing the impact if the account is compromised. |

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/971d0b47-3492-4e7c-9a2a-7d5a34e0104b" />
The Add Device page for DMT-AD-01 in the Servers group, with the IP address, Windows Server icon and Credentials for Windows Systems (password hidden).

---

## Step 2: Add a WMI CPU Sensor

1. I selected **DMT-AD-01** and clicked **Add Sensor**.
2. I filtered by **Target System: Windows** and **Technology: WMI**.
3. I selected **WMI CPU Load** and clicked **Create**.

Within about one polling interval (60 seconds by default) the sensor showed processor use. Internally, PRTG queries the Windows `Win32_Processor` WMI class. The sensor shows the current CPU percentage, history graphs, minimum and maximum values, average use and sensor availability. If users say the server was slow yesterday afternoon, I can look at that exact period instead of guessing.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/166a3581-6d56-4c7a-919a-e5f62edcfadb" />
 The WMI CPU Load sensor on DMT-AD-01 with the current value, history graph and channel list.

---

## Step 3: Use Recommended Sensors

Instead of searching hundreds of sensor types, I used **Add Sensor → Recommended Sensors**. PRTG scanned the server and suggested these for my AD server:

- Ping
- WMI Memory
- WMI Disk Free
- WMI Page File
- WMI Uptime
- WMI Network Adapter
- WMI Physical Disk
- DNS (WMI)
- Windows Update Status

I ticked each one and clicked **Add Selected**. Within minutes the server had a complete health profile.

### Why each sensor matters

| Sensor | Why it matters | Thresholds I used |
| ------ | -------------- | ----------------- |
| **Ping** | The most basic health check. If it fails, every other monitoring protocol will almost certainly fail too | n/a |
| **WMI Memory** | Memory shortages cause performance problems long before CPU is overloaded. As RAM fills, Windows moves memory pages to disk (paging), which is far slower | Warning 85%, Critical 95% |
| **WMI Disk Free** | Windows needs free space for updates, temporary files, page file growth, event logs and Active Directory database operations. A full system drive can make the server unstable | Warning 20% free, Critical 10% free |
| **WMI Page File** | Shows whether Windows relies heavily on virtual memory. High page file use plus high RAM use usually means more physical memory is needed | n/a |
| **WMI Physical Disk** | Measures storage *performance* (read latency, write latency, queue length, disk utilisation). High values can mean overloaded storage, failing disks or excessive application activity | n/a |
| **WMI Network Adapter** | Throughput, errors and discarded packets, to spot congestion or a failing adapter | n/a |
| **DNS (WMI)** | The server hosts DNS. If DNS fails, users cannot resolve names, AD authentication starts failing and many applications stop working. A healthy server with failed DNS is still a major outage | n/a |
| **Windows Update Status** | One of the most valuable sensors for security. Missing updates often mean missing security patches. PRTG alerts me when update compliance drops below my standard | n/a |

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/acca819e-4926-47fb-8e28-f5c40838582d" />
The Recommended Sensors list for DMT-AD-01 with the sensors ticked.

---

## Step 4: Install the SNMP Service on the Windows Server

Windows does not install SNMP by default.

1. On the server I opened **Server Manager**, then **Manage**, then **Add Roles and Features**.
2. I navigated to **Features** and ticked **SNMP Service**.
3. I clicked **Install**.
4. When it finished, I opened `services.msc`, located **SNMP Service** and opened its **Properties**.

```text
services.msc
```

### Agent tab (optional documentation)

| Field | Value |
| ----- | ----- |
| Contact | `lab-admin@corp.local` |
| Location | `Home Lab – Primary Server` |

These do not affect monitoring, but give useful inventory information.

### Traps tab

| Field | Value |
| ----- | ----- |
| Community | `danmilltraining` |
| Trap destination | `192.168.1.100` (the PRTG server) |

### Security tab

| Setting | Value |
| ------- | ----- |
| Accepted community name | `danmilltraining` |
| Rights | READ ONLY |
| Accept SNMP packets from these hosts | `192.168.1.100` only |

Read-only rights mean PRTG cannot change server configuration through SNMP, and accepting packets from the PRTG server only means no other device can query the server over SNMP.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/1c830104-3966-4598-88a0-44466d26d062" />
The SNMP Service Properties window with the Agent, Traps and Security tabs configured (community string and IPs visible as documented).

---

## Step 5: Configure SNMP Credentials in PRTG

I opened **DMT-AD-01 → Edit**, went to **Credentials for SNMP Devices**, disabled inheritance and set:

| Setting | Value |
| ------- | ----- |
| Version | SNMP v2c |
| Community | `danmilltraining` |

---

## Step 6: Add an SNMP System Uptime Sensor

I added **SNMP System Uptime** to report uptime independently of WMI. It is a second source of truth: if the server restarts overnight, uptime resets and can trigger an alert. Many administrators alert when uptime falls below **two hours** outside maintenance windows, which catches unplanned reboots almost immediately.

---

## Step 7: Summary of the Most Important Windows Sensors

| Sensor | Why it matters |
| ------ | -------------- |
| Ping | Confirms the server is reachable. Usually the first sign of an outage |
| WMI CPU Load | Sustained use above 80 to 85% suggests the processor may be overloaded |
| WMI Memory | Detects RAM exhaustion before applications slow down |
| WMI Disk Free | Prevents instability from a full system drive |
| WMI Physical Disk | Reveals storage bottlenecks through latency and queue length |
| WMI Network Adapter | Finds packet loss, interface errors and abnormal utilisation |
| WMI Page File | Highlights excessive paging (insufficient physical memory) |
| WMI Windows Update | Keeps security updates current |
| WMI Uptime | Detects unexpected reboots |
| SNMP System Uptime | Independent uptime check, giving redundancy if WMI is unavailable |

---

## Verification

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| WMI CPU Load sensor | Shows CPU data within about 60 seconds | Displayed processor use within one polling interval | ✅ |
| Recommended sensors | Added and building a health profile | Added; profile built within minutes | ✅ |
| SNMP service configured | Read-only, limited to PRTG server IP | Configured as documented | ✅ |
| SNMP System Uptime | Reports uptime separately from WMI | Reported from the next polling cycle | ✅ |

---

## What I Took Away

Good server monitoring goes far beyond checking whether a machine answers a ping. A server can respond while suffering from high CPU, memory exhaustion, failing disks, stalled services or missing security updates. Combining WMI's depth with SNMP's standard hardware view gave me a more complete and resilient solution, and I reinforced three habits: use dedicated service accounts, restrict SNMP to trusted hosts, and set meaningful alert thresholds.

---
