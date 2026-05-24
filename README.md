🌐 Project Overview
This project simulates a production-grade enterprise identity management environment. By deploying Windows Server 2025, I configured a centralized Active Directory domain to manage users, computers, and security policies. This lab serves as a foundational demonstration of my ability to implement Role-Based Access Control (RBAC), enforce security hardening baselines, and perform system administration—all essential skills for a Junior System Administrator or SOC Analyst.

🛠️ Infrastructure & Environment
Platform: Microsoft Azure (Standard_B2s Virtual Machine)

OS: Windows Server 2025 Datacenter

Core Services: Active Directory Domain Services (AD DS), DNS, GPMC

🚀 Lab Implementation Steps
1. Network Foundation & DNS
Configured a static IP environment and pointed the DNS server to the local loopback (127.0.0.1) to ensure the domain controller correctly manages its own directory resolution.

📸 [Screenshot Request]: Capture the IPv4 network properties showing your static IP and local DNS configuration.

2. AD DS & Forest Promotion
Installed the Active Directory role and promoted the server to a Domain Controller for the lab.local forest.

# Role Installation
Install-WindowsFeature -Name AD-Domain-Services, GPMC -IncludeManagementTools

📸 [Screenshot Request]: Capture the Server Manager dashboard showing AD DS and DNS roles as "Installed."

3. Identity Architecture (OUs & RBAC)Organized users into specific Organizational Units (OUs) to enforce the Principle of Least Privilege and streamline administration.

PowerShell
Create Organizational Units
$ous = "IT", "Finance", "HR", "Sales"
foreach ($ou in $ous) { New-ADOrganizationalUnit -Name $ou -Path "DC=lab,DC=local" }

📸 [Screenshot Request]: Capture the "Active Directory Users and Computers" (ADUC) console showing your organized OU structure.

4. Security Hardening via Group PolicyImplemented GPOs to automate security controls across the domain, such as password complexity and disabling USB mass storage.

📸 [Screenshot Request]: Capture the "Settings" tab in the Group Policy Management Console (GPMC) for one of your custom policies.

🔍 Troubleshooting & VerificationIn a real-world environment, issues are inevitable. Here is how I validated the integrity of my deployment:
Issue Encountered Resolution Strategy
GPO not applyingRun gpupdate /force on client; verify linkage in GPMC.
User login failure  Confirm account is Enabled and ChangePasswordAtLogon is set.
Remote Desktop (RDP) issue Ensure you are logging in as LAB\Administrator (domain admin), not local admin.

🎯 Key Takeaways
Identity Management: Mastered the creation of OUs, security groups, and automated user provisioning to streamline onboarding/offboarding.

Security Hardening: Gained experience applying GPOs to enforce enterprise-level password policies and system restrictions, effectively hardening the endpoint attack surface.

Troubleshooting Proficiency: Developed a systematic approach to diagnosing authentication and policy application issues—a critical skill for incident response and helpdesk support.
