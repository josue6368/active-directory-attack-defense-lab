# active-directory-attack-defense-lab
Built an Active Directory attack and defense lab with Windows Server, Windows 11, Kali Linux, and Wazuh. Simulated identity-based attacks, analyzed Windows security logs, and documented defensive monitoring, hardening steps, and MITRE ATT&amp;CK mappings.

# Active Directory Attack and Defense Lab

## Overview

This project demonstrates the creation of a small Active Directory lab environment used to monitor identity-based security events and apply basic defensive hardening.

The lab was built using Windows Server as a Domain Controller, a Windows 11 domain-joined workstation, Kali Linux for domain reconnaissance and authentication testing, and Wazuh SIEM for centralized log monitoring. The goal was to simulate common Active Directory security events, validate log collection, document defensive controls, and analyze identity-based activity from both internal workstation activity and Kali-based testing.

---

### Lab Architecture

- **Domain Controller:** Windows Server VM named `DC01`
- **Domain:** `homelab.local`
- **Domain Workstation:** Windows 11 VM named `WIN11-CLIENT`
- **Attacker/Test Machine:** Kali Linux VM
- **SIEM:** Wazuh Server on Ubuntu Server
- **Monitoring Agents:** Wazuh agents installed on `DC01` and `WIN11-CLIENT`
- **Platform:** VMware
- **Network:** Private NAT lab environment

---

### Tools and Technologies

- Windows Server
- Active Directory Domain Services
- Windows 11
- Kali Linux
- Nmap
- smbclient
- Wazuh SIEM
- Ubuntu Server
- VMware
- Group Policy Management
- Windows Event Logs

---

### Ethical Scope

This lab was conducted entirely within a private virtual environment. All users, systems, logon attempts, and account changes were created specifically for this project.

No production systems, real users, or external environments were used.

---

### Lab Setup Summary

- Installed and configured Windows Server as a Domain Controller
- Created a new Active Directory forest and domain: `homelab.local`
- Created test domain users and security groups
- Joined a Windows 11 workstation to the domain
- Installed Wazuh agents on both the Domain Controller and workstation
- Used Kali Linux to enumerate exposed Active Directory services on DC01
- Generated failed SMB authentication attempts from Kali for Wazuh detection
- Generated authentication and account management events
- Monitored Active Directory activity through Wazuh
- Implemented an account lockout policy using Group Policy

---


## Active Directory Domain Setup

#### Domain Information

```text
Domain Controller: DC01
Domain Name: homelab.local
Workstation: WIN11-CLIENT
```

The screenshots below show the initial Active Directory lab setup and validation.

**Figure 1. Server role installation on DC01**  
Server Manager showing Active Directory Domain Services (AD DS) and DNS installed on the Domain Controller.

<img width="1730" height="1241" alt="Screenshot 2026-05-04 190900" src="https://github.com/user-attachments/assets/3151cf46-e5d4-4d4e-8357-b7f8105b8dd5" />
<br />

**Figure 2. Domain Controller configuration**  
Server Manager showing DC01 joined to the `homelab.local` domain with Active Directory Domain Services (AD DS) and DNS roles available.
<br />

<img width="1718" height="1224" alt="Screenshot 2026-05-04 190941" src="https://github.com/user-attachments/assets/7cbbb88f-ae90-46c2-88af-9f55f01c75a9" />
<br />

**Figure 3. Workstation joined to domain**  
Confirmation that WIN11-CLIENT successfully joined the `homelab.local` domain.
<br />

<img width="412" height="270" alt="Screenshot 2026-05-04 214515" src="https://github.com/user-attachments/assets/38b324d4-9e34-4818-971b-de91c4262b6f" />
<br />

**Figure 4. Domain user login validation**  
Successful login by domain user `jsmith@homelab.local` on WIN11-CLIENT.

