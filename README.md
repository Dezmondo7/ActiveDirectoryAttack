🛡️ Active Directory Attack & Defense Lab
Environment: Windows Server 2022 | Windows 10 | Kali Linux
This laboratory project documents the end-to-end deployment of a Windows Domain environment, focusing on network orchestration, identity management, and offensive service enumeration.

<details>
<summary><b>01. OS Environment & ICMP Troubleshooting</b></summary>

The Objective
Establish a base-layer virtual network using VMWare Workstation.

Setup: Deployed Windows Server 2022 (Domain Controller), Windows 10 (Client), and Kali Linux (Attacker).

Connectivity Issue: Initial pings between Kali and Windows 10 timed out due to default security postures.

The Fix: Manually enabled the File and Printer Sharing (Echo Request - ICMPv4-In) rule within the Windows Defender Firewall.

Validation: Successfully verified the communication plane using ipconfig (Windows) and ip a (Kali), followed by a successful ICMP handshake.
<img width="800" alt="ENABLE Rule Windows 10 x64 - VMware Workstation" src="https://github.com/user-attachments/assets/e5202d43-fadb-4e3d-aac0-82c81384b728" />

</details>

<details>
<summary><b>02. Domain Controller & DNS Orchestration</b></summary>

Network Configuration
Renamed the server to DC01 and configured a Static IPv4 Architecture.

Static IP Rationale: Critical for the "Fixed Identity" of the Domain Controller. Services like Kerberos (88), LDAP (389), and DNS (53) require a predictable address for authentication authority.

DNS Logic: Configured the Preferred DNS to point to itself (127.0.0.1). This ensures the DC can register and resolve its own SRV records (_ldap._tcp & _kerberos._tcp), which are the backbone of Active Directory discovery.

Validation: Confirmed functionality via nslookup lab.local and ping lab.local.
<img width="800" alt="Domain_Joined" src="https://github.com/user-attachments/assets/b4b87e20-62ba-4031-84a7-4ea8bb6a8f43" />

</details>

<details>
<summary><b>03. AD Organizational Units (OU) & Structure</b></summary>

Enterprise Simulation
I implemented a logical hierarchy within Active Directory Users and Computers (ADUC).

The Build: Created an OU named "User" under the lab.local domain.

Why OUs? This mimics real-world enterprise environments, allowing for Group Policy Object (GPO) targeting and administrative delegation. It provides a structured boundary for security policies.
<img width="800" alt="user" src="https://github.com/user-attachments/assets/aa33c05e-fb04-42c0-864e-961f1178cd1d" />

</details>

<details>
<summary><b>04. Identity Seeding & Weak Password Simulation</b></summary>

The Vulnerability
To prepare for offensive testing, I provisioned user accounts with a "Security-Weak" configuration.

Configuration: Unchecked "User must change password at next logon" and assigned a weak password.

Purpose: This creates a predictable target for future brute-force or password-spraying simulations within the lab environment.
<img width="800" alt="john-loggin" src="https://github.com/user-attachments/assets/2c236367-f8d3-4d12-85bf-ce6fdcdbeac3" />

</details>

<details>
<summary><b>05. Joining the Domain: Establishing Trust</b></summary>

The Machine Identity
Transitioned the Windows 10 machine from a standalone Workgroup to the lab.local domain.

Under the Hood: A Computer Account (WIN10$) was created in AD. The client and DC now share a machine password and a secure Kerberos channel.

Result: Authentication shifted from the local Security Accounts Manager (SAM) to the centralized Domain Controller. This establishes the machine as a trusted entity within the domain security boundary.
<img width="800" alt="Domain_Joined" src="https://github.com/user-attachments/assets/0c248d1a-4bcf-41a9-a3a0-58df3b3679c4" />

</details>

<details>
<summary><b>06. Offensive Reconnaissance: Service Enumeration</b></summary>

Attack Phase 1: Nmap Discovery
Using Kali Linux, I performed a "Black Box" scan to map the DC’s attack surface.

Command: sudo nmap -sS -T4 -Pn [DC_IP]

Key Findings:

Port 88 (Kerberos): Potential for ticket-based attacks (AS-REP Roasting).

Port 389/636 (LDAP): Entry point for user/group enumeration.

Port 445 (SMB): Target for file share enumeration and relay attacks.
<img width="800" alt="reconissance1" src="https://github.com/user-attachments/assets/720d0428-e780-4d45-b317-29d7c14f4d56" />

</details>

<details>
<summary><b>07. Exploitation: Anonymous SMB Enumeration</b></summary>

Attack Phase 2: Null Sessions
Tested the SMB service for unauthenticated access.

Command: smbclient -L //[DC_IP] -N

The Result: The DC accepted the Null Session (Anonymous login).

Security Impact: This vulnerability allows an unauthenticated attacker to view available shares and gather directory information without a single credential, representing a significant risk to data confidentiality.
<img width="800" alt="attacksmbclient" src="https://github.com/user-attachments/assets/4829164a-e66c-479b-aece-3abf41f195dd" />

</details>
