# Day 7 — Active Directory Fundamentals

## 🎯 Objective

Day 7 focuses on Active Directory Domain Services (AD DS) from a SOC analyst and Windows security perspective.

Active Directory is the identity and access-management foundation used in many enterprise Windows environments. It centrally manages users, computers, groups, authentication, authorization, Group Policy, and administrative structure.

For a SOC analyst, AD is critical because enterprise attacks frequently involve compromised accounts, privilege escalation, credential theft, lateral movement, abnormal authentication, and unauthorized directory or Group Policy changes.

---

## 1. What Is Active Directory?

Active Directory Domain Services is Microsoft's directory service for Windows domain environments.

It provides centralized management of:

- Users
- Computers
- Groups
- Organizational Units
- Authentication
- Authorization
- Group Policy
- Domain resources
- Security principals

A simplified enterprise flow is:

    User
      ↓
    Domain Account
      ↓
    Domain Controller
      ↓
    Authentication
      ↓
    Authorization
      ↓
    Resource Access

### SOC relevance

When an alert involves an identity, the analyst may need to determine which account was involved, which host authenticated it, which domain controller processed the request, what authentication mechanism was used, whether privileges were involved, and whether the account subsequently accessed other systems.

---

## 2. Domain and Domain Controller

An Active Directory domain is a logical security and administrative boundary containing users, computers, groups, policies, and other directory objects.

A Domain Controller (DC) is a Windows Server running AD DS. It provides services including authentication, directory queries, Kerberos, LDAP, replication, and Group Policy distribution.

Domain controllers are high-value assets from a defensive perspective.

Important telemetry includes:

- Failed authentication
- Successful unusual authentication
- Privileged account use
- Account creation and modification
- Group membership changes
- Kerberos activity
- LDAP activity
- Group Policy changes

---

## 3. Active Directory Objects

Common AD objects include:

- User
- Computer
- Group
- Organizational Unit
- Contact
- Other directory objects

A user object can contain information such as username, UPN, SID, group memberships, account status, and logon-related attributes.

Changes to these objects can become security-relevant events.

Example:

    Existing User
         ↓
    Added to Privileged Group
         ↓
    Increased Access
         ↓
    Potential Security Incident

The membership change must still be validated against authorized administrative activity.

---

## 4. Organizational Units

An Organizational Unit (OU) is a logical container used to organize users and computers.

Example:

    corp.example.local
    ├── Users
    ├── Computers
    ├── Sales
    ├── HR
    ├── IT
    └── Servers

OUs are important because Group Policy can be linked to them and administrative delegation can be applied at the OU level.

### Practical lab

A test OU such as Sales can contain test users and computers. A security-focused GPO can then be linked to that OU to observe how policy scope works.

---

## 5. Users, Computers and SIDs

A domain user represents an identity in the domain. A computer account represents a domain-joined machine.

Windows uses Security Identifiers (SIDs) to uniquely identify security principals.

A SID commonly has a structure similar to:

    S-1-5-21-<domain-identifier>-<RID>

The SID is useful during investigations because security events frequently contain identity information in addition to the visible account name.

A SOC analyst should be able to correlate:

    SID
     ↓
    Account
     ↓
    Groups
     ↓
    Privileges
     ↓
    Activity

---

## 6. Groups and Privileged Access

Groups simplify permission and privilege management.

Common groups include:

- Domain Users
- Domain Admins
- Enterprise Admins
- Administrators
- Department-specific groups

A user's effective access depends heavily on group membership.

A particularly important SOC scenario is:

    Normal User
        ↓
    Added to Privileged Group
        ↓
    Elevated Access
        ↓
    Sensitive Resource Access

Unexpected changes to privileged group membership deserve investigation.

Questions for an analyst include:

1. Which account was changed?
2. Who made the change?
3. When did it happen?
4. From which host?
5. Was there an approved administrative reason?
6. What activity followed the change?

---

## 7. Kerberos Authentication

Kerberos is the primary authentication protocol used by modern Active Directory environments.

It uses a ticket-based model.