<br />
<img width="619" height="303" alt="Screenshot 2026-05-04 214749" src="https://github.com/user-attachments/assets/2ebff35e-ae21-476f-94a6-f9473c03291b" />
<br />


---

### Wazuh Agent Deployment

Wazuh agents were deployed to both the Domain Controller and the domain-joined workstation.

#### Monitored Endpoints

```
DC01
WIN11-CLIENT
```
Both systems were successfully connected to Wazuh and reported as active agents.

<br />

<img width="1896" height="1079" alt="Screenshot 2026-05-05 181137" src="https://github.com/user-attachments/assets/1854e4aa-b97b-454f-8f8d-a24241046bb1" />

---

### Kali-Based Domain Reconnaissance and Authentication Testing

Kali Linux was used to perform basic domain reconnaissance and generate failed authentication activity against the Domain Controller. This added an attacker/test machine perspective to the lab and helped validate that identity-based activity from external systems could be detected in Wazuh.

#### Domain Service Enumeration

An Nmap scan was performed against DC01 to identify exposed Active Directory-related services.

```
nmap -sV -p 53,88,135,139,389,445,464,593,636,3268,3269,3389 192.168.88.140
```
The scan identified common Active Directory services such as DNS, Kerberos, LDAP, SMB, RPC, and RDP.

<br />
<img width="1303" height="691" alt="Screenshot 2026-05-06 160020" src="https://github.com/user-attachments/assets/87df2490-ddfd-4581-a77b-3156daa71110" />

### Failed SMB Authentication Testing

Kali was also used to generate failed SMB authentication attempts against DC01 using a domain test account and an incorrect password.

```
for i in {1..8}; do smbclient -L //192.168.88.140 -U 'HOMELAB\jsmith%WrongPassword123!' -m SMB3; done
```

<img width="1301" height="428" alt="Screenshot 2026-05-06 161744" src="https://github.com/user-attachments/assets/729e86e4-d42a-4409-b0b9-dd1547516bf7" />

<br />

This generated failed authentication events on the Domain Controller, which were ingested and reviewed in Wazuh.

<br />

<img width="2306" height="1702" alt="Screenshot 2026-05-06 161753" src="https://github.com/user-attachments/assets/acc02e11-c40e-4bf2-a321-b3e1c7b52463" />

<br />

<img width="2294" height="1900" alt="Screenshot 2026-05-06 161831" src="https://github.com/user-attachments/assets/e957f825-1a38-44f5-b872-02ef58018d3e" />

---

### Active Directory Monitoring Events
#### Event 1: Failed Domain Logon Attempts

Failed domain logon attempts were generated using the test user account:

```
HOMELAB\jsmith
```
Incorrect passwords were entered from the domain-joined Windows 11 workstation and also generated from Kali Linux using SMB authentication attempts against DC01.

#### Windows Event ID

```
4625 – An account failed to log on
```
#### Evidence

Wazuh captured failed authentication events showing the attempted domain logon activity.


<img width="1898" height="1866" alt="Screenshot 2026-05-05 184449" src="https://github.com/user-attachments/assets/3d558cef-5e81-475e-98c0-1e3fe5da4bff" />

<br />

<img width="1896" height="1939" alt="Screenshot 2026-05-05 184522" src="https://github.com/user-attachments/assets/65afcc70-fa38-42dd-9773-8676ebb4b84f" />

#### Security Relevance

Repeated failed logon attempts may indicate:

* Password guessing
* Brute-force activity
* Unauthorized access attempts
* Misconfigured credentials
* Compromised account activity

#### Defensive Value

Monitoring failed logons helps identify suspicious authentication behavior and supports early detection of identity-based attacks.

---

#### Event 2: Successful Domain Logon

A successful domain logon was performed using the test domain user:
```
HOMELAB\jsmith
```
#### Windows Event ID
```
4624 – An account was successfully logged on
```
#### Evidence

Wazuh captured successful domain authentication activity from the workstation.


