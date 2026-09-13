# 🛡️ Active Directory Security Audit

> **Target Domain:** `proxym.tn`
> **Audit Date:** 27 August 2026
> **Auditor:** Aya NASR
> **Supervisor:** Houda Mabrouk

---

## 📋 Table of Contents

* [Project Information](#-project-information)
* [Network Architecture](#-network-architecture)
* [Active Directory Structure](#-active-directory-structure)
* [Infrastructure](#-infrastructure)
* [Part 1 — Group Policy Configuration](#-part-1--group-policy-configuration)
* [Part 2 — BloodHound Analysis](#-part-2--bloodhound-analysis)
* [Session Analysis](#-session-analysis)
* [ACL Analysis](#-acl-analysis)
* [Security Findings](#-security-findings)
* [Identified Vulnerabilities](#-identified-vulnerabilities)
* [Remediation Plan](#-remediation-plan)
* [Conclusion](#-conclusion)

---

# 👤 Project Information

| Field              | Details                         |
| ------------------ | ------------------------------- |
| **Intern**         | Aya NASR                        |
| **Supervisor**     | Houda Mabrouk                   |
| **Project**        | Active Directory Security Audit |
| **Target Domain**  | `proxym.tn`                     |
| **Audit Date**     | 27 August 2026                  |
| **Environment**    | VMware Lab                      |
| **Audit Platform** | Kali Linux                      |
| **Main Tools**     | BloodHound CE + SharpHound      |

---

# 🌐 Network Architecture

**Network:** `192.168.36.0/24`
**Network Type:** VMware NAT

```text
┌──────────────────────────────────────────────────────────────┐
│                    ACTIVE DIRECTORY LAB                      │
│                                                              │
│  VM 1 — Windows Server 2019                                  │
│  ├── Hostname: DC01                                          │
│  ├── IP Address: 192.168.36.10                               │
│  └── Roles: AD DS + DNS + DHCP                               │
│                                                              │
│  VM 2 — Windows 10 Client                                    │
│  ├── Hostname: DESKTOP-A92LHE9                               │
│  ├── IP Address: 192.168.36.101                              │
│  └── Role: Domain Client                                     │
│                                                              │
│  VM 3 — Kali Linux                                           │
│  ├── IP Address: 192.168.36.130                              │
│  └── Role: Security Audit / BloodHound Analysis              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

# 🏗️ Active Directory Structure

**Domain:** `proxym.tn`

```text
proxym.tn
│
├── Domain Controllers
│   └── DC01
│
├── Admins
│   ├── Admin1
│   └── Admin2
│
├── Users
│   ├── User1
│   └── User2
│
└── Computers
    └── DESKTOP-A92LHE9
```

## 👥 Created Accounts

| Account      | Role                     | Group         |
| ------------ | ------------------------ | ------------- |
| `SuperAdmin` | Built-in Administrator   | Domain Admins |
| `Admin1`     | Primary Administrator    | Domain Admins |
| `Admin2`     | Backup Administrator     | Domain Admins |
| `User1`      | Standard User            | Domain Users  |
| `User2`      | Standard User            | Domain Users  |
| `KRBTGT`     | Kerberos Service Account | —             |
| `XGuest`     | Guest Account (Disabled) | —             |

---

# 📸 Infrastructure

## Windows Server 2019 — Network Configuration

![Windows Server 2019 IP Configuration](docs/screenshots/01-infrastructure/01-server-ipconfig.png)

---

## Windows 10 Client — Network Configuration

![Windows 10 Client IP Configuration](docs/screenshots/01-infrastructure/02-client-ipconfig.png)

---

## Active Directory Users and Computers

![Active Directory Users and Computers](docs/screenshots/01-infrastructure/03-ad-users-computers.png)

---

## DHCP Scope Configuration

![DHCP Scope Configuration](docs/screenshots/01-infrastructure/04-dhcp-scope.png)

---

## DHCP Reservation

![DHCP Reservation](docs/screenshots/01-infrastructure/05-dhcp-reservation.png)

---

## DNS Configuration

![DNS Zones](docs/screenshots/01-infrastructure/06-dns-zones.png)

---

## Domain Controller Diagnostics

![DCDiag Results](docs/screenshots/01-infrastructure/07-dcdiag.png)

---

## Domain Join

![Windows 10 Domain Join](docs/screenshots/01-infrastructure/08-domain-join.png)

---

## Server Manager

![Windows Server Manager](docs/screenshots/01-infrastructure/09-server-manager.png)

---

## Administrative Accounts

![Administrative Accounts](docs/screenshots/01-infrastructure/10-admin-accounts.png)

---

## Network Shares

![Network Shares](docs/screenshots/01-infrastructure/11-net-share.png)

---

## Connectivity Tests

![Ping Tests](docs/screenshots/01-infrastructure/12-ping-tests.png)

---

# 🔧 Part 1 — Group Policy Configuration

## Configured Group Policies

| # | GPO                   | Description                      | Status |
| - | --------------------- | -------------------------------- | ------ |
| 1 | Default Domain Policy | Password + Account Lockout       | ✅      |
| 2 | GPO_Block_USB         | Complete USB Restriction         | ✅      |
| 3 | GPO_Windows_Security  | Windows Security Hardening       | ✅      |
| 4 | GPO_Defender          | Microsoft Defender Configuration | ✅      |
| 5 | GPO_Firewall          | Windows Firewall                 | ✅      |
| 6 | GPO_Windows_Update    | Automatic Updates                | ✅      |
| 7 | GPO_Software_Deploy   | 7-Zip Deployment                 | ✅      |
| 8 | GPO_Security_Users    | User Restrictions                | ✅      |

---

## All Configured GPOs

![All Configured GPOs](docs/screenshots/02-gpo/13-all-gpos-list.png)

---

## 🔐 Password Policy

![Password Policy](docs/screenshots/02-gpo/14-password-policy.png)

| Setting                 | Configuration |
| ----------------------- | ------------- |
| Minimum password length | 10 characters |
| Password complexity     | Enabled       |
| Maximum password age    | 60 days       |
| Minimum password age    | 1 day         |
| Password history        | 5 passwords   |
| Reversible encryption   | Disabled      |

---

## 🔒 Account Lockout Policy

![Account Lockout Policy](docs/screenshots/02-gpo/15-account-lockout.png)

| Setting           | Configuration |
| ----------------- | ------------- |
| Lockout threshold | 5 attempts    |
| Lockout duration  | 15 minutes    |
| Counter reset     | 15 minutes    |

---

## 🔌 USB Device Restrictions

![USB Block Policy](docs/screenshots/02-gpo/16-usb-block.png)

* ❌ All Removable Storage — Deny All
* ❌ USB Read Access — Denied
* ❌ USB Write Access — Denied
* ❌ CD/DVD Read Access — Denied
* ❌ CD/DVD Write Access — Denied
* ❌ WPD Devices — Denied

---

## 🛡️ Microsoft Defender Configuration

> Multiple screenshots were captured for this configuration.

### Microsoft Defender — Configuration 1

![Microsoft Defender Configuration 1](docs/screenshots/02-gpo/17-defender-gpo'.png)

### Microsoft Defender — Configuration 2

![Microsoft Defender Configuration 2](docs/screenshots/02-gpo/17-defender-gpo''.png)

### Microsoft Defender — Configuration 3

![Microsoft Defender Configuration 3](docs/screenshots/02-gpo/17-defender-gpo'''.png)

### Microsoft Defender — Configuration 4

![Microsoft Defender Configuration 4](docs/screenshots/02-gpo/17-defender-gpo''''.png)

### Security Settings

* ✅ Real-time protection enabled
* ✅ Behaviour monitoring enabled
* ☁️ Cloud protection configured
* 🔍 Scheduled scans configured
* 🔄 Security intelligence updates configured
* 📥 Download scanning enabled

---

## 🔥 Windows Firewall

![Windows Firewall GPO](docs/screenshots/02-gpo/18-firewall-gpo.png)

| Profile | Firewall | Inbound | Outbound |
| ------- | -------- | ------- | -------- |
| Domain  | ON       | Blocked | Allowed  |
| Private | ON       | Blocked | Allowed  |
| Public  | ON       | Blocked | Blocked  |

### Allowed Rules

| Service | Protocol / Port | Scope       |
| ------- | --------------- | ----------- |
| RDP     | TCP 3389        | Domain Only |
| DNS     | UDP 53          | Required    |
| ICMP    | Ping            | Allowed     |

---

## 🔄 Windows Update Configuration

![Windows Update GPO](docs/screenshots/02-gpo/19-update-gpo.png)

* ✅ Automatic updates enabled
* 📥 Automatic download and installation
* 🕒 Scheduled installation configured
* 🕗 Active hours configured
* 🔄 Restart behaviour configured

---

## 📦 Software Deployment

![Software Deployment](docs/screenshots/02-gpo/20-software-deploy.png)

| Setting         | Configuration                     |
| --------------- | --------------------------------- |
| Software        | 7-Zip                             |
| Deployment Type | Assigned                          |
| Source          | `\\DC01\Software_Deploy\7zip.msi` |
| Deployment      | Automatic at startup              |

---

## 📊 Applied GPOs on the Client

### GPResult

![GPResult Client](docs/screenshots/02-gpo/21-gpresult-client.png)

### Additional GPResult Evidence

![GPResult Client Additional](docs/screenshots/02-gpo/21-gpresult-client''.png)

---

## 📄 GPO HTML Report

![GPO HTML Report](docs/screenshots/02-gpo/22-gpo-html-report.png)

---

## 👤 Security Users GPO

![Security Users GPO](docs/screenshots/02-gpo/23-security-users-gpo.png)

---

# 🩸 Part 2 — BloodHound Analysis

## 🛠️ Tools Used

| Tool          | Version | Purpose                          |
| ------------- | ------- | -------------------------------- |
| SharpHound    | v2.14.0 | Active Directory Data Collection |
| BloodHound CE | v5.x    | Analysis and Visualisation       |
| Neo4j         | Latest  | Graph Database                   |
| PostgreSQL    | Latest  | Main Database                    |

---

## 📊 Collection Statistics

| Metric               |     Result |
| -------------------- | ---------: |
| Objects Collected    |        315 |
| Sessions             |          5 |
| ACEs                 |      1,819 |
| Relationships        |      2,982 |
| Group Completeness   |       100% |
| Session Completeness |       100% |
| Domain               |  proxym.tn |
| Collection Duration  | 12 seconds |

---

## BloodHound Login

![BloodHound Login](docs/screenshots/03-bloodhound/26-bh-login.png)

---

## BloodHound Dashboard

![BloodHound Dashboard](docs/screenshots/03-bloodhound/27-bh-dashboard.png)

---

## Data Quality

![BloodHound Data Quality](docs/screenshots/03-bloodhound/28-data-quality.png)

---

## All Active Directory Users

![All Active Directory Users](docs/screenshots/03-bloodhound/29-all-users.png)

---

## All Active Directory Computers

![All Active Directory Computers](docs/screenshots/03-bloodhound/30-all-computers.png)

---

## All Active Directory Groups

### Group Analysis 1

![All Active Directory Groups 1](docs/screenshots/03-bloodhound/31-all-groups'.png)

### Group Analysis 2

![All Active Directory Groups 2](docs/screenshots/03-bloodhound/31-all-groups''.png)

### Group Analysis 3

![All Active Directory Groups 3](docs/screenshots/03-bloodhound/31-all-groups'''.png)

---

## Object Count

![BloodHound Object Count](docs/screenshots/03-bloodhound/32-object-count.png)

### Active Directory Object Statistics

| Object Type   | Count |
| ------------- | ----: |
| Computers     |     2 |
| Users         |     9 |
| Groups        |    62 |
| OUs           |     4 |
| GPOs          |    10 |
| Sessions      |     5 |
| ACEs          | 1,819 |
| Relationships | 2,982 |

---

## Domain Admin Members

![Domain Admin Members](docs/screenshots/03-bloodhound/33-domain-admins-members.png)

---

## All Relationships

### Relationship Analysis 1

![All Relationships 1](docs/screenshots/03-bloodhound/34-all-relationships'.png)

### Relationship Analysis 2

![All Relationships 2](docs/screenshots/03-bloodhound/34-all-relationships''.png)

---

## Administrative Sessions

![Administrative Sessions](docs/screenshots/03-bloodhound/38-admin-sessions.png)

---

## Group Memberships

![Group Memberships](docs/screenshots/03-bloodhound/39-group-memberships.png)

---

## All Group Policy Objects

![All Group Policy Objects](docs/screenshots/03-bloodhound/40-all-gpos.png)

---

## All Organizational Units

![All Organizational Units](docs/screenshots/03-bloodhound/41-all-ous.png)

---

## High Value Targets

![High Value Targets](docs/screenshots/03-bloodhound/42-high-value-targets.png)

---

## Attack Path

![Attack Path](docs/screenshots/03-bloodhound/43-attack-path.png)

---

## DC01 Properties

![DC01 Properties](docs/screenshots/03-bloodhound/44-dc01-properties.png)

---

## Domain Admin Graph

![Domain Admin Graph](docs/screenshots/03-bloodhound/45-domain-admins-graph.png)

---

## Attack Path Graph

![Attack Path Graph](docs/screenshots/03-bloodhound/46-attack-path-graph.png)

---

## DC01 Inbound Control

![DC01 Inbound Control](docs/screenshots/03-bloodhound/49-dc01-inbound-control.png)

---

# 👥 Session Analysis

## Windows 10 Desktop Sessions

![Desktop Sessions](docs/screenshots/04-sessions/47-desktop-sessions.png)

---

## Active Sessions on Windows 10

![Active Sessions Windows 10](docs/screenshots/04-sessions/50-active-sessions-win10.png)

---

## Active Sessions on DC01

![Active Sessions DC01](docs/screenshots/04-sessions/51-active-sessions-dc01.png)

---

## Last Logon Analysis

![Last Logon Analysis](docs/screenshots/04-sessions/52-last-logon.png)

---

## Logon Events

![Windows Logon Events](docs/screenshots/04-sessions/53-logon-events.png)

---

# 🔐 ACL Analysis

## Dangerous ACLs

![Dangerous ACLs](docs/screenshots/05-acl/35-dangerous-acls.png)

---

## DCSync Rights

![DCSync Rights](docs/screenshots/05-acl/36-dcsync-rights.png)

---

## Admin2 Group Memberships

![Admin2 Group Memberships](docs/screenshots/05-acl/44-admin2-memberships.png)

---

## User1 Group Memberships

![User1 Group Memberships](docs/screenshots/05-acl/45-user1-memberships.png)

---

## Group Membership Analysis

![Group Membership Analysis](docs/screenshots/05-acl/46-group-memberships.png)

---

## Admin1 Properties

![Admin1 Properties](docs/screenshots/05-acl/48-admin1-properties.png)

---

# ⚠️ Security Findings

## Privileged Account Session

> **Finding:** A privileged account was identified with a session on a workstation.

| Category               | Details                                                          |
| ---------------------- | ---------------------------------------------------------------- |
| 🔴 Risk                | Administrative credentials could potentially be exposed          |
| ⚠️ Impact              | Increased risk of credential theft                               |
| ⚠️ Potential Technique | Pass-the-Hash                                                    |
| ⚠️ Potential Technique | Pass-the-Ticket                                                  |
| ✅ Recommendation       | Restrict privileged accounts to dedicated administrative systems |
| 🔐 Additional Control  | Implement Privileged Access Workstations (PAW)                   |

---

# 🔍 Identified Vulnerabilities

## 🔴 LDAP Signing Disabled

**Description:** LDAP communications are not configured to require signing.

**Risk:** Potential exposure to man-in-the-middle attacks and modification of LDAP communications.

**Impact:** 🔴 **CRITICAL**

---

## 🔴 Unconstrained Delegation Enabled

**Description:** Unconstrained Kerberos delegation was identified in the environment.

**Potential Risks:**

* Kerberos ticket exposure
* Potential impersonation of authenticated users
* Increased risk of credential compromise

**Impact:** 🔴 **CRITICAL**

---

## 🔴 LAPS Not Deployed

**Description:** Local Administrator Password Solution (LAPS) was not identified as deployed.

**Potential Risks:**

* Local administrator password reuse
* Increased risk of lateral movement
* Potential Pass-the-Hash exposure

**Impact:** 🔴 **HIGH**

---

## 🟡 Multiple Privileged Accounts

**Description:** Multiple accounts have privileged Domain Admin permissions.

**Risk:** Increasing the number of privileged accounts expands the attack surface.

**Impact:** 🟡 **MEDIUM**

---

## 🟡 Privileged Account on Workstation

**Description:** A privileged account session was identified on a workstation.

**Risk:**

* Privileged credential exposure
* Potential Pass-the-Hash attacks
* Potential Pass-the-Ticket attacks

**Impact:** 🟡 **MEDIUM**

---

# 🛠️ Remediation Plan

## 🔴 High Priority

| # | Action                      | Recommendation                               |
| - | --------------------------- | -------------------------------------------- |
| 1 | Enable LDAP Signing         | Configure Domain Controllers GPO             |
| 2 | Review Delegation           | Disable unnecessary unconstrained delegation |
| 3 | Deploy Microsoft LAPS       | Configure automatic password rotation        |
| 4 | Enable LDAP Channel Binding | Configure according to Microsoft guidance    |

---

## 🟡 Medium Priority

| # | Action               | Recommendation                         |
| - | -------------------- | -------------------------------------- |
| 5 | Review Domain Admins | Keep only required privileged accounts |
| 6 | Implement PAW        | Dedicated administrator workstations   |
| 7 | Enable MFA           | Protect privileged accounts            |
| 8 | Implement PAM        | Privileged access management           |

---

## 🟢 Long-Term Improvements

| #  | Action                | Recommendation                 |
| -- | --------------------- | ------------------------------ |
| 9  | BloodHound Audits     | Perform periodic assessments   |
| 10 | Monitor Event ID 4624 | SIEM or Event Log monitoring   |
| 11 | Security Awareness    | Train users and administrators |
| 12 | Implement JIT Access  | Temporary privileged access    |

---

# 🏆 Conclusion

## Audit Summary

| Category                        | Result       |
| ------------------------------- | ------------ |
| Active Directory Infrastructure | ✅ Functional |
| Security GPOs Configured        | ✅ 8          |
| Objects Analysed                | ✅ 315        |
| Sessions Analysed               | ✅ 5          |
| ACE Permissions Analysed        | ✅ 1,819      |
| Relationships Analysed          | ✅ 2,982      |

---

## 💪 Security Strengths

* ✅ SMB Signing enabled
* ✅ AES Encryption supported
* ✅ Strong password policy configured
* ✅ USB devices restricted
* ✅ Firewall configured
* ✅ Microsoft Defender configured through GPO
* ✅ Windows Updates configured
* ✅ DNS and DHCP infrastructure configured
* ✅ Active Directory environment successfully deployed

---

## ⚠️ Areas for Improvement

* ❌ LDAP Signing configuration requires review
* ❌ Microsoft LAPS deployment should be considered
* ❌ Kerberos delegation configuration requires review
* ❌ Privileged account exposure should be reduced
* ❌ Administrative sessions should be restricted to dedicated systems

---

## 📌 General Assessment

The `proxym.tn` Active Directory environment is functional and includes multiple security controls implemented through Group Policy Objects.

The BloodHound analysis provided visibility into users, computers, groups, sessions, ACLs, privileged relationships, and potential attack paths within the Active Directory environment.

The identified findings should be validated and prioritised according to the organisation's operational requirements and risk management process. Implementing the proposed remediation plan will help reduce the Active Directory attack surface and improve the overall security posture of the environment.

---

# 📁 Repository Structure

```text
goad/
│
├── README.md
│
└── docs/
    └── screenshots/
        │
        ├── 01-infrastructure/
        │   ├── 01-server-ipconfig.png
        │   ├── 02-client-ipconfig.png
        │   ├── 03-ad-users-computers.png
        │   ├── 04-dhcp-scope.png
        │   ├── 05-dhcp-reservation.png
        │   ├── 06-dns-zones.png
        │   ├── 07-dcdiag.png
        │   ├── 08-domain-join.png
        │   ├── 09-server-manager.png
        │   ├── 10-admin-accounts.png
        │   ├── 11-net-share.png
        │   └── 12-ping-tests.png
        │
        ├── 02-gpo/
        │
        ├── 03-bloodhound/
        │
        ├── 04-sessions/
        │
        └── 05-acl/
```

---

<div align="center">

## 🛡️ Active Directory Security Audit

**proxym.tn**

BloodHound CE • SharpHound • Active Directory • VMware

**Auditor:** Aya NASR
**Supervisor:** Houda Mabrouk
**Date:** 27 August 2026

</div>
