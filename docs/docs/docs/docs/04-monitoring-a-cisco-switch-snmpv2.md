# 4. Monitoring a Cisco Switch with SNMP Version 2
---

## Overview

Adding my Cisco Catalyst switch was the first time I connected a real network device to PRTG. I chose a switch because it is one of the most common devices in enterprise networks. If a switch fails, the impact can range from one disconnected workstation to a whole building losing connectivity.

My goal was for PRTG to poll the switch every **sixty seconds** and collect availability, uptime, CPU, memory, hardware health and interface traffic, and to alert me if the switch became unreachable or a hardware component began to fail.

---

## Understanding SNMP

**SNMP** (Simple Network Management Protocol) is one of the most important protocols in network administration. Routers, switches, firewalls, wireless controllers, printers, storage arrays, UPS systems, environmental sensors and many IoT devices support it.

It uses a **manager-agent model**:

- **PRTG** is the SNMP **Manager**.
- Every monitored device runs an SNMP **Agent**.
- At regular intervals, the manager asks the agent questions such as: What is your CPU usage? How much memory are you using? How long have you been running? Which interfaces are active? Are your fans normal? Has a power supply failed?

PRTG stores the answers, draws graphs, compares values against thresholds and raises alerts. I think of SNMP as a routine health check: PRTG does regular check-ups so I can spot warning signs before they become outages.

### Key SNMP concepts

| Concept | What it is |
| ------- | ---------- |
| **OID** (Object Identifier) | A unique numerical address for one value inside a device, like a file path that points to a piece of information. For example, `1.3.6.1.2.1.1.3.0` is system uptime on almost every SNMP device. |
| **MIB** (Management Information Base) | A dictionary that translates OIDs into readable names (for example, that OID means "System Uptime"). Cisco, Synology, WatchGuard, Dell, HP and others publish MIBs. |
| **Community string** | SNMP v2 has no usernames or passwords. A community string works like a shared password: the device answers only if it matches. I used **Read-Only (RO)** access so PRTG can read but never change configuration. |
| **Polling** | PRTG asks the device for data every sixty seconds. |
| **Traps** | The device sends a message *immediately* when something important happens (interface failure, power supply fault, fan failure, excessive temperature, reboot). Polling plus traps is more responsive than either alone. |

---

## Step 1: Configure SNMP on the Cisco Switch

I connected to my **Cisco Catalyst 2960X** by SSH, entered privileged EXEC mode, then global configuration mode:

```text
Switch> enable
Switch# configure terminal

! Configure a read-only SNMP community
Switch(config)# snmp-server community danmilltraining RO

! Enable SNMP trap generation
Switch(config)# snmp-server enable traps

! Optional but useful administrative information
Switch(config)# snmp-server contact lab-admin@corp.local
Switch(config)# snmp-server location Home Lab - Switch Rack

Switch(config)# end

Switch# copy running-config startup-config
```

Very little configuration was needed. The switch could now answer SNMP queries, and PRTG would use the same community string to authenticate every poll.

| Question | Answer |
| -------- | ------ |
| **Why `RO`?** | Monitoring rarely needs write access, so read-only is best practice. |
| **Why `copy running-config startup-config`?** | It saves the configuration so SNMP survives a reboot. |
| **If skipped** | The switch would revert to its old configuration after a reboot and monitoring would silently stop. |

> [!WARNING]
> Many devices ship with default community strings such as `public` and `private`. These are widely known and must never be used in production, because anyone who knows the string can query the device. In production I would also add an **SNMP access list** that only allows requests from the PRTG server's IP address, so even a discovered community string is useless from any other system.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/f9d10fc1-a4f8-408e-a5af-ea5da76641a2" />
The SSH session to the Cisco switch with the SNMP commands entered and `copy running-config startup-config` completed (redact the community string if desired).

---
## Step 2: Add the Switch to PRTG

I navigated to **Devices → Network Infrastructure → Switching**, right-clicked and selected **Add Device**.