<img width="2485" height="1720" alt="Screenshot 2026-05-05 184902" src="https://github.com/user-attachments/assets/7fad0851-5017-4bfc-aeff-35461533c93e" />

<br />
<img width="2475" height="1721" alt="Screenshot 2026-05-05 184832" src="https://github.com/user-attachments/assets/6285b3b4-3569-4eec-bf72-f5c53bb5d688" />

#### Security Relevance

Successful logon monitoring helps analysts validate:

* User authentication activity
* Logon source systems
* Account usage patterns
* Potential unauthorized access after repeated failures

#### Defensive Value

Successful logons are important when correlated with failed logons, unusual times, unexpected workstations, or privileged accounts.

---

#### Event 3: Group Membership Added

The test user `jsmith` was added to the `IT Support` security group in Active Directory.

#### Windows Event ID

``` 
4728 – A member was added to a security-enabled global group
```

#### Evidence

Wazuh captured the group membership change from the Domain Controller.




<img width="1058" height="757" alt="Screenshot 2026-05-05 193746" src="https://github.com/user-attachments/assets/07b26939-d5e5-4ebc-b17b-1f1af7e5b951" />

<br />

<img width="2182" height="1947" alt="Screenshot 2026-05-05 194003" src="https://github.com/user-attachments/assets/dcce80d7-5223-4778-a1b5-32f31245dbb4" />

#### Security Relevance

Group membership changes are important because attackers often attempt to add accounts to privileged or useful groups after gaining access.

#### Potential risks include:

* Privilege escalation
* Unauthorized access expansion
* Persistence
* Abuse of delegated permissions

#### Defensive Value

Monitoring group changes helps detect unauthorized privilege changes and supports identity governance.

---

#### Event 4: Group Membership Removed

The test user `jsmith` was removed from the `IT Support` security group.

#### Windows Event ID

```
4729 – A member was removed from a security-enabled global group
```
#### Evidence

Wazuh captured the group membership removal event.


<img width="1027" height="716" alt="Screenshot 2026-05-05 194141" src="https://github.com/user-attachments/assets/7515d9e7-35fb-4a9d-a1d1-05a31b4e70ee" />

<br />
<img width="2181" height="1948" alt="Screenshot 2026-05-05 194152" src="https://github.com/user-attachments/assets/072967f7-cf03-4321-863d-8b192e20fd30" />


#### Security Relevance

Group membership removals can indicate:

* Administrative cleanup
* Account deprovisioning
* Privilege removal
* Suspicious changes if performed unexpectedly

#### Defensive Value

Tracking both additions and removals provides a more complete view of privilege changes in Active Directory.

---

#### Event 5: User Account Created

A temporary test account was created in Active Directory.

#### Test Account

```
tempuser
```
#### Windows Event ID

```
4720 – A user account was created
```
#### Evidence

Wazuh captured the user creation event from the Domain Controller.

<br />
<img width="981" height="702" alt="Screenshot 2026-05-05 195223" src="https://github.com/user-attachments/assets/df5a244d-2749-42e8-893d-0a25caec5c48" />

<br />
<img width="2177" height="1938" alt="Screenshot 2026-05-05 195305" src="https://github.com/user-attachments/assets/8e8f21ad-0d29-47bf-83b9-d59d0a3df54b" />


#### Security Relevance

New account creation should be monitored because unauthorized accounts may be created for persistence or future access.

Potential risks include:

* Rogue account creation
* Persistence mechanisms
* Unauthorized access
* Privilege abuse

#### Defensive Value

User creation monitoring supports account lifecycle management and detection of unauthorized identity changes.

---

#### Event 6: User Account Deleted

The temporary test account `tempuser` was deleted from Active Directory.

#### Windows Event ID 

```
4726 – A user account was deleted
```

#### Evidence

Wazuh captured the user deletion event from the Domain Controller.

<br />