Simplified flow:

    User
      ↓
    Authentication
      ↓
    KDC
      ↓
    Ticket Granting Ticket
      ↓
    Service Ticket
      ↓
    Requested Service

The Key Distribution Center operates on domain controllers.

### SOC relevance

Kerberos telemetry can help analysts investigate:

- Unusual authentication
- TGT requests
- Service-ticket activity
- Service account usage
- Cross-host authentication
- Privileged authentication

Kerberos attacks such as Kerberoasting and Golden Ticket abuse will be covered later.

---

## 8. NTLM

NTLM is another Windows authentication mechanism. Although Kerberos is preferred in many domain scenarios, NTLM can still appear for compatibility and specific authentication situations.

The presence of NTLM is not automatically malicious.

An analyst should correlate:

    Source Host
       ↓
    User
       ↓
    Authentication Protocol
       ↓
    Destination
       ↓
    Resource Access

Context determines whether the authentication is unusual.

---

## 9. LDAP and Directory Discovery

Lightweight Directory Access Protocol (LDAP) is used to interact with directory information.

Active Directory uses LDAP for directory operations such as querying and modifying objects.

Directory information can reveal:

- Users
- Groups
- Computers
- Organizational structure
- Domain configuration

This makes directory discovery useful to both administrators and attackers.

A SOC analyst should consider unusual enumeration alongside other evidence rather than treating every LDAP query as malicious.

---

## 10. DNS and Active Directory

DNS is fundamental to Active Directory.

Clients use DNS to locate domain services and domain controllers.

A simplified relationship is:

    Client
      ↓
    DNS Query
      ↓
    Domain Service
      ↓
    Authentication / Directory Service

Useful troubleshooting commands include:

    ipconfig /all
    nslookup
    nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>

Unexpected DNS activity can also provide security-investigation context.

---

## 11. Group Policy

Group Policy provides centralized configuration and security management for Windows systems.

Administrators can manage:

- Security settings
- Password policies
- Windows Defender configuration
- Firewall settings
- Software configuration
- User restrictions
- Administrative templates
- Scripts
- System behavior

A Group Policy Object (GPO) can be linked to a site, domain, or OU.

---

## 12. GPO Processing and Scope

A simplified policy structure is:

    Local Policy
         ↓
    Site
         ↓
    Domain
         ↓
    Organizational Unit

Actual effective policy can also depend on:

- Link order
- Security filtering
- Inheritance
- Enforced settings
- WMI filtering

Therefore, the presence of a GPO does not automatically mean every target system receives every setting.

### SOC relevance

Unauthorized modification of a GPO can have a large blast radius because one policy can affect many users or computers.

---

## 13. Security Filtering and WMI Filtering

Security filtering controls which users or computers are allowed to apply a GPO.

WMI filtering provides an additional conditional mechanism.

Conceptually:

    GPO
      ↓
    WMI Query
      ↓
    Condition True?
      ├── Yes → Apply
      └── No  → Do Not Apply

This is important when troubleshooting or investigating why a policy is not effective.

For example, a GPO linked to an OU can still fail to apply if its WMI filter evaluates to false on the target system.

---

## 14. Blocked Inheritance and Enforced GPOs

Blocked Inheritance can prevent policies from higher levels from being inherited by an OU.

An Enforced GPO can retain precedence in situations where normal inheritance would otherwise allow another policy to override it.

When investigating policy behavior, examine the complete chain:

    GPO Link
       ↓
    Link Order
       ↓
    Inheritance
       ↓
    Security Filtering
       ↓
    WMI Filtering
       ↓
    Effective Policy

This is more reliable than assuming that a linked GPO is automatically active.

---

## 15. Domain Authentication

A simplified domain logon looks like:

    User Credentials
          ↓
    Windows Client
          ↓
    Domain Controller
          ↓
    Authentication
          ↓
    Security Context
          ↓
    Resource Access

For investigations, important fields include:

- Username
- SID
- Source workstation
- Destination system
- Logon type
- Authentication package
- Timestamp
- Result
- Privilege information

---

## 16. Windows Logon Types

Common Windows logon types include:

