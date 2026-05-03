Home Lab Project — Part 04
# 📁 File Server with NTFS Permissions on Windows Server 2022

![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue?style=flat-square&logo=windows)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Integrated-green?style=flat-square)
![NTFS](https://img.shields.io/badge/NTFS-Permissions-orange?style=flat-square)
![SMB](https://img.shields.io/badge/SMB-File%20Share-blue?style=flat-square)
![PowerShell](https://img.shields.io/badge/PowerShell-Automation-blue?style=flat-square&logo=powershell)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)

> **Home Lab Project — Part 4**
> Configuring a centralized File Server on Windows Server 2022 with department-based NTFS permissions, Active Directory Security Groups, and role-based access control — simulating real enterprise file server management.

---

## 📑 Table of Contents

- [Lab Environment](#lab-environment)
- [Objectives](#objectives)
- [What is a File Server?](#what-is-a-file-server)
- [Folder Structure](#folder-structure)
- [Security Groups](#security-groups)
- [Department Users](#department-users)
- [SMB Share Configuration](#smb-share-configuration)
- [NTFS Permissions](#ntfs-permissions)
- [Access Verification](#access-verification)
- [Permission Summary](#permission-summary)
- [Architecture](#architecture)
- [Issues & Fixes](#issues--fixes)
- [Skills Demonstrated](#skills-demonstrated)
- [Related Projects](#related-projects)

---

## 🏗️ Lab Environment

| Component | Details |
|---|---|
| **Hypervisor** | Oracle VirtualBox |
| **Domain Controller** | WIN-6ARV79FMB48 — Windows Server 2022 |
| **Client Machine** | CPC-NSimon-001 — Windows 10 Pro |
| **Domain** | mylab.local |
| **Server IP** | 192.168.1.200 |
| **File Server Path** | C:\FileServer |
| **Share Name** | \\192.168.1.200\FileServer |

---

## 🎯 Objectives

- [x] Create department folder structure on Windows Server 2022
- [x] Create Active Directory Security Groups per department
- [x] Create department user accounts
- [x] Add users to their respective security groups
- [x] Configure SMB share with Everyone — Full Control
- [x] Set NTFS permissions per department folder
- [x] Disable permission inheritance on all folders
- [x] Verify access as ituser01 — IT and General only
- [x] Verify access as hruser01 — HR and General only

---

## 📖 What is a File Server?

A **File Server** is a centralized storage server where users can access, store, and share files based on their role in the organization. Instead of storing files locally on each computer, files are stored on the server — making them accessible from any computer on the network.

### Two Layers of Permissions

| Layer | Controls | Applied To |
|---|---|---|
| **Share Permissions** | Who can access the folder over the network | The shared folder root |
| **NTFS Permissions** | Who can access files at the file system level | Each individual folder/file |

> **Best Practice:** Set Share Permissions to "Everyone — Full Control" and manage all access control through NTFS Permissions. This gives more granular and secure control.

### Why Security Groups instead of individual users?

In enterprise environments, permissions are **never** assigned directly to individual users. Instead, they are assigned to **Security Groups** using **Role-Based Access Control (RBAC)**:

- ✅ Easier to manage — add/remove users from groups instead of changing permissions
- ✅ More secure — permissions are consistent across all group members
- ✅ Scalable — new employees just get added to the right group
- ✅ Auditable — easy to see who has access to what

---

## 📂 Folder Structure

```
C:\FileServer\
├── IT\           → IT-Team only (Modify)
├── HR\           → HR-Team only (Modify)
├── Finance\      → Finance-Team only (Modify)
├── Management\   → Management-Team only (Modify)
└── General\      → Everyone (Read)
```

```powershell
# Create folder structure
New-Item -Path "C:\FileServer" -ItemType Directory
New-Item -Path "C:\FileServer\IT" -ItemType Directory
New-Item -Path "C:\FileServer\HR" -ItemType Directory
New-Item -Path "C:\FileServer\Finance" -ItemType Directory
New-Item -Path "C:\FileServer\Management" -ItemType Directory
New-Item -Path "C:\FileServer\General" -ItemType Directory
```

---

## 👥 Security Groups

Security Groups were created in the VMUsers OU and linked to their respective department folders.

```powershell
New-ADGroup -Name "IT-Team" `
  -GroupScope Global -GroupCategory Security `
  -Path "OU=VMUsers,DC=mylab,DC=local" `
  -Description "IT Department Access Group"

New-ADGroup -Name "HR-Team" `
  -GroupScope Global -GroupCategory Security `
  -Path "OU=VMUsers,DC=mylab,DC=local" `
  -Description "HR Department Access Group"

New-ADGroup -Name "Finance-Team" `
  -GroupScope Global -GroupCategory Security `
  -Path "OU=VMUsers,DC=mylab,DC=local" `
  -Description "Finance Department Access Group"

New-ADGroup -Name "Management-Team" `
  -GroupScope Global -GroupCategory Security `
  -Path "OU=VMUsers,DC=mylab,DC=local" `
  -Description "Management Department Access Group"
```

### Groups Created

| Group | Scope | Category | Folder Access |
|---|---|---|---|
| **IT-Team** | Global | Security | IT, General |
| **HR-Team** | Global | Security | HR, General |
| **Finance-Team** | Global | Security | Finance, General |
| **Management-Team** | Global | Security | Management, General |

---

## 👤 Department Users

One user was created per department for testing purposes.

```powershell
New-ADUser -Name "IT User01" -SamAccountName "ituser01" `
  -UserPrincipalName "ituser01@mylab.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome123!" -AsPlainText -Force) `
  -Enabled $true -Path "OU=VMUsers,DC=mylab,DC=local"

New-ADUser -Name "HR User01" -SamAccountName "hruser01" `
  -UserPrincipalName "hruser01@mylab.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome123!" -AsPlainText -Force) `
  -Enabled $true -Path "OU=VMUsers,DC=mylab,DC=local"

New-ADUser -Name "Finance User01" -SamAccountName "finuser01" `
  -UserPrincipalName "finuser01@mylab.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome123!" -AsPlainText -Force) `
  -Enabled $true -Path "OU=VMUsers,DC=mylab,DC=local"

New-ADUser -Name "Management User01" -SamAccountName "mgmtuser01" `
  -UserPrincipalName "mgmtuser01@mylab.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome123!" -AsPlainText -Force) `
  -Enabled $true -Path "OU=VMUsers,DC=mylab,DC=local"
```

### Group Memberships

```powershell
Add-ADGroupMember -Identity "IT-Team" -Members "ituser01","nsimon"
Add-ADGroupMember -Identity "HR-Team" -Members "hruser01"
Add-ADGroupMember -Identity "Finance-Team" -Members "finuser01"
Add-ADGroupMember -Identity "Management-Team" -Members "mgmtuser01"
```

| User | Department | Group | Password |
|---|---|---|---|
| **ituser01** | IT | IT-Team | Welcome123! |
| **hruser01** | HR | HR-Team | Welcome123! |
| **finuser01** | Finance | Finance-Team | Welcome123! |
| **mgmtuser01** | Management | Management-Team | Welcome123! |
| **nsimon** | IT Admin | IT-Team | Welcome123! |

---

## 🌐 SMB Share Configuration

The root FileServer folder was shared with Everyone — Full Control at the share level. All access control is handled through NTFS permissions.

```powershell
New-SmbShare -Name "FileServer" `
  -Path "C:\FileServer" `
  -FullAccess "Everyone" `
  -Description "Company File Server"
```

> **Why Everyone — Full Control on the share?**
> This is intentional best practice. The share permission acts as the "front door" — we open it wide. The NTFS permissions inside act as the individual "room locks" — controlling exactly who gets in where. Managing access at the NTFS level gives more flexibility and granularity.

---

## 🔒 NTFS Permissions

Each department folder was configured with:
1. **Inheritance disabled** — parent folder permissions don't flow down
2. **Existing permissions removed** — clean slate
3. **Domain Admins** — Full Control (IT admin access)
4. **Department group** — Modify access (read, write, delete)
5. **General folder** — Everyone gets Read access

```powershell
function Set-FolderPermission {
  param($FolderPath, $GroupName, $Permission)

  $acl = Get-Acl $FolderPath
  $acl.SetAccessRuleProtection($true, $false)
  $acl.Access | ForEach-Object { $acl.RemoveAccessRule($_) }

  $adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "MYLAB\Domain Admins","FullControl","ContainerInherit,ObjectInherit","None","Allow")
  $acl.AddAccessRule($adminRule)

  $groupRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "MYLAB\$GroupName",$Permission,"ContainerInherit,ObjectInherit","None","Allow")
  $acl.AddAccessRule($groupRule)

  Set-Acl $FolderPath $acl
  Write-Host "Permissions set for $FolderPath" -ForegroundColor Green
}

Set-FolderPermission -FolderPath "C:\FileServer\IT" -GroupName "IT-Team" -Permission "Modify"
Set-FolderPermission -FolderPath "C:\FileServer\HR" -GroupName "HR-Team" -Permission "Modify"
Set-FolderPermission -FolderPath "C:\FileServer\Finance" -GroupName "Finance-Team" -Permission "Modify"
Set-FolderPermission -FolderPath "C:\FileServer\Management" -GroupName "Management-Team" -Permission "Modify"
```

### Verified NTFS Permissions

```
C:\FileServer\IT
  MYLAB\Domain Admins    → FullControl
  MYLAB\IT-Team          → Modify, Synchronize

C:\FileServer\HR
  MYLAB\Domain Admins    → FullControl
  MYLAB\HR-Team          → Modify, Synchronize

C:\FileServer\Finance
  MYLAB\Domain Admins    → FullControl
  MYLAB\Finance-Team     → Modify, Synchronize

C:\FileServer\Management
  MYLAB\Domain Admins    → FullControl
  MYLAB\Management-Team  → Modify, Synchronize

C:\FileServer\General
  Everyone               → ReadAndExecute, Synchronize
  MYLAB\Domain Admins    → FullControl
```

---

## ✅ Access Verification

Permissions were verified by logging into the Windows 10 client as each user and testing folder access.

### Test as ituser01 (IT-Team member)

```powershell
$folders = @("IT", "HR", "Finance", "Management", "General")
foreach ($folder in $folders) {
    try {
        Get-ChildItem "\\192.168.1.200\FileServer\$folder" -ErrorAction Stop
        Write-Host "✅ $folder — ACCESS GRANTED" -ForegroundColor Green
    } catch {
        Write-Host "❌ $folder — ACCESS DENIED" -ForegroundColor Red
    }
}
```

**Results:**
```
✅ IT         — ACCESS GRANTED
❌ HR         — ACCESS DENIED
❌ Finance    — ACCESS DENIED
❌ Management — ACCESS DENIED
✅ General    — ACCESS GRANTED
```

### Test as hruser01 (HR-Team member)

**Results:**
```
❌ IT         — ACCESS DENIED
✅ HR         — ACCESS GRANTED
❌ Finance    — ACCESS DENIED
❌ Management — ACCESS DENIED
✅ General    — ACCESS GRANTED
```

---

## 📊 Permission Summary

| Folder | IT-Team | HR-Team | Finance-Team | Management-Team | Everyone | Domain Admins |
|---|---|---|---|---|---|---|
| **IT** | ✅ Modify | ❌ | ❌ | ❌ | ❌ | ✅ Full |
| **HR** | ❌ | ✅ Modify | ❌ | ❌ | ❌ | ✅ Full |
| **Finance** | ❌ | ❌ | ✅ Modify | ❌ | ❌ | ✅ Full |
| **Management** | ❌ | ❌ | ❌ | ✅ Modify | ❌ | ✅ Full |
| **General** | ✅ Read | ✅ Read | ✅ Read | ✅ Read | ✅ Read | ✅ Full |

---

## 📐 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       mylab.local Domain                         │
│                                                                 │
│  ┌──────────────────────────┐   ┌────────────────────────────┐  │
│  │   WinServer2022          │   │   CPC-NSimon-001           │  │
│  │   File Server            │   │   Windows 10 Pro           │  │
│  │                          │   │                            │  │
│  │   C:\FileServer\         │   │   ituser01 logs in:        │  │
│  │   ├── IT\     (IT-Team)  │◄──│   ✅ \\server\FileServer\IT│  │
│  │   ├── HR\     (HR-Team)  │   │   ❌ \\server\FileServer\HR│  │
│  │   ├── Finance\(Fin-Team) │   │                            │  │
│  │   ├── Mgmt\   (Mgmt-Team)│   │   hruser01 logs in:        │  │
│  │   └── General\(Everyone) │   │   ❌ \\server\FileServer\IT│  │
│  │                          │   │   ✅ \\server\FileServer\HR│  │
│  │   IP: 192.168.1.200      │   │   IP: 192.168.1.10         │  │
│  └──────────────────────────┘   └────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔴 Issues & Fixes

### Issue 1: SMB Share Path Prompt
| | |
|---|---|
| **Cause** | Backtick line continuation broke in PowerShell when pasting multi-line commands |
| **Fix** | Entered the path manually when prompted and ran remaining parameters separately |

### Issue 2: Inherited Permissions Still Showing
| | |
|---|---|
| **Cause** | Default Windows folders inherit permissions from parent |
| **Fix** | Used `SetAccessRuleProtection($true, $false)` to disable inheritance and remove inherited rules before applying new ones |

---

## 📚 Skills Demonstrated

- Windows Server File Server Configuration
- SMB Share Creation & Management
- NTFS Permission Configuration
- Active Directory Security Group Management
- Role-Based Access Control (RBAC)
- PowerShell Scripting for Permission Management
- Access Control List (ACL) Management
- Permission Inheritance Management
- Multi-user Access Testing & Verification
- Enterprise File Server Best Practices
- IT Documentation

---

## 🚀 Future Improvements

- [ ] Auto-map department drives via GPO logon script
- [ ] Configure folder quotas to limit storage per department
- [ ] Enable access-based enumeration — users only see folders they can access
- [ ] Set up file auditing to track who accesses what
- [ ] Configure DFS (Distributed File System) for redundancy
- [ ] Add Finance folder encryption for sensitive data

---

## 🔗 Related Projects

| Project | Description |
|---|---|
| **Part 1** | [DNS & DHCP Server Setup](https://yuzuki007.github.io/DNS-DHCP-Server-Setup-on-Windows-Server-2022/) |
| **Part 2** | [Windows 10 Client & AD Integration](https://yuzuki007.github.io/Windows-10-Client-Setup-Active-Directory-Integration/) |
| **Part 3** | Group Policy Objects (GPOs) |
| **Part 4** | File Server with NTFS Permissions ← You are here |

---

## 👤 Author

**Neil Marvin Simon**
Home Lab Project — Part 4 | Windows Server 2022 | File Server | NTFS Permissions
*Built for IT Portfolio purposes*
