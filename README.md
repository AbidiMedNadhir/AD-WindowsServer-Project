![header](https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:1a0a0a,100:6b1010&height=200&section=header&text=Windows%20Server%202019&fontSize=42&fontColor=ffffff&fontAlignY=38&desc=Active%20Directory%20Administration%20%2526%20Security&descSize=18&descAlignY=60&descFontColor=e63946)

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Windows%20Server%202019-0078D6?style=for-the-badge&amp;logo=windows&amp;logoColor=white"/>
  <img src="https://img.shields.io/badge/Domain-Active%20Directory-e63946?style=for-the-badge&amp;logo=microsoft&amp;logoColor=white"/>
  <img src="https://img.shields.io/badge/Status-Completed-2ea043?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Environment-Lab%20%2F%20Academic-6b1010?style=for-the-badge"/>
</p>

---

## 🎯 Overview

This project is an academic lab focusing on the **deployment, administration, and security hardening** of a Windows Server 2019 environment with Active Directory Domain Services (AD DS).

The work covers the full lifecycle: from bare-metal server setup and domain creation, to fine-grained GPO enforcement, DHCP/DNS configuration, Firewall hardening, backup & recovery — and concludes with a hands-on simulation of two real-world AD attack techniques to reinforce defensive knowledge.

---

## 🏗️ Lab Environment

| Component | Details |
|-----------|---------|
| **Domain Controller** | Windows Server 2019 Standard Evaluation |
| **Hostname (DC)** | `PDC_Abidi_Med_Nadhir` |
| **Domain Name** | `test.local` |
| **Client Machine** | Windows 10 — `PC1_Abidi_Med_Nadhir` |
| **Network** | `192.168.176.0/24` — Gateway `192.168.176.2` |
| **Services** | AD DS · DNS · DHCP · Windows Defender Firewall |

---

## 📁 Repository Structure

```
AD-WindowsServer-Project/
├── README.md
├── Project Report AD.pdf                                        # Full technical report (34 pages)
└── Kerberoasting & Mimikatz Active Directory Attack Simulation.pdf  # Attack simulation presentation
```

---

## ⚙️ Implementation

### Part 1 — Environment Setup

- Installed and configured **Windows Server 2019** as the Primary Domain Controller
- Installed and configured a **Windows 10** client machine
- Created the **`test.local`** Active Directory domain
- Joined the Windows 10 client (`PC1_Abidi_Med_Nadhir`) to the domain

---

### Part 2 — Active Directory Object Management

**Organizational Units (OUs) created:**

| OU Name | Purpose |
|---------|---------|
| `services_informatiques_Nadhir_Abidi` | IT department users |
| `ressources_humaines_Nadhir_Abidi` | HR department users |
| `Administration_Nadhir_Abidi` | Administrative accounts |
| `Classe_ING4-SSIRF-A_Nadhir_Abidi` | Student accounts |

**Users created with enforced password-change on first login:**

| User | OU | Login |
|------|-----|-------|
| Oussama Riahi | services_informatiques | `oussama.riahi@test.local` |
| Mabrouk Olfa | Administration | `olfa.mabrouk@test.local` |
| Abidi Med Nadhir | Classe_ING4-SSIRF-A | `Abidi.Med.Nadhir@test.local` |

**Group management:**
- Created `IT-Group` with nested privilege groups: `Admins du domaine`, `Administrateurs DHCP`, `Administrateurs Hyper-V`
- Added `oussama.riahi` to `IT-Group`

**Domain join verification:**
- Logged in as `test\abidi.med.nadhir` on the Windows 10 client → confirmed via `whoami`

---

### Part 3 — Group Policy Objects (GPOs)

#### 🔒 GPO applied to `Classe_ING4-SSIRF-A` — `GPO_BLOCK_CMD_Nadhir`

| # | Policy | Effect |
|---|--------|--------|
| a | Block Command Prompt (`cmd`) | Users cannot open CMD |
| b | Disable Control Panel | Blocks access to system settings |
| c | Block PowerShell execution | `powershell.exe` added to blocked apps list |
| d | Block Registry Editor (`regedit`) | Prevents registry tampering |

#### 🔒 GPO applied to `Administration` — Password Policy

| # | Policy | Value |
|---|--------|-------|
| e | Minimum password length | 12 characters |
| f | Maximum password age | 60 days |
| g | Screen saver timeout | 5 minutes with password lock |
| h | Password history | 24 passwords memorized |

