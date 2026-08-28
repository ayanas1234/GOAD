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

## Windows Server 2019

![Server IP Configuration](docs/screenshots/01-infrastructure/01-server-ipconfig.png)

## Windows 10 Client

![Client IP Configuration](docs/screenshots/01-infrastructure/02-client-ipconfig.png)

## Active Directory Users and Computers

![Active Directory Users](docs/screenshots/01-infrastructure/03-ad-users-computers.png)

## DHCP Configuration

![DHCP Configuration](docs/screenshots/01-infrastructure/04-dhcp-scope.png)

## DNS Configuration

![DNS Configuration](docs/screenshots/01-infrastructure/05-dns-zones.png)

## DCDiag Results

![DCDiag Results](docs/screenshots/01-infrastructure/06-dcdiag.png)

## Domain Join

![Domain Join](docs/screenshots/01-infrastructure/07-domain-join.png)

## Server Manager

![Server Manager](docs/screenshots/01-infrastructure/08-server-manager.png)

## Network Shares

![Net Share](docs/screenshots/01-infrastructure/09-net-share.png)

## Connectivity Tests

![Ping Tests](docs/screenshots/01-infrastructure/10-ping-tests.png)

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

### All Configured GPOs

![All GPOs](docs/screenshots/02-gpo/11-all-gpos-list.png)

---

## 🔐 GPO 1 — Password Policy

![Password Policy](docs/screenshots/02-gpo/12-password-policy.png)

| Setting                 | Configuration |
| ----------------------- | ------------- |
| Minimum password length | 10 characters |
| Password complexity     | Enabled       |
| Maximum password age    | 60 days       |
| Minimum password age    | 1 day         |
| Password history        | 5 passwords   |
| Reversible encryption   | Disabled      |

---

## 🔒 GPO 2 — Account Lockout Policy

![Account Lockout](docs/screenshots/02-gpo/13-account-lockout.png)

| Setting           | Configuration |
| ----------------- | ------------- |
| Lockout threshold | 5 attempts    |
| Lockout duration  | 15 minutes    |
| Counter reset     | 15 minutes    |

---

## 🔌 GPO 3 — USB Device Restrictions

![USB Block](docs/screenshots/02-gpo/14-usb-block.png)

* ❌ All Removable Storage — Deny All
* ❌ USB Read Access — Denied
* ❌ USB Write Access — Denied
* ❌ CD/DVD Read Access — Denied
* ❌ CD/DVD Write Access — Denied
* ❌ WPD Devices — Denied

---

## 🪟 GPO 4 — Windows Security Hardening

* ❌ Remote Registry disabled
* ❌ Telnet disabled
* ❌ Guest account disabled
* ⚠️ Security warning message enabled
* 🔒 Last logged-in user hidden
* 🔒 Shutdown without login disabled

---

## 🛡️ GPO 5 — Microsoft Defender

![Defender](docs/screenshots/02-gpo/15-defender-gpo.png)

* ✅ Real-time protection enabled
* ✅ Behaviour monitoring enabled
* ☁️ Cloud protection configured at high level
* 🔍 Full scheduled scan every Saturday at 02:00
* 🔄 Security intelligence updates every 8 hours
* 📥 Download scanning enabled

---

## 🔥 GPO 6 — Windows Firewall

![Firewall](docs/screenshots/02-gpo/16-firewall-gpo.png)

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

## 🔄 GPO 7 — Windows Update

![Windows Update](docs/screenshots/02-gpo/17-update-gpo.png)

* ✅ Automatic updates enabled
* 📥 Automatic download and installation
* 🕒 Scheduled installation: **03:00**
* 🕗 Active hours: **08:00 – 18:00**
* 🔄 No forced restart during active hours

---

## 📦 GPO 8 — Software Deployment

![Software Deployment](docs/screenshots/02-gpo/18-software-deploy.png)

| Setting         | Configuration                     |
| --------------- | --------------------------------- |
| Software        | 7-Zip 24.07                       |
| Deployment Type | Assigned                          |
| Source          | `\\DC01\Software_Deploy\7zip.msi` |
| Deployment      | Automatic at startup              |

---

## 📊 Applied GPOs on the Client

![GPResult](docs/screenshots/02-gpo/19-gpresult-client.png)

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

## SharpHound Data Collection

![SharpHound](docs/screenshots/03-bloodhound/20-sharphound-running.png)

## BloodHound Login

![BloodHound Login](docs/screenshots/03-bloodhound/22-bh-login.png)

## BloodHound Dashboard

![BloodHound Dashboard](docs/screenshots/03-bloodhound/23-bh-dashboard.png)

## Data Quality Statistics

![Data Quality](docs/screenshots/03-bloodhound/24-data-quality.png)

