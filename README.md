Active Directory Domain Services (AD DS) Setup & GPO Management

**Course:** [Independent Lab]  
**Author:** [Olakay Ladapo]  
**Date:** August 2026  

---

## Overview
This lab demonstrates the installation, configuration, and management of **Active Directory Domain Services (AD DS)** in a virtualized lab environment. The goal was to establish a functional primary Domain Controller (DC), create an enterprise Organizational Unit (OU) structure, automate user creation via PowerShell, and enforce baseline Group Policy Objects (GPOs).

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