| Setting | Value |
| ------- | ----- |
| Device name | `DMT-SW-01` |
| IP version | IPv4 |
| IP address | The switch's management IP |
| Icon | Cisco |
| SNMP inheritance | Disabled (to configure this device individually for learning) |
| SNMP version | SNMP v2c |
| Community string | `danmilltraining` |
| UDP port | 161 |
| Timeout | 5 seconds |
| Auto-discovery | Disabled temporarily (I ran it manually once the device was fully configured) |

**My naming convention:**

| Part | Meaning |
| ---- | ------- |
| `DMT` | Organisation abbreviation |
| `SW` | Device type (Switch) |
| `01` | Device number |

A consistent naming convention makes devices much easier to find as the environment grows.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/3e1bc61c-2bc5-4596-8cb8-037f3d5601f4" />
The "Add Device" page for DMT-SW-01 with the IP address, Cisco icon and SNMP settings (v2c, community, UDP 161, timeout 5 seconds).

---

## Step 3: Run Auto-Discovery

I right-clicked the switch and selected **Run Auto-Discovery**. PRTG queried the switch via SNMP, worked out which metrics were available, and after about one to two minutes created recommended sensors:

| Sensor | What it monitors |
| ------ | ---------------- |
| Ping | Whether the device is reachable |
| SNMP System Uptime | How long the switch has run since its last reboot |
| SNMP CPU Load | Processor use over time |
| SNMP Memory | Memory use over time |
| Fan Health | Hardware (fan) status |
| Interface Traffic | Traffic on every active port, one sensor per interface |
| SSL Certificate | Certificate status |

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ebb12798-e3e3-4ce0-ae40-eb190ee74a4b" />
The Auto-Discovery results for DMT-SW-01 listing Ping, System Uptime, CPU Load, Memory, Fan Health, Interface Traffic and SSL Certificate sensors.

---

## Step 4: Explore the Monitoring Data

I opened the **SNMP CPU Load** sensor. It showed the current CPU percentage, a 24-hour history graph, and a larger interactive graph under **Live Data** that I could zoom into to see when CPU rose and how long spikes lasted.

If someone reports that "the network was slow yesterday afternoon", I can look at CPU, interface traffic, memory and uptime graphs for that exact period instead of guessing. This is objective evidence rather than opinion.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/38620597-e38b-4d05-90a9-51546e8e6d51" />
 The SNMP CPU Load sensor page with the current value, the 24-hour graph and the Live Data tab.

---

## Step 5: Test the Monitoring System

A monitoring system is only useful if it actually detects failures, so I ran a controlled test:

1. I disconnected the network cable between the switch and the firewall.
2. I waited about one polling cycle.
3. I reconnected the cable and waited again.

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| Cable disconnected | Device turns red, Ping reports packet loss, SNMP sensors fail | Device icon turned red; Ping reported complete packet loss; every SNMP sensor changed state | ✅ |
| Failure alert | Email notification of outage | Email arrived in my inbox | ✅ |
| Cable reconnected | Sensors return to green | All sensors gradually returned to green | ✅ |
| Recovery alert | Second email on recovery | Second email received | ✅ |

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9f33e855-bbd7-4776-beb0-335b60a7fc66" />
The PRTG device tree with DMT-SW-01 red/Down during the cable test (Ping and SNMP sensors Down).

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/d4420265-355a-491e-b15b-033ad49e6fe6" />
The failure email and the recovery email in my inbox.

Without PRTG I would only find out about this outage when a user reported losing connectivity. With PRTG the platform found the problem first, recorded it, alerted me and confirmed recovery. That shift from reacting after users notice to finding issues before they do is a major reason organisations invest in monitoring.

---

## What I Took Away

- Configuring SNMP on a switch needs surprisingly little effort but unlocks a lot of visibility.
- **Verification matters.** Simulating a failure proved my monitoring *and* alerting worked. From then on I was not only monitoring my network, I was verifying that my monitoring system could be trusted.