| Logon Type | General Meaning |
|---|---|
| 2 | Interactive |
| 3 | Network |
| 4 | Batch |
| 5 | Service |
| 7 | Unlock |
| 8 | NetworkCleartext |
| 9 | NewCredentials |
| 10 | RemoteInteractive, commonly RDP |
| 11 | CachedInteractive |

These values should always be interpreted using the complete event context.

For example, a Type 10 logon is not automatically suspicious. The analyst should consider whether the source, user, destination, time, and sequence of activity are expected.

---

## 17. Active Directory and Lateral Movement

Attackers may use compromised credentials to move from one endpoint to another.

A simplified attack sequence is:

    Initial Compromise
          ↓
    Credential Access
          ↓
    Valid Account
          ↓
    Authentication to Another Host
          ↓
    Remote Access / Execution
          ↓
    Lateral Movement

Common Windows mechanisms include:

- RDP
- SMB
- WinRM
- Remote services
- Remote administration tools

The key SOC question is whether the authentication fits the user's normal source, destination, role, time, and behavior.

---

## 18. Important Security Event IDs

Common Windows Security events relevant to AD investigations include:

| Event ID | General Context |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Logoff |
| 4648 | Explicit credentials used |
| 4672 | Special privileges assigned to new logon |
| 4688 | Process creation |
| 4720 | User account created |
| 4722 | User account enabled |
| 4723 | Password change attempt |
| 4724 | Password reset attempt |
| 4725 | User account disabled |
| 4726 | User account deleted |
| 4732 | Member added to local security-enabled group |
| 4738 | User account changed |
| 4740 | Account locked out |
| 4768 | Kerberos authentication ticket requested |
| 4769 | Kerberos service ticket requested |
| 4771 | Kerberos pre-authentication failed |
| 4776 | Domain Controller credential validation |

These events must be correlated with timestamps, account, source host, destination host, and surrounding activity.

---

## 19. Practical Active Directory Lab

The following activities were performed in an authorized Windows AD lab.

### Lab 1 — Explore the Domain

Inspect:

- Domain
- Domain Controller
- Users
- Computers
- Groups
- Organizational Units

Understand how the objects relate to each other.

### Lab 2 — Create an Organizational Unit

Create a test OU such as:

    Sales

Place authorized test users or computers inside it.

### Lab 3 — Create a Test User

Create a test domain account:

    Username: Dj

Inspect:

- Account properties
- Group membership
- Account status
- SID
- Logon configuration

### Lab 4 — Create a GPO

Create a GPO linked to the test OU.

Example:

    Sales OU
       ↓
    Sales Security GPO
       ↓
    Control Panel restriction

The objective is to understand how GPO scope and OU placement work.

### Lab 5 — Test GPO Application

Use:

    gpupdate /force

Then inspect policy results:

    gpresult /r
    gpresult /h gpresult.html

This demonstrates the difference between a configured policy and an actually applied policy.

### Lab 6 — Understand WMI Filtering

Create or inspect a WMI filter based on operating-system properties.

Test what happens when the query evaluates to true versus false.

### Lab 7 — Investigate Authentication

Generate normal authentication activity in the lab and inspect the resulting Security events.

Focus on:

- Account
- Logon type
- Source
- Destination
- Authentication package
- Timestamp
- Result

### Lab 8 — Investigate Account Changes

Create or modify a test account and identify the resulting Windows Security events.

The goal is to connect:

    Administrative Action
          ↓
    Directory Change
          ↓
    Security Event
          ↓
    SOC Investigation

---

## 20. SOC Investigation Scenario

Suppose an alert reports that a privileged account authenticated to a workstation from an unusual source.

A structured investigation would be:

### Step 1 — Identify the identity

Determine:

- Username
- SID
- Group membership
- Privilege level

### Step 2 — Identify source and destination

Determine:

- Source IP
- Source hostname
- Destination host
- Whether the path is expected

### Step 3 — Identify authentication

Review:

- Logon type
- Authentication package
- Kerberos or NTLM context
- Timestamp

### Step 4 — Correlate endpoint activity

Look for:

- Process creation
- PowerShell
- CMD
- Remote administration
- Suspicious executable activity

