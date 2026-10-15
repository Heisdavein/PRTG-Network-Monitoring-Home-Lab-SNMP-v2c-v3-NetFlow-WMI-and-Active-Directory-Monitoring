# 1. Downloading and Installing PRTG

## Overview

PRTG turned out to be one of the simplest parts of this lab to install, even though it is an enterprise-grade platform. I installed it directly on my **Windows Server 2022 VM** in VMware Workstation. Keeping the whole monitoring platform inside one VM made future management, backups and updates easier.

---
## Step 1: Download the Installer

1. I logged into my Windows Server VM and opened a web browser.
2. I went to the official Paessler website and selected **Products > PRTG Network Monitor**.
3. On the download page I clicked **Free Download**. This gives a fully featured **30-day trial licence**. When the trial ends, the installation reverts to the free edition (up to 100 sensors). No payment details are needed.
4. While the installer downloaded, the website showed a unique **licence key** tied to my Paessler account. I copied it somewhere safe in case the installer asked for manual activation.

| Detail | What I observed |
| ------ | --------------- |
| Installer size | About 400 MB |
| Download time | Roughly 3 to 8 minutes, depending on network speed |

> [!TIP]
> I download software **inside the VM** whenever I can. That avoids copying large files from the host into the VM.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9381f307-4a5f-431e-843e-5ae9ee931c19" />
The Paessler PRTG download page with the "Free Download" button and the licence key shown (blur or redact the key before publishing.

---
## Step 2: Run the Installer

1. I double-clicked the PRTG Network Monitor installer.
2. I chose **English** as the language and clicked **OK**.
3. On the welcome screen I read and accepted the licence agreement, then clicked **Next**.
4. Because I downloaded while signed in to my account, the installer detected my licence automatically. (If this does not happen, the key from the download page can be entered manually.)
5. The installer asked for an **email address**. This becomes PRTG's primary notification account for alerts, reports and system messages. I entered mine.
6. PRTG contacted Paessler's licensing servers to validate the licence. This took about 15 to 30 seconds.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/94e4104b-1283-4502-b593-98fc062a99b7" />
The PRTG Setup Wizard pages for language selection, licence agreement, licence information, email address and licence validation.

---
## Step 3: Choose the Installation Type

PRTG offered **Express** and **Custom** installation.

| Feature | Express Installation | Custom Installation |
| ------- | -------------------- | ------------------- |
| Installation location | Default directory | Custom installation path |
| Device discovery | Scans the network automatically after install | Configured manually later |
| Configuration | Minimal input needed | More control over settings |
| Best suited for | Home labs, demos, small environments | Production environments with specific requirements |

I chose **Express Installation**. I wanted PRTG to start discovering devices straight away so I could confirm the monitoring server was talking to the rest of my lab. The installer then copied the application files, installed the Windows services, configured the built-in web server and prepared the monitoring database. This took about **3 to 10 minutes**, and PRTG created a desktop shortcut when it finished.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/f133c0a2-1447-4398-83ef-337a74087da9" />
The "Select Installation Type" page with Express Installation selected, followed by the installing progress page and the "Completing the PRTG Network Monitor Setup Wizard" page.

---
## Step 4: Verify the PRTG Services

Before opening the web interface, I checked that PRTG's services had started. I pressed **Windows + R**, typed `services.msc` and pressed Enter.

```text
services.msc
```

| Service | What it does | Startup type | Status |
| ------- | ------------ | ------------ | ------ |
| **PRTG Core Server Service** | The main service. Runs the web interface, stores monitoring data, processes alerts, generates reports, schedules sensor scans and coordinates all probes | Automatic | Running |
| **PRTG Probe Service** | Does the actual monitoring. Polls devices, collects sensor data, runs protocols such as SNMP and WMI, and sends results to the Core Server | Automatic | Running |

> [!NOTE]
> If either service is stopped after installation, right-click it and select **Start**.

> [!IMPORTANT]
> I avoid restarting PRTG services unless I really have to. Restarting the Core Server interrupts sensor polling and can leave gaps in monitoring data, graphs and alert processing. In production I would do it in a planned maintenance window.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/1203f1b2-feeb-4cad-8359-76037fb12f92" />
The Windows Services console with PRTG Core Server Service and PRTG Probe Service both showing Running and Automatic.

---
## Step 5: Log In for the First Time

I double-clicked the PRTG desktop shortcut, which opened the web console in my browser. The default credentials were:

| Field | Value |
| ----- | ----- |
| Username | `prtgadmin` |
| Password | `prtgadmin` |

The dashboard loaded within moments. Because I chose Express Installation, PRTG had already begun discovering devices such as my default gateway, DNS servers and several VMs. That confirmed the Core Server and Probe were working and that PRTG could see my network.

> [!WARNING]
> These default credentials are the same on every new PRTG installation. I changed them immediately. See [document 2](02-initial-setup-and-security-hardening.md).

 <img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/44281f22-0cb3-4fc6-b9e8-81a763e6ef09" />
  The PRTG login page, then the first dashboard view with auto-discovered devices appearing.

---

## Verification

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| Installer completes | PRTG installed, desktop shortcut created | Installed; shortcut created | ✅ |
| PRTG Core Server Service | Running, Automatic | Running, Automatic | ✅ |
| PRTG Probe Service | Running, Automatic | Running, Automatic | ✅ |
| Web interface login | Dashboard loads | Dashboard loaded | ✅ |
| Auto-discovery | Devices start appearing | Gateway, DNS servers and VMs discovered | ✅ |

---

## Why These Steps Matter

| Question | Answer |
| -------- | ------ |
| **Why check the services first?** | If the Core or Probe service is not running, nothing else in PRTG will work, and a missing dashboard is easier to diagnose from `services.msc`. |
| **Why Express?** | It gave me instant feedback that monitoring was working. A production deployment with specific requirements would use Custom. |
| **What if I skipped verification?** | I could have spent time troubleshooting devices when the real problem was a stopped service. |
