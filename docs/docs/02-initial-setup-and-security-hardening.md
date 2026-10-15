# 2. Initial Setup and Security Hardening

## Overview

When I first logged in, I resisted the urge to start adding devices. The first few minutes after installing any management platform should be spent **securing it**, not using it.

Two warning banners appeared straight away:

1. I was still using the **default administrator password**.
2. I was using **HTTP instead of HTTPS**.

A monitoring platform often holds credentials, device information and network topology that would be very valuable to an attacker. Before trusting PRTG to monitor my infrastructure, I wanted to trust PRTG itself.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The PRTG dashboard with the two warning banners ("Default administrator password is still active" and "You are using HTTP").

---

## Step 1: Cleaning Up the Auto-Discovered Devices

Express Installation made PRTG scan my network immediately, and within minutes it had found my gateway, DNS server, Linux machine and others. I deliberately removed them. I wanted to add each device manually later so I would understand exactly how credentials are stored, how sensors are created, which protocol each device uses, and how PRTG talks to different systems.

I went to **Devices**, right-clicked each device, selected **Delete** and confirmed.

| Removed | Kept (built-in monitoring objects) |
| ------- | ---------------------------------- |
| DNS Server | Local Probe |
| Gateway / Firewall | Core Server |
| Linux Machine | Internet Connectivity Sensor |
| | DNS Health Sensor |

I kept the built-in objects because they continuously watch the health of the monitoring server itself, and deleting them would only mean recreating them.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The Devices tree after cleanup, with Local Probe, Core Server, Internet Connectivity and DNS Health kept and the auto-discovered devices removed.

---

## Step 2: Replacing the Default Administrator Password

Leaving `prtgadmin` / `prtgadmin` in place would be like leaving the front door unlocked, because anyone who reached the web interface would already know the credentials.

1. I clicked my username in the top-right corner and opened **Account Settings**.
2. The page showed **Login Name**, **Display Name**, **Primary Email Address** and **Password**.
3. After entering the current password, I created a new one using uppercase letters, lowercase letters, numbers and special characters, and made it **longer than fourteen characters**.
4. I clicked **Save**, and the warning banner disappeared.

| Question | Answer |
| -------- | ------ |
| **Why?** | Default credentials are public knowledge. |
| **Production difference** | Real organisations use **named administrator accounts** instead of shared ones. Each engineer gets their own account, which gives audit logs of who changed what, MFA support, easier permission management and simple account removal when staff leave. A single admin account is fine for my lab, but would be unacceptable in a real organisation. |
| **If skipped** | Anyone on the network could log in as administrator. |

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The Account Settings page with the password change fields and the strength indicator (do not show the real password).

---

## Step 3: Switching from HTTP to HTTPS

Even with a new password, I was still using plain HTTP, which sends everything unencrypted. Usernames, passwords, cookies, session tokens and monitoring data could be read by anyone able to watch the traffic. This matters because PRTG holds credentials for switches, firewalls, Windows servers and storage.

1. I clicked the HTTPS warning banner, which opened the SSL configuration page.
2. PRTG offered to generate a **self-signed certificate**. I accepted, because my lab is isolated.
3. PRTG restarted its internal web service.
4. My browser showed a **certificate warning**. This is expected: the certificate was created locally, not by a trusted Certificate Authority.
5. I selected **Advanced**, then **Proceed**, and logged in again over HTTPS.

| Environment | Certificate |
| ----------- | ----------- |
| Home lab (mine) | Self-signed certificate is acceptable |
| Production | A certificate from a trusted CA such as DigiCert, GlobalSign, Let's Encrypt, or the organisation's internal Active Directory Certificate Services. This removes browser warnings and keeps admin traffic encrypted |

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The Web Server Settings page with HTTPS enabled and "Generate a self-signed certificate" selected, then the browser certificate warning with Advanced > Proceed.

---

## Step 4: Enabling Browser Notifications

My browser (not PRTG) asked if I wanted to allow notifications, and I chose **Allow**. Email is still the main alert method, but browser notifications add another layer: alerts can pop up on my desktop even if I am in another tab. In production this would sit alongside email, SMS, Microsoft Teams, Slack or mobile push.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The browser permission prompt asking to show notifications, with Allow selected.

