📖 Overview

This project demonstrates the deployment of Native Windows LAPS (Local Administrator Password Solution) within a custom Windows Server 2022 Active Directory environment. The objective of this project is to harden Tier 2 endpoints against Pass-the-Hash attacks and prevent lateral movement by ensuring every workstation has a unique, automatically rotating, and highly complex local administrator password securely backed up to Active Directory.

🏗️ Lab Architecture

    Hypervisor: Hyper-V

    Domain Controllers: Windows Server 2022 (DC-01)

    Endpoints: Windows 11 Pro workstation located in a Tier 2 - Endpoints Organizational Unit (OU).

    Access Control: Strict Tiered Administrative Model implemented. Tier 0 accounts (Domain Admins) are restricted from accessing Tier 2 workstations, necessitating dedicated Tier 2 administrative accounts (e.g., admin.t2) for endpoint management.

⚙️ Deployment Lifecycle
1. Active Directory Schema Expansion

To support modern Native LAPS, the Active Directory schema required expansion to house the new password and expiration timestamp attributes.

    Execution: Temporarily elevated a dedicated administrative account to Schema Admins and Enterprise Admins.

    Command: Update-LapsADSchema was executed via an elevated PowerShell session to inject the required attributes into the domain forest blueprint.

2. Delegating Access Rights

Workstations require explicit permissions to write their newly generated passwords to their own computer objects in Active Directory.

    Execution: Located the exact Distinguished Name (DN) of the target OU.

    Command: Executed Set-LapsADComputerSelfPermission -Identity "OU=Workstations,OU=Tier 2 - Endpoints,DC=lab,DC=local" to grant self-write permissions securely.

3. Group Policy Object (GPO) Configuration

A new GPO named Windows LAPS Configuration was created and linked directly to the Tier 2 Endpoints OU.

    Configuration: Enabled Configure password backup directory and pointed it to Active Directory, effectively activating the modern Native LAPS engine.

    Password Complexity: Enforced a 14-character minimum utilizing large letters, small letters, numbers, and special characters, with an automated 30-day rotation cycle.

4. Client-Side Execution & Tier Validation

    Logged into the Windows 11 endpoint to force the policy update.

    Security Validation: Attempted to authorize the policy update using a Tier 0 Domain Admin credential. The tiered architecture successfully blocked the execution, validating that highly privileged credentials cannot be cached in Tier 2 workstation memory.

    Execution: Successfully bypassed the restriction using the dedicated admin.t2 Tier 2 account and executed Invoke-LapsPolicyProcessing to immediately generate and securely transmit the new randomized password to the Domain Controller.

🛡️ Strategic Impact (Blue Team Perspective)

Deploying LAPS is a fundamental defense mechanism against lateral movement. In an environment without LAPS, IT departments often deploy a single, identical local administrator password across all endpoints. If an adversary compromises a single workstation and dumps the local SAM database, they can utilize that shared credential to laterally pivot across the entire corporate network. LAPS effectively neutralizes this attack vector by isolating local administrative credentials.
