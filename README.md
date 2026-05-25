# 🌐 Project Overview
This project simulates a production-grade enterprise identity management environment. By deploying Windows Server 2025, I configured a centralized Active Directory domain to manage users, computers, and security policies. This lab serves as a foundational demonstration of my ability to implement Role-Based Access Control (RBAC), enforce security hardening baselines, and perform system administration—all essential skills for a Junior System Administrator or SOC Analyst.

## 🏗️ Lab Architecture & Scope
* **Operating System:** Windows Server 2025 Datacenter (Gen2 Evaluation Platform)
* **Deployment Vector:** Cloud Virtual Machine (Microsoft Azure Standard_B2s Instance)
* **Target Core Services:** Active Directory Domain Services (AD DS), DNS, Group Policy Management Console (GPMC)
* **Private Network Context:** Isolated VNet (`10.0.0.0/16` boundary), static IP mapping

---

## 🚀 Step-by-Step Implementation Guide

### Step 1: Network Foundation & IP Reservation
To guarantee directory services stability, the server requires a persistent IP address. Rather than relying on standard dynamic DHCP, I leveraged Azure's platform-level IP reservation to ensure the Domain Controller maintains a fixed internal address (10.0.0.4) throughout its lifecycle.

Provisioned a Windows Server 2025 Datacenter instance within an isolated Azure Virtual Network.

Configured the Azure Virtual Network Interface (NIC) to assign a Static Private IP address to the Domain Controller.

Verified local DNS resolution by ensuring the server correctly identified its own network loopback address as the primary resolver for directory services.

<img width="1000" alt="image" src="https://github.com/DavidPatrick92/Active_Directory/blob/main/screenshots/Screenshot%202026-05-24%20220459.png">
---

### Step 2: Active Directory Domain Services Installation
The installation phase deploys the core payload binaries and administrative tools, preparing the machine for its system lifecycle transformation into a Domain Controller.

1. Initiated the Server Manager interface inside the instance.
2. Navigated through the **Add Roles and Features Wizard** to install the **Active Directory Domain Services** payload binaries along with relevant Remote Server Administration Tools (RSAT).
3. Concurrent to AD DS installation, initialized the standalone Group Policy Management Console (GPMC) package to handle domain-wide security boundaries.

```powershell
# Administrative PowerShell Pipeline: Role Payload Installation
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-WindowsFeature -Name GPMC
```

### Step 3: Domain Controller Forest Promotion
Installing binaries does not automatically create a domain environment. This phase builds a brand-new directory framework, declaring this asset the roots of identity orchestration.

Triggered the Deployment Configuration post-install warning thread within Server Manager.

Formulated a brand new directory forest designated explicitly under internal roots domain constraints: lab.local.

Established a secure Directory Services Restore Mode (DSRM) disaster-recovery token password.

Execution triggered an active system compilation and reboot phase, turning the unit into the root Domain Controller.

```powershell
# Administrative PowerShell Pipeline: Automated Forest Inception
Import-Module ADDSDeployment
Install-ADDSForest `
    -DomainName 'lab.local' `
    -DomainNetBiosName 'LAB' `
    -InstallDns:$true `
    -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourSecureDSRM123!' -AsPlainText -Force) `
    -Force:$true
```
<img width="1000" alt="image" src="https://github.com/DavidPatrick92/Active_Directory/blob/main/screenshots/Screenshot%202026-05-23%20233822.png">.

### Step 4: Enterprise Hierarchy (OUs, Security Groups, & Users)
To simulate practical operations, this layer designs an explicit role-based access model across standard operational department vectors (IT, Finance, HR, Sales).

Opened the Active Directory Users and Computers (ADUC) panel.

Implemented hierarchical separation by creating separate Organizational Units (OUs) to serve as administrative containment targets for users and devices.

Created discrete departmental Security Groups to facilitate clean, scalable role-based access control (RBAC).

Auto-provisioned target user personas leveraging enterprise name synchronization boundaries (firstname.lastname), appending them directly to matching security wrappers.

``` powershell
# Administrative PowerShell Pipeline: Directory Structure & Account Provisioning
# 1. Instantiate Organizational Units
New-ADOrganizationalUnit -Name "IT" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```
``` powershell
# 2. Instantiate Security Groups (RBAC Wrappers)
New-ADGroup -Name "IT_Admins" -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users" -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users" -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```
```powershell
# 3. Provision Sample User Accounts & Assign Group Memberships
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true
New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true
New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true
New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