#### 🔒 GPO applied to `Ressources_Humaines` — `GPO_RH_RESTRICTIONS`

| # | Policy | Effect |
|---|--------|--------|
| j | Block local network connection properties | Users cannot modify NIC settings |
| k | Lock the taskbar | Taskbar locked, cannot be repositioned |

---

### Part 4 — DHCP Configuration

- Installed and configured the **DHCP Server** role on the domain controller
- Created an IPv4 scope: `192.168.176.0/24`
- Configured scope options:
  - **Default Gateway:** `192.168.176.2`
  - **DNS Domain:** `test.local`
  - **DNS Server:** `192.168.176.128`

---

### Part 5 — Windows Defender Firewall Rules

| Rule Name | Protocol | Port | Action |
|-----------|----------|------|--------|
| `BLOCK_TELNET_NADHIR` | TCP | 23 | Block |
| `BLCOK_FTP_NADHIR` | TCP | 21 | Block |

---

### Part 5 (cont.) — Backup & Recovery

- Configured **Windows Server Backup** for local backup
- Performed a successful backup on `28/12/2025 10:40`
- Verified file recovery via **Restore Files** — status: ✅ Success
- Backup data stored in `C:\data_Nadhir\`

---

## ⚔️ Part 6 — Attack Simulation (Educational Lab)

> ⚠️ **Disclaimer:** All attack simulations were performed in an isolated lab environment for educational purposes only. The goal is to understand Windows authentication weaknesses in order to build stronger defenses.

### 🔴 Kerberoasting

Kerberoasting exploits the Kerberos authentication protocol to extract service account ticket hashes (TGS) and crack them offline — without requiring any elevated privileges.

**Attack chain:**
1. Enumerate Service Principal Names (SPNs) in the domain
2. Request TGS tickets for those SPNs as a regular domain user
3. Extract the encrypted ticket hashes
4. Crack offline with hashcat or John the Ripper

**Mitigation:**
- Use long, random passwords (25+ chars) for service accounts
- Prefer **Group Managed Service Accounts (gMSA)** — auto-rotating passwords
- Monitor for unusual `TGS-REQ` Kerberos events (Event ID 4769)
- Apply **AES-only encryption** for Kerberos (disable RC4/DES)

---

### 🔴 Mimikatz — Credential Dumping

Mimikatz is a post-exploitation tool used to extract plaintext passwords, NTLM hashes, and Kerberos tickets directly from Windows memory (LSASS process).

**Key modules used in the simulation:**

| Module | Action |
|--------|--------|
| `sekurlsa::logonpasswords` | Dump credentials from LSASS |
| `lsadump::sam` | Dump SAM database hashes |
| `kerberos::list` | List in-memory Kerberos tickets |
| `pass-the-hash` | Authenticate using NTLM hash without cleartext password |

**Mitigation:**
- Enable **Credential Guard** (Windows Defender)
- Enable **Protected Users** security group for privileged accounts
- Restrict debug privileges — remove `SeDebugPrivilege` from non-admins
- Deploy **EDR/AV** solutions that detect LSASS memory access
- Monitor Event ID **4624 / 4625** and **10** (Sysmon LSASS access)

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [`Project Report AD.pdf`](./Project%20Report%20AD.pdf) | Full 34-page technical lab report with screenshots |
| [`Kerberoasting & Mimikatz Active Directory Attack Simulation.pdf`](./Kerberoasting%20%26%20Mimikatz%20Active%20Directory%20Attack%20Simulation.pdf) | Attack simulation slide deck |

---

## 🔗 Author

<table>
  <tr>
    <td><b>Name</b></td>
    <td>Mohamed Nadhir Abidi</td>
  </tr>
  <tr>
    <td><b>GitHub</b></td>
    <td><a href="https://github.com/AbidiMedNadhir">github.com/AbidiMedNadhir</a></td>
  </tr>
  <tr>
    <td><b>LinkedIn</b></td>
    <td><a href="https://linkedin.com/in/nadhir-abidi-95837325b">linkedin.com/in/nadhir-abidi-95837325b</a></td>
  </tr>
  <tr>
    <td><b>Email</b></td>
    <td>abidimednadhir@gmail.com</td>
  </tr>
</table>

---

![footer](https://capsule-render.vercel.app/api?type=waving&color=0:6b1010,50:1a0a0a,100:0d1117&height=120&section=footer)