## All Users

![All Users](docs/screenshots/03-bloodhound/25-all-users.png)

## All Computers

![All Computers](docs/screenshots/03-bloodhound/26-all-computers.png)

## Object Count

![Object Count](docs/screenshots/03-bloodhound/27-object-count.png)

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

![Domain Admin Members](docs/screenshots/03-bloodhound/28-domain-admins-members.png)

## All Relationships

![All Relationships](docs/screenshots/03-bloodhound/29-all-relationships.png)

## DC01 Properties

![DC01 Properties](docs/screenshots/03-bloodhound/30-dc01-properties.png)

## Domain Admin Graph

![Domain Admin Graph](docs/screenshots/03-bloodhound/31-domain-admins-graph.png)

## Attack Path Analysis

![Attack Path](docs/screenshots/03-bloodhound/32-attack-path.png)

---

# 👥 Session Analysis

## Method 1 — Windows Command

```cmd
query user /server:DESKTOP-A92LHE9
```

### Active Sessions on Windows 10

| User     | Computer        | Status |
| -------- | --------------- | ------ |
| `User1`  | DESKTOP-A92LHE9 | Active |
| `Admin1` | DESKTOP-A92LHE9 | Active |

### Sessions on DC01

```cmd
query user /server:DC01
```

---

## Method 2 — PowerShell

```powershell
Get-ADUser -Filter * -Properties LastLogonDate |
Select-Object Name, LastLogonDate, Enabled |
Sort-Object LastLogonDate -Descending |
Format-Table -AutoSize
```

---

## Method 3 — BloodHound / Cypher

```cypher
MATCH (u:User)-[:HasSession]->(c:Computer)
RETURN u.name AS User, c.name AS Computer
```

---

# ⚠️ Security Finding — Privileged Account Session

> **Finding:** `Admin1` was logged into `DESKTOP-A92LHE9`.

| Category               | Details                                                          |
| ---------------------- | ---------------------------------------------------------------- |
| 🔴 Risk                | Administrative credentials could potentially be exposed          |
| ⚠️ Impact              | Increased risk of credential theft                               |
| ⚠️ Potential Technique | Pass-the-Hash                                                    |
| ⚠️ Potential Technique | Pass-the-Ticket                                                  |
| ✅ Recommendation       | Restrict privileged accounts to dedicated administrative systems |
| 🔐 Additional Control  | Implement Privileged Access Workstations (PAW)                   |

---

# 🔐 Active Directory Security Analysis

## DC01 Security Properties

| Parameter                | Value | Risk Level  |
| ------------------------ | ----- | ----------- |
| LDAP Signing             | FALSE | 🔴 CRITICAL |
| LAPS Enabled             | FALSE | 🔴 HIGH     |
| Unconstrained Delegation | TRUE  | 🔴 CRITICAL |
| SMB Signing              | TRUE  | ✅ GOOD      |
| LDAPS Available          | TRUE  | ✅ GOOD      |
| AES Encryption           | TRUE  | ✅ GOOD      |

---

## 👑 Domain Admin Members

```text
SUPERADMIN@PROXYM.TN  → Built-in Administrator
ADMIN1@PROXYM.TN      → Primary Administrator
ADMIN2@PROXYM.TN      → Backup Administrator
```

---

# 🔍 Identified Vulnerabilities

## 🔴 Vulnerability 1 — LDAP Signing Disabled

**Description:** LDAP communications are not configured to require signing.

**Risk:** Potential exposure to man-in-the-middle attacks and modification of LDAP communications.

**Impact:** 🔴 **CRITICAL**

### Recommended Remediation

Configure:

```text
Default Domain Controllers Policy
└── Security Settings
    └── Security Options
        └── Domain controller: LDAP server signing requirements
            → Require signing
```

---

## 🔴 Vulnerability 2 — Unconstrained Delegation Enabled

**Description:** Unconstrained Kerberos Delegation is enabled on the Domain Controller.

**Risk:**

* Kerberos ticket exposure
* Potential impersonation of authenticated users
* Increased risk of credential compromise

**Impact:** 🔴 **CRITICAL**

### Recommended Remediation

* Disable unnecessary unconstrained delegation.
* Use constrained delegation where appropriate.
* Review delegation settings in Active Directory.
* Apply the principle of least privilege.

---

## 🔴 Vulnerability 3 — LAPS Not Deployed

**Description:** Local Administrator Password Solution (LAPS) is not deployed.

**Risk:**

* Potential reuse of local administrator credentials.
* Increased risk of lateral movement.
* Potential Pass-the-Hash exposure.

**Impact:** 🔴 **HIGH**

### Recommended Remediation

