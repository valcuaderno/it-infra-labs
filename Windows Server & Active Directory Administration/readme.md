# Week 06 — Windows Server & Active Directory Administration

## 1. Lab Overview

**Objective:** Build and configure a Windows Server 2019 Active Directory environment, manage domain users and groups, configure shared folder permissions, deploy Group Policy, and perform basic Active Directory troubleshooting.

**Lab Environment:**

| Component | Configuration |
|---|---|
| Domain Controller | DC01 |
| Operating System | Windows Server 2019 Standard Evaluation |
| Domain Name | adlab.test |
| NetBIOS Name | ADLAB |
| Domain Controller IP | 192.168.85.132 |
| Network | VMware NAT (VMnet8) |
| Client OS | Windows Client VM |
| Client IP | 192.168.85.130 |
| Test User | jdoe |
| Security Group | IT-Admins |

## 2. Active Directory Installation and Configuration

Installed the Active Directory Domain Services (AD DS) and DNS roles on Windows Server 2019.

Configured the server as a new forest with the domain name:

`adlab.test`

**Verification commands:**

```powershell
Get-ADDomain
Get-ADDomainController
Get-Service DNS,NTDS,Netlogon,KDC
```

**Result:**
- AD DS successfully installed.
- Domain controller successfully promoted.
- DNS role installed and operational.
- Active Directory services running.

## 3. Organizational Unit (OU) Management

Created Organizational Units to organize users, computers, and groups.

**IT OU Structure:**

```text
adlab.test
└── IT
    ├── Users
    ├── Computers
    └── Groups
```

**HR OU Structure:**

```text
adlab.test
└── HR
    ├── Users
    ├── Computers
    └── Groups
```

Enabled accidental deletion protection on the main OUs.

## 4. User Account Management

Created a domain user account for testing.

| Attribute | Value |
|---|---|
| Full Name | John Doe |
| Username | jdoe |
| UPN | jdoe@adlab.test |
| Department | IT |
| Job Title | IT Support Technician |
| OU | IT/Users |

**Verification command:**

```powershell
Get-ADUser jdoe -Properties Enabled,Department,Title
```

**Result:** User account successfully created and enabled.

## 5. Security Group Management

Created a Global Security Group named:

`IT-Admins`

Group location:

`OU=Groups,OU=IT,DC=adlab,DC=test`

Added John Doe to the IT-Admins group.

**Verification commands:**

```powershell
Get-ADGroupMember IT-Admins
```

```powershell
Get-ADPrincipalGroupMembership jdoe
```

**Result:** John Doe successfully added to IT-Admins.

## 6. Shared Folder and Access Control

Created an IT shared folder:

`C:\LabShares\IT`

Configured NTFS permissions:
- ADLAB\IT-Admins — Modify
- SYSTEM — Existing system permissions
- Administrators — Existing administrative permissions

Created an SMB share:

`\\DC01\IT`

Configured SMB share permissions:
- Administrators — Full Control
- ADLAB\IT-Admins — Change
- Everyone — Removed

**Verification commands:**

```powershell
Get-SmbShareAccess -Name IT
```

```powershell
Get-Acl C:\LabShares\IT
```

**Client-side testing:**

```cmd
whoami
```

Expected:

`adlab\jdoe`

```powershell
Test-Path \\DC01\IT
```

Expected:

`True`

Successfully accessed the shared folder and created a test text file using the domain account.

## 7. Group Policy Management

Created a Group Policy Object:

`IT-User-Security-Policy`

Linked it to:

`OU=IT,DC=adlab,DC=test`

Configured the following policy:

**Path:**

User Configuration → Policies → Administrative Templates → Start Menu and Taskbar

**Policy:** Remove Run menu from Start Menu

**Configuration:** Enabled

Applied the policy:

```cmd
gpupdate /force
```

Verified the applied policy:

```cmd
gpresult /r /scope:user
```

**Result:**
- IT-User-Security-Policy appeared under Applied Group Policy Objects.
- The Run command was restricted for the test user.
- Policy successfully applied to the domain user jdoe.

## 8. Domain Client Integration

Configured the Windows Client VM to use DC01 as its DNS server.

**Client DNS:**

`192.168.85.132`

Joined the Windows client to:

`adlab.test`

Authenticated using the domain account:

`ADLAB\jdoe`

Verified the domain login:

```cmd
whoami
```

Expected:

`adlab\jdoe`

Successfully logged in using the Active Directory domain account.

## 9. Active Directory Troubleshooting

Performed basic troubleshooting and verification.

### Domain Controller Discovery

```cmd
nltest /dsgetdc:adlab.test
```

Result: Successfully discovered DC01.

### DNS Resolution

```cmd
nslookup adlab.test
```

```cmd
nslookup DC01.adlab.test
```

Result: Both resolved to 192.168.85.132.

### Active Directory Services

```powershell
Get-Service DNS,NTDS,Netlogon,KDC
```

Result: All four services were Running.

### Domain Controller Health Check

```powershell
dcdiag
```

Result: All reported tests passed.

### Group Policy Verification

```cmd
gpresult /r /scope:user
```

Result: IT-User-Security-Policy appeared under Applied Group Policy Objects.

## 10. Skills Practiced

- Windows Server 2019 administration
- Active Directory Domain Services
- DNS configuration and troubleshooting
- Organizational Unit management
- Active Directory user and group administration
- NTFS permissions
- SMB share permissions
- Group Policy Objects
- Domain joining and authentication
- Kerberos and Netlogon fundamentals
- PowerShell administration
- Active Directory troubleshooting

## 11. Conclusion

Successfully deployed a small enterprise-style Active Directory environment using Windows Server 2019 and a Windows client VM.

Demonstrated centralized identity management, security group membership, resource access control, Group Policy deployment, domain authentication, and basic troubleshooting.

This lab provided practical experience in Windows Server administration and Active Directory operations relevant to System Administrator, IT Support, and Network Security Engineer roles.

**Lab Status:** Completed  
**Week:** 06 of Cybersecurity Engineering Academy

---

## Evidence and Screenshots

Add screenshots to this section:

1. AD DS and DNS roles installed.
2. Domain Controller configuration.
3. Organizational Unit structure.
4. John Doe user account.
5. IT-Admins group membership.
6. NTFS and SMB share permissions.
7. GPO configuration.
8. Windows client domain join.
9. Successful domain login (`adlab\jdoe`).
10. `gpresult` showing applied GPO.
11. `dcdiag` successful test results.