---

## Step 5: Reviewing Notification Templates

Monitoring without notifications has very little value. I opened **Setup → Notifications**. PRTG had already created a default email template using the address I gave during installation, so I reviewed how it behaves instead of building a new one.

The default sends a notification when a monitored object:
- first goes **offline**, or
- comes back **online**.

I like this because it tells me both when a problem starts and when service returns.

**How I would design alerting in production:**

| Device type | Notification approach |
| ----------- | --------------------- |
| Critical (domain controllers, firewalls, internet gateways, backup servers) | Several methods at once: email, SMS and push |
| Less important devices | Email only |
| Schedules | I left notifications on 24/7 for my lab. In a business, planned maintenance windows normally suppress non-critical alerts while engineers work |

> [!IMPORTANT]
> **Alert fatigue** is a real danger. If every small fluctuation produces a warning, administrators stop reading alerts and the one that matters gets missed. I would rather receive ten meaningful alerts than one thousand unnecessary ones. Effective monitoring is about quality, not quantity.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: Setup > Notifications with the default email notification template open, showing the "goes Down" and "comes Up" triggers.

---

## Step 6: Learning My Way Around the Interface

Before adding devices I explored the top menu:

| Section | Purpose |
| ------- | ------- |
| **Devices** | Monitored infrastructure organised into groups, devices and sensors |
| **Sensors** | A complete list of every measurement being collected |
| **Alarms** | Active warnings and failures needing attention |
| **Maps** | Custom dashboards and graphical network maps |
| **Reports** | Historical reports and availability statistics |
| **Logs** | Configuration changes, events and system activity |
| **Tickets** | Built-in incident tracking and integrations with service desks |
| **Setup** | Administrative configuration for the whole platform |

---

## Step 7: Reviewing Automatic Updates

I visited **Setup → Auto-Update**. PRTG can update itself automatically or manually.

| Environment | My choice | Reason |
| ----------- | --------- | ------ |
| Home lab | Automatic updates left on | Downtime only affects me |
| Production | Manual updates | Test in a lab or staging environment first. If an update breaks the monitoring platform, administrators lose visibility into the network exactly when they might need it most |

---

## Step 8: Creating My First Configuration Backup

Before monitoring any device, I took my first backup:

1. I opened **Setup → Administrative Tools**.
2. I selected **Save Configuration to File**.
3. PRTG produced a **ZIP archive** containing its configuration database.

It took moments, and it gave me a recovery point in case I misconfigured something later. I made it a habit to take another before any significant change.

| Backup type | What it protects |
| ----------- | ---------------- |
| **PRTG configuration backup** | Device settings, sensor configuration, notification templates, historical monitoring data |
| **Virtual machine backup** | The Windows operating system, installed applications, system files, registry, and recovery from hardware failure or ransomware |

Both are needed and neither replaces the other. True disaster recovery protects both the application and the platform it runs on.

> **📸 Screenshot Placeholder**
> Insert screenshot showing: Setup > Administrative Tools > Save Configuration to File, with the "Include history data" option and the resulting ZIP file saved.

---

## Verification

| Test | Expected result | Actual result | Status |
| ---- | --------------- | ------------- | ------ |
| Default password warning | Banner disappears after password change | Banner disappeared | ✅ |
| HTTPS | Interface reachable over HTTPS | Connected over HTTPS after accepting the self-signed certificate warning | ✅ |
| Notification template | Default email template present, with Down and Up triggers | Present as described | ✅ |
| Configuration backup | ZIP archive created | ZIP archive created | ✅ |

---

## Visual Summary

> **📸 Screenshot Placeholder**
> Insert screenshot showing: The "Section 2: Initial Setup and Security Hardening" summary graphic from my original documentation (page 10 of the PDF).

---

## What I Took Away

Spending time securing PRTG before adding devices was one of the best decisions I made. A monitoring platform holds sensitive information about the whole network, and securing it properly means I can trust it to protect and monitor everything else.
