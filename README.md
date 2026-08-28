Active Directory Domain Services (AD DS) Setup & GPO Management

**Course:** [Independent Lab]  
**Author:** [Olakay Ladapo]  
**Date:** August 2026  

---

## Overview
This lab demonstrates the installation, configuration, and management of **Active Directory Domain Services (AD DS)** in a virtualized lab environment. The goal was to establish a functional primary Domain Controller (DC), create an enterprise Organizational Unit (OU) structure, automate user creation via PowerShell, and enforce baseline Group Policy Objects (GPOs).

---

## Execution Steps

### Phase 1: Gateway & Network Infrastructure Setup
1. **Identify DC Adapters:** Distinguish between the External (Internet-facing) and Internal (LAN) interface.
2. **Configure LAN Adapter:** Set static IPv4 address (e.g., `172.16.0.1`), leaving default gateway blank and setting Primary DNS to `127.0.0.1`.

### Phase 2: Active Directory & Privilege Elevation
4. **Install AD DS & Promote DC:** Install Active Directory Domain Services role and promote `DC` as root controller for `CORPINTERNAL.com` (NetBIOS: `CORP`).
5. **Establish Administrative OU Structure:** Open `dsa.msc` and create a dedicated administrative Organizational Unit (e.g., `ADMINS`).
6. **Provision & Elevate Admin User:** Create a primary administrative account in `ADMINS` and add it to `Domain Admins` (`Add-ADGroupMember -Identity "Domain Admins" -Members "<username>"`).
7. **Verify Admin Access:** Sign out of `DC` and log back in using the new elevated domain account.

### Phase 3: Routing, NAT, and DHCP Services
8. **Configure Remote Access & NAT:** Install the Remote Access role, launch `rrasmgmt.msc`, enable NAT, and bind it to the Internet-facing adapter so internal client traffic routes externally through `DC`.
9. **Configure & Authorize DHCP Server:** Install DHCP, create a scope (`172.16.1.100`–`172.16.1.200`), set Router (Option 003), then authorize server in AD.

### Phase 4: Automated User Population
10. **Execute User Provisioning Script:** Set execution policy (`Set-ExecutionPolicy Unrestricted -Force`) and run `1_CREATE_USERS.ps1` to bulk-generate accounts from `names.txt` into `OU=_USERS`.
11. **Verify User Accounts:** Inspect `dsa.msc` to confirm users are created and enabled.

### Phase 5: Client Staging & Domain Integration
12. **Configure Client Adapter:** Assign `CLIENT1` to the same hypervisor Internal Network switch as `DC`'s LAN interface.
13. **Validate Lease & Routing:** Run `ipconfig /all` on `CLIENT1` to confirm DHCP lease acquisition, gateway routing through `DC`, and DNS resolution.
14. **Join Client to Domain:** Rename machine and join domain, authenticating with domain admin credentials.
15. **Confirm Sign-In Capability:** Reboot client and log in using both a script-created user (e.g., `CORP\jdoe` / `Password1`) and domain admin credentials.

### Phase 6: Group Policy Deployment & Endpoint Hardening
16. **Launch Group Policy Management:** Open `gpmc.msc` on `DC` and expand `CORPINTERNAL.com` to review existing organizational units and GPO links.
17. **Create Custom Hardening GPO:** Right-click **Group Policy Objects** $\rightarrow$ select **New** $\rightarrow$ name the policy (e.g., `GPO_Workstation_Hardening`).
18. **Configure Security Policies:** Edit the new GPO (`Right-Click` $\rightarrow$ **Edit**) to define enterprise security baselines:
    * **Account Lockout Policy:** Set threshold to 3 invalid attempts within a 15-minute window (`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Account Lockout Policy`).
    * **Remote Desktop Access:** Enable RDP services (`Computer Configuration -> Administrative Templates -> Windows Components -> Remote Desktop Services`).
    * **Windows Firewall Baselines:** Domain and Private profile rules enabled (`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Firewall with Advanced Security`).
19. **Link GPO to Target OU:** Drag and link `GPO_Workstation_Hardening` directly to the `USERS` or workstation OU containing `CLIENT`.
20. **Force Policy Processing on Client:** Open PowerShell as Administrator on `WIN11-CL01` and execute `gpupdate /force` to pull the latest GPO settings from `DC01`.
21. **Verify Policy Enforcement:** Generate an RSoP report on `CLIENT` using `gpresult /scope computer /r` to confirm `GPO_Workstation_Hardening` is listed under **Applied Group Policy Objects**.
   
---

## Environment & Topology

* **Hypervisor:** VirtualBox
* **Domain Controller:** Windows Server 2022 (Hostname: `DC`, Static IP: `172.16.0.1/24`)
* **Client Workstation:** Windows 10 Pro (Hostname: `CLIENT1`, Joined to Domain)
* **Domain Name:** `CORPINTERNAL.com`
* **Subnet:** `172.16.0.0/24` (NAT)

### Network Topology Diagram
```text
[ Windows 11 Client ] <---> [ Internal Switch ] <---> [ Domain Controller (DC01) ]
  IP: 172.16.0.100                                     IP: 172.16.0.1
                                                        AD DS, DHCP Roles, GPO
