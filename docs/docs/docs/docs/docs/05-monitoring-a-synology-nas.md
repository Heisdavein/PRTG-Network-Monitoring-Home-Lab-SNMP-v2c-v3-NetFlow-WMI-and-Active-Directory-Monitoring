# 5. Monitoring a Synology NAS with Manual Sensors

## Overview

For the Synology NAS I deliberately **skipped auto-discovery** and added each sensor by hand. Auto-discovery is useful and saves time, but I wanted to understand exactly what I was monitoring and why each sensor mattered. It was one of the most educational parts of the lab, and it reinforced that good monitoring is about collecting the **right** data, not the most data.

## Why a NAS Should Be Monitored

A NAS often stores shared company files, documents, VM backups, surveillance footage or user home folders. If a switch fails, users lose connectivity for a while. If a router fails, they lose internet access. If a storage device fails without warning, an organisation can lose **years of data**.

Questions I wanted PRTG to answer:

- Are the hard drives healthy?
- Are any disks overheating?
- Is the RAID array healthy?
- Is memory usage becoming excessive?
- Is the NAS reachable?
- Is network traffic unusually high?
- Is storage capacity approaching its limit?

Those questions decided which sensors I built.

---

## Step 1: Enable SNMP on the Synology NAS

Unlike the Cisco switch (command line), Synology DSM has a graphical interface.

1. I opened a browser, entered the NAS IP address and logged into **Synology DiskStation Manager (DSM)** as administrator.
2. I went to **Control Panel → Terminal & SNMP → SNMP**.
3. On the SNMP tab:
   1. Ticked **Enable SNMP Service**.
   2. Selected **SNMP Version: SNMPv2c**.
   3. Entered the community string `danmilltraining`.
   4. Clicked **Apply**.

> [!IMPORTANT]
> The community string must exactly match what PRTG uses. A single wrong character makes every SNMP request fail.

Synology makes SNMP much simpler than Cisco IOS, but the underlying protocol is exactly the same.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/b1628b18-0eb0-44a1-93bb-1ac36da74d3b" />
Synology DSM Control Panel > Terminal & SNMP > SNMP with "Enable SNMP service" ticked, SNMPv2c selected and the community string field (redact if desired).

---

## Step 2: Create the Storage Group and Add the Device

1. In PRTG I right-clicked **Local Probe** and selected **Add Group**.
2. I named it `Storage`, disabled Auto Discovery and clicked **OK**.
3. Inside it I added my NAS:

| Setting | Value |
| ------- | ----- |
| Device name | `DMT-NAS-01` |
| IP version | IPv4 |
| IP address | NAS management IP |
| Device icon | Generic Storage (there is no Synology-specific icon) |
| Inherit from Parent | Disabled |
| SNMP version | SNMP v2c |
| Community string | `danmilltraining` |

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/27b66cf0-91f3-4c57-9da2-456e528b3123" />
The Add Device page for DMT-NAS-01 in the Storage group with SNMP v2c settings.

---

## Step 3: Add Sensors Manually

Instead of Auto-Discovery, I clicked **Add Sensor** and searched for "Synology", which returned several Synology-specific sensors.

| Sensor | What it checks | Why I chose it |
| ------ | -------------- | -------------- |
| **Synology System Health** | Overall system status, fan health, power supply, system temperature, hard drive condition | An at-a-glance health overview. Turns from green to warning or error if any component reports a problem |
| **Synology Physical Disk** | For each of my **two disks**: operational status, temperature, health, read/write condition | Drives rarely fail instantly. They show warning signs first: rising temperature, changing SMART values, more read errors. Catching these lets an engineer replace a drive before users notice |
| **Synology Logical Disk** | RAID health, available capacity, used capacity, logical volume status | Users use logical volumes, not physical disks. If the RAID degrades because of a failed disk, this sensor alerts me |


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ee4eaf7e-b21a-4764-a1f5-47e5b8c3e918" />
The Add Sensor page filtered by "synology" showing the Synology-specific sensor types.

---

## Step 4: Add Generic SNMP Sensors

I also wanted resources every network device has.

| Sensor | Configuration | Why it matters |
| ------ | ------------- | -------------- |
| **SNMP Memory** | Physical Memory, Virtual Memory, Swap Space | Trends matter more than snapshots. Memory that creeps up every day without dropping can mean a leak, runaway processes, too little RAM or poorly performing applications |
| **Ping** | Default | Probably the most important sensor. If the NAS stops answering ICMP, it could be powered off, disconnected, suffering a network failure or crashed. Every other sensor depends on the device being reachable |
| **SNMP Traffic** | The NAS network interface | Inbound and outbound bandwidth, utilisation and traffic history. A backup job suddenly moving hundreds of gigabytes during business hours would show up at an exact time |

---

## Step 5: Verify the Sensors

After a few polling cycles every sensor turned green, which told me two things: PRTG could talk to the NAS, and the SNMP configuration was correct. I opened the **Physical Disk** sensor and saw disk status, current temperature, a temperature history graph and long-term health trends. If a drive slowly warmed from 34°C to 49°C over several weeks, I would spot it long before it failed.

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| Sensors reach green | PRTG communicates with NAS | All sensors turned green | ✅ |
| SNMP community match | No SNMP errors | No errors | ✅ |
| Physical Disk sensor | Status, temperature and trend graphs visible | Visible | ✅ |

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/fbec5c11-a33f-47a9-aa26-ab504a9a5b87" />
The DMT-NAS-01 device page with all sensors green (Ping, System Health, both Physical Disks, Logical Disk, Memory, Traffic).


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/b4865da9-675a-4735-8182-d49f1b423cba" />
The Synology Physical Disk sensor page with the temperature history graph.

---

## What I Took Away

I used to think monitoring storage meant checking whether it was online. A healthy NAS is one whose disks stay cool, whose RAID is healthy, whose memory is stable and whose traffic follows expected patterns. The biggest lesson was the value of **historical data**: current status shows how a device is now, but graphs over days and weeks show whether it is getting healthier or drifting towards failure.