### Step 5 — Check privilege activity

Look for:

- Privileged logons
- Group membership changes
- Service creation
- Scheduled tasks
- GPO modifications

### Step 6 — Build the timeline

    Authentication
          ↓
    Process Execution
          ↓
    Privilege Activity
          ↓
    Network Communication
          ↓
    Additional Host Access

### Step 7 — Determine scope

Search available telemetry for the same:

- Account
- Source IP
- Destination
- Process
- Hash
- Authentication pattern

This is the foundation of enterprise AD investigation.

---

## 21. Active Directory Attack Concepts for Future Days

The challenge will later cover defensive detection concepts around:

- Password spraying
- Brute-force attacks
- Kerberoasting
- AS-REP Roasting
- Pass-the-Hash
- Pass-the-Ticket
- Golden Ticket
- Silver Ticket
- DCSync
- NTDS.dit attacks
- Credential dumping
- Privilege escalation
- Lateral movement
- Domain persistence
- Malicious Group Policy modification

These should be practiced only in authorized lab environments.

---

## 22. MITRE ATT&CK Relevance

Active Directory activity maps to multiple ATT&CK techniques, including concepts such as:

- Valid Accounts
- Account Discovery
- Permission Groups Discovery
- Remote Services
- Credential Dumping
- Domain Trust Discovery
- Account Manipulation

A practical SOC workflow is:

    Observed Behavior
          ↓
        Evidence
          ↓
    Technique Mapping
          ↓
      Detection
          ↓
    Investigation
          ↓
      Response

MITRE ATT&CK supports the investigation; it does not replace evidence-based analysis.

---

## 23. Active Directory Investigation Checklist

### Identity

- Which account?
- User or service account?
- Privileged account?
- SID?
- Group membership?

### Authentication

- Successful or failed?
- Logon type?
- Kerberos or NTLM?
- Source host?
- Destination host?
- Time?

### Endpoint

- Process created?
- Parent process?
- Command line?
- PowerShell?
- Suspicious executable?

### Directory

- Account created?
- Account modified?
- Group membership changed?
- Privileges changed?
- GPO modified?

### Network

- Internal or external source?
- New destination?
- Lateral movement pattern?

### Timeline

- What happened immediately before?
- What happened immediately after?
- Is this isolated or part of a larger sequence?

---

## 🔍 SOC Analyst Perspective

Active Directory should be viewed as an **identity and control plane**, not simply as a Windows administration technology.

When an account performs an action, the analyst should be able to connect:

    Identity
       ↓
    Authentication
       ↓
    Host
       ↓
    Process
       ↓
    Privilege
       ↓
    Network Activity
       ↓
    Resource Access

A successful login by itself is normally just an event. A sequence such as:

    Privileged Account
          ↓
    Unusual Source Host
          ↓
    Successful Remote Logon
          ↓
    PowerShell Execution
          ↓
    Credential Access
          ↓
    Additional Host Authentication

provides substantially more investigation context.

The central lesson is:

> **Understand the identity, understand the authentication, understand the privilege, and correlate the activity across the environment.**

---

## 📌 Key Concepts Learned

- Active Directory Domain Services
- Domains and Domain Controllers
- AD objects
- Users and computer accounts
- Security Identifiers
- Groups and privileged groups
- Organizational Units
- Kerberos
- NTLM
- LDAP
- DNS and AD
- Group Policy
- Security filtering
- WMI filtering
- Blocked inheritance
- Enforced GPOs
- Domain authentication
- Windows logon types
- Windows Security Event IDs
- Lateral movement
- Account and group changes
- AD investigation methodology
- MITRE ATT&CK relevance

---

## 📌 Day 7 Outcome

Day 7 established the Active Directory foundation required for enterprise SOC monitoring.

The focus was on understanding **domains, domain controllers, users, computers, groups, OUs, authentication, Kerberos, LDAP, Group Policy, security filtering, WMI filtering, Windows Security events, and AD investigation methodology**.

The next stage will move into **Active Directory attacks and detection**, connecting attacker techniques with Windows security telemetry and SOC investigation workflows.

**Status: ✅ Completed**
