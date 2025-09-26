# WN10-CC-000005
# 📷 Vulnerability Management Lab – WN10-CC-000005

**Title:** Camera access from the lock screen must be disabled  
**STIG ID:** WN10-CC-000005  
**Compliance Standards:** DISA STIG, NIST 800-53, HIPAA, PCI DSS  
**Lab Type:** Vulnerability Simulation → Tenable Scan → PowerShell Remediation  
**Tools Used:** Azure, Windows 10, PowerShell, Tenable.sc / Nessus  

---

## 📋 Lab Objective

This lab demonstrates how to detect and remediate a misconfiguration in **Windows 10 lock screen camera access** using Azure and Tenable tools. You'll simulate the vulnerability `WN10-CC-000005`, detect it through a Tenable STIG compliance scan, remediate using PowerShell, and confirm success through verification scans.

---

## 📁 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Simulation](#vulnerability-simulation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Vulnerability Scan](#initial-vulnerability-scan)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Post-Remediation Verification](#post-remediation-verification)  
7. [Security Rationale](#security-rationale)  
8. [Appendix: PowerShell Commands](#appendix-powershell-commands)

---

## ☁️ Azure VM Setup

### 🔹 VM Provisioning

| Setting              | Value                    |
|----------------------|--------------------------|
| VM Name              | `Win10-STIGLab-Camera`   |
| OS Image             | Windows 10 Pro (Gen 2)   |
| VM Size              | Standard D2s v3          |
| Resource Group       | `vm-lab-camera`          |
| Region               | Closest to analyst       |

### 🔹 Credential Requirements

> ⚠️ **DO NOT use weak/default credentials** (e.g., `labuser/Cyberlab123!`)  
> ✅ Use a strong password that meets complexity standards and is securely stored.

### 🔹 Network Security Group (NSG)

- Inbound Rules:
  - ✅ RDP (TCP 3389)
  - ✅ WinRM (TCP 5985) *(if needed)*
  - ❌ Deny all other ports

### 🔹 VM Configuration

#### Disable Windows Firewall *(lab only)*
- Open `wf.msc`
- Disable Domain, Private, and Public profiles

#### Allow Remote PowerShell
```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

---

## ⚠️ Vulnerability Simulation

### 🔸 STIG Overview

By default, Windows 10 may allow **camera access on the lock screen**, creating a privacy and physical security risk. STIG ID `WN10-CC-000005` requires this setting to be **disabled**.

### 🔸 Simulate the Vulnerability

```powershell
# Simulate insecure camera access on lock screen
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Camera" `
  -Name "AllowCamera" -Value 1 -Type DWord
```

> `1` = Camera is accessible from the lock screen (non-compliant)  
> `0` = Camera is blocked on the lock screen (compliant)

📸 **Screenshot Placeholder:** `Screenshot_01_Camera_Vulnerability_Config_PowerShell.png`

---

## 🔍 Tenable Scan Configuration

### 🔸 Template: **Advanced Network Scan**

#### ✅ Basic Settings
- Name: `STIG Scan – Lock Screen Camera Access`
- Targets: Public IP of the Azure VM

#### ✅ Discovery Settings
- Ping remote host  
- TCP Full Connect  
- Enable NetBIOS/SMB Discovery

#### ✅ Assessment Settings
- Enable:
  - Remote Registry  
  - Admin Shares  
  - Thorough Checks  
- Provide valid admin credentials for authenticated scan

#### ✅ Compliance Checks
- Audit File:  
  `DISA STIG – Microsoft Windows 10 v3r4.audit`

---

## 🧪 Initial Vulnerability Scan

After completing the scan, review the compliance report in Tenable.

| STIG Control | WN10-CC-000005 |
|--------------|----------------|
| Status       | ❌ **Fail**     |
| Detected     | `AllowCamera = 1` |
| Required     | `AllowCamera = 0` |

📸 **Screenshot Placeholder:** `Screenshot_02_Tenable_Vuln_Finding_BeforeFix.png`

---

## 🛠️ Remediation via PowerShell

### 🔸 Apply Secure Configuration

```powershell
# Disable camera access from lock screen (compliant setting)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Camera" `
  -Name "AllowCamera" -Value 0 -Type DWord
```

📸 **Screenshot Placeholder:** `Screenshot_03_PowerShell_Remediation_Applied.png`

---

## 🔁 Post-Remediation Verification

### 🔸 Manual Registry Check

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Camera" `
  -Name "AllowCamera"
```

**Expected Output:**
```
AllowCamera : 0
```

### 🔸 Tenable Rescan

| STIG Control | WN10-CC-000005 |
|--------------|----------------|
| Status       | ✅ **Pass**     |
| Value        | `AllowCamera = 0` |

📸 **Screenshot Placeholder:** `Screenshot_04_Tenable_AfterRemediation_Pass.png`

---

## 🔐 Security Rationale

Allowing lock screen camera access may:
- Bypass device restrictions in secure environments  
- Allow unintended recordings or information leakage  
- Undermine physical access controls

### Compliance Mapping

| Framework       | Requirement                              |
|------------------|-------------------------------------------|
| **DISA STIG**     | WN10-CC-000005                            |
| **NIST 800-53**   | AC-17, PE-3 – Physical & remote access    |
| **HIPAA**         | §164.312(a)(1) – Access control           |
| **PCI DSS**       | 7.2.1 – Limit access to system components |

---

## 🧼 Post-Lab Cleanup

After completing the lab, perform the following cleanup:

- ✅ Restart the VM to ensure all registry settings apply  
- 🧹 Delete the resource group to clean up lab resources:

```bash
az group delete --name vm-lab-camera --yes --no-wait
```

- 🔐 Remove or rotate any lab credentials used during scanning

---

## 📎 Appendix: PowerShell Commands

| Task                      | Command |
|---------------------------|---------|
| Simulate vulnerability    | `Set-ItemProperty -Path HKLM:\...\Camera -Name AllowCamera -Value 1` |
| Apply secure config       | `Set-ItemProperty -Path HKLM:\...\Camera -Name AllowCamera -Value 0` |
| Verify registry value     | `Get-ItemProperty -Path HKLM:\...\Camera -Name AllowCamera` |
| Allow remote PowerShell   | `Set-ItemProperty -Path HKLM:\...\System -Name LocalAccountTokenFilterPolicy -Value 1` |

---

✅ **Lab Complete**

You've successfully executed the full vulnerability management lifecycle for **WN10-CC-000005** — from misconfiguration to Tenable detection, PowerShell remediation, and compliance verification.

Browse the `/labs/` directory for more STIG remediation guides.