<img width="2186" height="1947" alt="Screenshot 2026-05-05 195345" src="https://github.com/user-attachments/assets/55bab729-d337-406e-b09e-20f8b15936b9" />


#### Security Relevance

Account deletion events are important for monitoring administrative actions and validating user lifecycle management.

Unexpected deletions could indicate:

* Malicious cleanup activity
* Insider activity
* Misconfiguration
* Unauthorized administrative changes

#### Defensive Value

Monitoring account deletion helps support auditability and accountability in Active Directory environments.

---

### Defensive Hardening
#### Account Lockout Policy

A Group Policy Object was created and linked to the domain to reduce brute-force risk.

#### Policy Name

```
Account Lockout Policy
```
#### Configured Settings

```
Account lockout threshold: 5 invalid logon attempts
Account lockout duration: 15 minutes
Reset account lockout counter after: 15 minutes
```
<img width="995" height="722" alt="Screenshot 2026-05-05 200132" src="https://github.com/user-attachments/assets/f63213e8-b768-47dd-96db-06dc759c8564" />


#### Security Benefit

Account lockout policies help reduce the effectiveness of brute-force and password guessing attacks by temporarily locking accounts after a defined number of failed attempts.

This control helps protect domain accounts from repeated authentication attempts and supports defensive monitoring by creating clear events that can be investigated by security teams.

---

### Detection and Monitoring Summary

| Activity                | Event ID | Description                                         |
| ----------------------- | -------: | --------------------------------------------------- |
| Failed domain logon | 4625 | Failed authentication from WIN11-CLIENT and Kali SMB testing |
| Successful domain logon |     4624 | An account was successfully logged on               |
| Group member added      |     4728 | Member added to a security-enabled global group     |
| Group member removed    |     4729 | Member removed from a security-enabled global group |
| User account created    |     4720 | A user account was created                          |
| User account deleted    |     4726 | A user account was deleted                          |
| Kali AD service enumeration | N/A | Nmap scan identified exposed AD-related services on DC01 |


### MITRE ATT&CK Mapping

| Activity                  | MITRE Technique              | Description                                 |
| ------------------------- | ---------------------------- | ------------------------------------------- |
| Repeated failed logons    | T1110 – Brute Force          | Attempting to guess valid credentials       |
| Valid user logon activity | T1078 – Valid Accounts       | Use of valid domain credentials             |
| Group membership changes  | T1098 – Account Manipulation | Modifying accounts or permissions           |
| User account creation     | T1136 – Create Account       | Creating accounts for access or persistence |
| AD service enumeration | T1046 – Network Service Discovery | Scanning DC01 for exposed services |
| SMB authentication attempts | T1110 – Brute Force | Repeated failed authentication using a domain account |

---

### Analyst Takeaways

This lab demonstrated how Active Directory identity activity can be monitored using Windows security events and Wazuh SIEM.

Key takeaways include:

* Domain Controller logs provide high-value identity telemetry
* Failed and successful logons should be reviewed together
* Group membership changes can indicate privilege escalation or account manipulation
* Account creation and deletion events support identity lifecycle monitoring
* Account lockout policies help reduce brute-force risk
* Centralized log collection improves visibility across domain systems

### Skills Demonstrated

* Active Directory lab setup
* Domain Controller configuration
* Windows workstation domain join
* Wazuh agent deployment
* Windows Event Log analysis
* Kali-based Active Directory reconnaissance
* Nmap service enumeration
* SMB authentication testing
* Domain authentication monitoring
* Group membership change monitoring
* Account lifecycle monitoring
* Group Policy configuration
* Brute-force risk reduction
* MITRE ATT&CK mapping
* Security documentation

> [!NOTE]
> This lab was built for educational and portfolio purposes. All activity was generated in a controlled private environment using test systems and test accounts.


### Author
:floppy_disk: josue6368 <br/>
Cybersecurity Analyst | IT Professional






