Add-ADGroupMember -Identity "IT_Admins" -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users" -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users" -Members "david.smith"
```
<img width="1000" alt="image" src="screenshots/Screenshot 2026-05-23 234700.png">

### Step 5: Enforcing Enterprise Security Policies via GPO
Group Policy Object (GPO) integration automates corporate baseline hardening across endpoints and users instantly, neutralizing physical exfiltration paths and credential threats globally.

Initiated the Group Policy Management Console (GPMC) interface.

Engineered a targeted container policy artifact named: IT Security Policy, linking it directly onto the managed IT department OU block.

Opened the GPO editor to define specific system settings matching rigid identity hygiene benchmarks:

### Security Baseline Policy Configuration
The following table outlines the technical security controls enforced across the IT department Organizational Unit via Group Policy Objects (GPOs):

| Policy Path Location | Target Security Setting | Enforced Baseline Value |
| :--- | :--- | :--- |
| `Computer Config -> Windows Settings -> Security -> Account Policies -> Password Policy` | Minimum password length | **12 Characters** |
| `Computer Config -> Windows Settings -> Security -> Account Policies -> Password Policy` | Password must meet complexity requirements | **Enabled** |
| `Computer Config -> Windows Settings -> Security -> Local Policies -> Security Options` | Interactive logon: Machine inactivity limit | **900 Seconds (15 Min)** |
| `Computer Config -> Administrative Templates -> System -> Removable Storage Access` | All removable storage classes: Deny all access | **Enabled (Blocks USB Exfiltration)** |

<img width="500" alt="image" src="screenshots/Screenshot 2026-05-24 000114.png">

<img width="500" alt="image" src="screenshots/Screenshot 2026-05-24 000304.png">

<img width="500" alt="image" src="screenshots/Screenshot 2026-05-24 000426.png">

<img width="500" alt="image" src="screenshots/Screenshot 2026-05-24 000704.png">

## 🧪 Validating Security Policies: GPO Testing
After linking the IT Security Policy to the IT Organizational Unit, I verified the policy's efficacy by deploying a domain-joined client VM. This ensured that GPO inheritance and security hardening were correctly functioning within the directory hierarchy.

### Step 1: Client VM Configuration
To facilitate seamless domain integration, the client VM required specific network adjustments:

Static DNS Assignment: Configured the client's Primary DNS server to match the Domain Controller's IP (10.0.0.4), ensuring authoritative resolution of the lab.local domain.

Domain Integration: Joined the VM to the lab.local forest and relocated its computer object into the IT OU to bring it within the GPO's scope of influence.

### Step 2: Policy Enforcement & Verification
I performed a programmatic policy refresh and validated the results using administrative audit tools:

```powershell
# Force GPO application from the client-side
gpupdate /force