* Deploy Microsoft LAPS.
* Enable automatic password rotation.
* Ensure unique local administrator passwords.

---

## 🟡 Vulnerability 4 — Excessive Domain Admin Accounts

**Description:** Three active accounts have Domain Admin privileges.

**Risk:** Increasing the number of privileged accounts expands the attack surface.

**Impact:** 🟡 **MEDIUM**

### Recommended Remediation

* Reduce privileged accounts to the minimum required.
* Apply a Tier 0 / Tier 1 / Tier 2 administrative model.
* Consider Just-In-Time (JIT) privileged access.

---

## 🟡 Vulnerability 5 — Administrator Logged Into a Workstation

**Description:** `Admin1` has an active session on `DESKTOP-A92LHE9`.

**Risk:**

* Privileged credential exposure.
* Potential Pass-the-Hash attacks.
* Potential Pass-the-Ticket attacks.

**Impact:** 🟡 **MEDIUM**

### Recommended Remediation

* Deploy Privileged Access Workstations (PAW).
* Restrict privileged accounts to administrative systems.
* Separate standard user and administrator accounts.

---

# 🛠️ Remediation Plan

## 🔴 High Priority — Immediate

| # | Action                           | Recommendation                            |
| - | -------------------------------- | ----------------------------------------- |
| 1 | Enable LDAP Signing              | Configure Domain Controllers GPO          |
| 2 | Disable Unconstrained Delegation | Review AD delegation settings             |
| 3 | Deploy Microsoft LAPS            | Configure automatic password rotation     |
| 4 | Enable LDAP Channel Binding      | Configure according to Microsoft guidance |

---

## 🟡 Medium Priority — Within 1 Month

| # | Action               | Recommendation                         |
| - | -------------------- | -------------------------------------- |
| 5 | Reduce Domain Admins | Keep only required privileged accounts |
| 6 | Implement PAW        | Dedicated administrator workstations   |
| 7 | Enable MFA           | Protect privileged accounts            |
| 8 | Implement PAM        | Deploy privileged access management    |

---

## 🟢 Low Priority — Within 3 Months

| #  | Action                    | Recommendation                |
| -- | ------------------------- | ----------------------------- |
| 9  | Monthly BloodHound Audits | Automate periodic assessments |
| 10 | Monitor Event ID 4624     | SIEM or Event Log monitoring  |
| 11 | User Security Awareness   | Security awareness training   |
| 12 | Implement JIT Access      | Temporary privileged access   |

---

# 🏆 Conclusion

## Audit Summary

| Category                        | Result       |
| ------------------------------- | ------------ |
| Active Directory Infrastructure | ✅ Functional |
| Security GPOs Configured        | ✅ 8          |
| Objects Analysed                | ✅ 315        |
| Active Sessions Identified      | ✅ 2          |
| ACE Permissions Analysed        | ✅ 1,819      |
| Critical / High Findings        | 🔴 3         |
| Medium Findings                 | 🟡 2         |
| Total Recommendations           | 📋 12        |

---

## 💪 Security Strengths

* ✅ SMB Signing enabled
* ✅ AES Encryption supported
* ✅ Strong password policy configured
* ✅ USB devices restricted
* ✅ Firewall configured for all profiles
* ✅ Microsoft Defender enforced through GPO
* ✅ Automatic Windows Updates configured
* ✅ Administrative account redundancy
* ✅ LDAPS available

---

## ⚠️ Areas for Improvement

* ❌ LDAP Signing not required
* ❌ Microsoft LAPS not deployed
* ❌ Unconstrained Kerberos Delegation enabled
* ❌ Multiple Domain Admin accounts
* ❌ Privileged account logged into a workstation

---

## 📌 General Assessment

The `proxym.tn` Active Directory infrastructure is functional and includes a solid baseline of security controls through Group Policy Objects.

However, the BloodHound analysis identified several important security weaknesses requiring remediation. The highest-priority findings concern LDAP signing, unconstrained Kerberos delegation, and the absence of Microsoft LAPS.

Implementing the proposed remediation plan will significantly reduce the Active Directory attack surface and improve the overall security posture of the environment.

---

# 📚 Project Information

```text
Project:      Active Directory Security Audit
Environment:  VMware Laboratory
Domain:       proxym.tn

Main Tools:
├── BloodHound CE
├── SharpHound
├── Neo4j
├── PostgreSQL
└── Kali Linux

Auditor:      Aya NASR
Supervisor:   Houda Mabrouk
Date:         27 August 2026
```

---

<div align="center">

### 🛡️ Active Directory Security Audit — proxym.tn

**BloodHound CE • SharpHound • Active Directory • VMware**

Made for an authorised security audit environment.

</div>
