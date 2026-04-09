# Fine-Grained Password Policies in Active Directory

> **Author:** Rifat Rahman  
> **Date:** April 9, 2026  
> **Domain:** Active Directory | Windows Server | IT Security

---

## Overview

This project documents the configuration and management of **Fine-Grained Password Policies (FGPPs)** in Active Directory Domain Services (AD DS). FGPPs allow administrators to define different password and account lockout policies for different groups of users within the **same AD domain** — for example, enforcing stricter settings for administrators while giving standard users more relaxed requirements.

FGPPs are implemented using **Password Settings Objects (PSOs)**, which can be linked to users or global security groups.

---

## Table of Contents

- [Key Concepts](#key-concepts)
- [Prerequisites](#prerequisites)
- [Designing Your FGPP](#designing-your-fgpp)
- [Creating an FGPP via ADAC (GUI)](#creating-an-fgpp-via-adac-gui)
- [Creating an FGPP via PowerShell](#creating-an-fgpp-via-powershell)
- [Checking Which Policy Applies](#checking-which-policy-applies)
- [Important Behaviors and Limits](#important-behaviors-and-limits)
- [Best Practices](#best-practices)
- [Files in This Repository](#files-in-this-repository)
- [References](#references)

---

## Key Concepts

| Concept | Description |
|---|---|
| **PSO** | Password Settings Object — the AD object that stores FGPP settings |
| **Precedence** | Lower number = higher priority when multiple PSOs apply |
| **Domain Functional Level** | Must be Windows Server 2008 or higher |
| **Scope** | Applies to domain users or global security groups only |

---

## Prerequisites

- Domain functional level set to **Windows Server 2008 or higher**
- Membership in **Domain Admins** or equivalent rights
- RSAT tools installed:
  - **Active Directory Administrative Center (ADAC)** — `dsac.exe`
  - **Active Directory module for Windows PowerShell**

---

## Designing Your FGPP

Before creating any PSO, plan which groups need special policies:

| Group Type | Recommended Settings |
|---|---|
| Domain Admins / Privileged | Long passwords (15+ chars), strict lockout, short max age |
| Service Accounts | Very long passwords, no interactive logon, monitored separately |
| Standard Users | Balanced complexity and usability |

**Key settings to configure:**
- Minimum password length
- Password history count
- Maximum / minimum password age
- Complexity requirement
- Lockout threshold, duration, and observation window

---

## Creating an FGPP via ADAC (GUI)

1. Open **Active Directory Administrative Center** (`dsac.exe`) from Server Manager → Tools
2. Select the correct domain in ADAC
3. Expand **System** → open the **Password Settings Container**
4. In the Tasks pane, click **New → Password Settings**
5. Fill in all PSO fields (Name, Precedence, password settings, lockout settings)
6. Under **Directly Applies To**, add the target security group or user
7. Click **OK** to save

### Example: Strict Policy for Server Admins

```
Name:                  Admins-Strict-Policy
Precedence:            1
Min Password Length:   15 characters
Password History:      24 passwords
Max Password Age:      30 days
Lockout Threshold:     5 invalid attempts
Lockout Duration:      30 minutes
Applied To:            Server-Admins (global security group)
```

---

## Creating an FGPP via PowerShell

```powershell
Import-Module ActiveDirectory

$policyParams = @{
    Name                            = "Admins-Strict-Policy"
    Precedence                      = 1
    ComplexityEnabled               = $true
    MinPasswordLength               = 15
    PasswordHistoryCount            = 24
    MaxPasswordAge                  = "30.00:00:00"
    MinPasswordAge                  = "1.00:00:00"
    LockoutThreshold                = 5
    LockoutDuration                 = "00:30:00"
    LockoutObservationWindow        = "00:30:00"
    ReversibleEncryptionEnabled     = $false
    ProtectedFromAccidentalDeletion = $true
}

New-ADFineGrainedPasswordPolicy @policyParams

# Link the PSO to a security group
Add-ADFineGrainedPasswordPolicySubject `
    -Identity "Admins-Strict-Policy" `
    -Subjects "Server-Admins"
```

---

## Checking Which Policy Applies

### Via ADAC
1. Open Active Directory Administrative Center
2. Browse to the user object
3. Open **Properties** → view the effective password policy section

### Via PowerShell

```powershell
Import-Module ActiveDirectory

# Show the resultant PSO for a specific user
Get-ADUserResultantPasswordPolicy -Identity "some.user"
```

---

## Important Behaviors and Limits

- FGPPs **override** the default domain password policy for targeted users
- When a user belongs to multiple groups with different PSOs, the PSO with the **lowest Precedence value wins**
- FGPPs only apply to **domain users or global security groups** — not local accounts
- A sensible base policy at the domain level is still required; FGPPs refine it, they do not replace it entirely

---

## Best Practices

- Apply FGPPs first to **privileged accounts** (Domain Admins, Tier 0 systems)
- Set very strong requirements for those groups — long passwords, high history count, strict lockout
- Keep the FGPP design **simple** — avoid too many different PSOs
- Always **document** which groups receive which policy
- Combine FGPPs with **Multi-Factor Authentication (MFA)** and **Privileged Access Management (PAM)** where possible

---

## Files in This Repository

| File | Description |
|---|---|
| `Password_and_Account_Lockout.pdf` | Full technical guide with step-by-step instructions, PowerShell scripts, and references |
| `README.md` | Project overview and quick reference guide |

---

## References

1. [Microsoft Docs — Configure Fine-Grained Password Policies for AD DS](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/adac/fine-grained-password-policies)
2. [ActiveDirectoryPro — Create Fine Grained Password Policy (Step-by-Step)](https://activedirectorypro.com/create-fine-grained-password-policies/)
3. [Cayosoft — How to Protect AD with Fine-Grained Password Policy](https://www.cayosoft.com/blog/fine-grained-password-policy/)
4. [CSA Cyber — Implementing a Fine-grained Password Policy for Domain Admins](https://csacyber.com/blog/implementing-a-fine-grained-password-policy-for-domain-admins)
5. [Securden — Active Directory Password Policy: Configuration & Best Practices](https://www.securden.com/educational/active-directory-password-policy.html)
6. [AdminDroid — Set Up Fine-Grained Password Policies in Active Directory](https://blog.admindroid.com/how-to-configure-fine-grained-password-policy-in-active-directory/)
7. [Windows-FAQ — Die Limits der Fine Grained Password Policies in Windows AD](https://www.windows-faq.de/2023/04/18/die-limits-der-fine-grained-password-policies-in-windows-ad/)

---

*This project is part of a personal IT & Active Directory learning portfolio by Rifat Rahman.*