# Validate applied policies
gpresult /r
```
### Step 3: Functional Verification (Alice.chen)
I authenticated as the domain user alice.chen to confirm the enforcement of the following security baselines:

Password Hardening: Verified that the 12-character minimum length and complexity requirements were enforced during credential management.

Inactivity Lock: Confirmed that the system automatically locked the session after the defined 15-minute inactivity threshold.


### Step 6: Operational Identity Management & Administrative Auditing
This phase outlines daily helpdesk and administrator tasks commonly executed on an enterprise network to maintain directory hygiene.

### Core Help Desk Execution Scenarios:

``` powershell
# Scenario A: Perform Password Reset & Enforce Change at Next Login Cycle
Set-ADAccountPassword -Identity "bob.patel" -Reset -NewPassword (ConvertTo-SecureString "NewTempPass2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```
```
# Scenario B: Remediate Account Lockout Event Following Brute-Force Triage
Unlock-ADAccount -Identity "carol.jones"
```
```
# Scenario C: Disable Account Lifecycle Profile During Offboarding Phase
Disable-ADAccount -Identity "david.smith"
```
### Directory Compliance Auditing Queries:

``` powershell
# Compliance Audit: Discover Stale/Inactive Identities (No Authentication Event For 90 Days)
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Access Verification Audit: Validate Nested Token Memberships For Elevated Accounts
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```
### 🔍 Validation Framework (Ensuring Lab Stability)
To verify everything is configured correctly, run these programmatic assertions in an administrative PowerShell console:

| Check | PowerShell Command | Expected Result |
| :--- | :--- | :--- |
| **Domain Controller** | `Get-ADDomainController` | Returns DC info including forest `lab.local` |
| **Organizational Units** | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs you created |
| **User Accounts** | `Get-ADUser -Filter {Enabled -eq $true}` | Lists your 4 test accounts |
| **Group Memberships** | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| **GPO Linkage** | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

```powershell
# 1. Validate Core Domain Controller Status & Registration
Get-ADDomainController | Select-Object Name, Forest, OperatingSystem

# 2. Assert Organization Units Structure Generation
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# 3. Assert Active Operational Accounts Status
Get-ADUser -Filter {Enabled -eq $true} | Select-Object SamAccountName, Name

# 4. Assert Group Policy Linkage and Target OU Inheritance
Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'
```
## 🛠️ Troubleshooting & Known Issues
During the deployment of the domain-joined client VM, I encountered a specific Remote Desktop Authorization Error when attempting to log in with a domain-level identity (alice.chen). Even with valid credentials, the client refused the connection.

Issue: "User not authorized for remote login"
After joining the client to the lab.local domain, domain users were unable to establish an RDP session, despite having valid credentials. This is because Windows security policies default to denying remote access to domain users on newly joined machines.

The Ultimate Remediation
To resolve this, I manually updated the Remote Desktop User Rights Assignment on the client VM:

Granting RDP Access Rights:

Logged in to the client VM using the local administrator account.

Accessed the Local Security Policy editor by running secpol.msc.

Navigated to: Local Policies > User Rights Assignment.

Modified the policy: "Allow log on through Remote Desktop Services".

Added the specific domain user (alice.chen) and/or the Domain Users group to the authorized list.

Verifying System Properties:

Confirmed the change by running sysdm.cpl and navigating to the Remote tab.

Verified that the user was explicitly added to the "Select Users..." list within the Remote Desktop settings to ensure the GUI and the security policy were synchronized.

<img width="500" alt="image" src="screenshots/Screenshot 2026-05-24 014021.png">

## 🎯 Key Takeaways & Portfolio Summaries
Centralized Identity Lifecycles: Managed user provisioning, updates, temporary account freezes, and termination phases down to a single identity source. This minimizes management overhead and closes every enterprise access path simultaneously during account offboarding.

Role-Based Access Control (RBAC): Applied the core security concept of Least Privilege. Users were systematically added to distinct departmental wrappers rather than being granted permissions manually. This prevents privilege creep and ensures a cleaner administrative attack surface.

Scalable Architecture Governance: Leveraged Organization Units matched with Group Policy Objects (GPOs) to enforce strict technical controls (such as blocking data exfiltration via USB drives and establishing automated idle lock timers) instantly across thousands of systems from a centralized console.

Administrative Engineering Readiness: Built practical experience managing common, real-world ticketing events (such as locked profiles and password compliance management) alongside scripting custom directory compliance reports using PowerShell pipelines.

