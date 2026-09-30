# Day 8 — Active Directory Attacks & Detection

## 🎯 Objective

Day 8 focuses on understanding how attackers abuse Active Directory after gaining an initial foothold in an enterprise environment.

The goal is not simply to memorize attack names. A SOC analyst needs to understand the relationship between an attacker action, the Windows/Active Directory telemetry it can generate, the indicators that may appear in a SIEM, and the additional evidence required to determine whether the activity is legitimate or malicious.

The core investigation model for this day is:

```text
Attack Technique
      ↓
Attacker Behavior
      ↓
Windows / AD Telemetry
      ↓
Detection Logic
      ↓
Alert
      ↓
Investigation
      ↓
Scope and Response
```

All practical testing should be performed only in an authorized lab environment.

---

## 1. Why Active Directory Is a High-Value Target

Active Directory centralizes identity, authentication, authorization, and administrative control.

An attacker who compromises a normal workstation may therefore attempt to move toward:

- Valid user credentials
- Local administrator access
- Domain administrator privileges
- Service accounts
- Domain Controllers
- Sensitive servers
- High-value business systems

A simplified attack path is:

```text
Initial Access
      ↓
Credential Access
      ↓
Account Discovery
      ↓
Privilege Escalation
      ↓
Lateral Movement
      ↓
Domain Persistence
      ↓
Impact
```

This is why AD telemetry is extremely valuable to a SOC.

---

## 2. Valid Accounts

One of the most important concepts in AD security is the abuse of legitimate credentials.

An attacker does not always need to create a new account. They may steal or obtain an existing account and use it to authenticate normally.

Example:

```text
Compromised User
      ↓
Valid Credentials
      ↓
Successful Authentication
      ↓
Internal Resource Access
```

The authentication event itself may look legitimate.

### SOC investigation

The analyst should compare:

- User
- Source host
- Destination host
- Time
- Logon type
- Authentication protocol
- Historical behavior
- Privilege level
- Subsequent activity

A valid account being used from an unusual workstation can be more meaningful than the account name alone.

---

## 3. Password Spraying

Password spraying attempts a small number of commonly used passwords against many accounts.

This differs from traditional brute force.

### Brute force

```text
One Account
     ↓
Many Password Attempts
```

### Password spraying

```text
Many Accounts
     ↓
One or Few Password Attempts
```

The objective is to avoid triggering account lockout policies while finding accounts with weak passwords.

### SOC detection perspective

Look for patterns such as:

```text
Same Source
    ↓
Many Usernames
    ↓
Authentication Failures
    ↓
Similar Time Window
```

Relevant evidence can include Event ID 4625 and domain-controller authentication telemetry.

A detection should consider normal authentication patterns and avoid treating every failed login as password spraying.

---

## 4. Brute-Force Authentication

A brute-force attack repeatedly attempts authentication against a target account or service.

A simplified pattern is:

```text
Target Account
      ↓
Many Failed Attempts
      ↓
Potential Successful Login
```

A SOC analyst should investigate:

- Number of failures
- Time window
- Source IP
- Source hostname
- Target account
- Target system
- Whether a successful login followed
- Whether the source is expected

A single failed authentication is usually weak evidence. A high-volume, correlated pattern is more useful.

---

## 5. Account Lockout Investigation

Repeated failed authentication can trigger account lockout.

A useful investigation sequence is:

```text
Failed Authentication
        ↓
Repeated Attempts
        ↓
Account Lockout
        ↓
Source Identification
        ↓
Root Cause Investigation
```

An account may be locked because of:

- User error
- Old stored credentials
- Service configuration
- Scheduled task
- Application authentication
- Password spraying
- Brute-force activity

Therefore, account lockout is an investigation lead, not automatic proof of attack.

---

## 6. Credential Access

Attackers may attempt to obtain credentials after compromising a Windows system.

Potential credential sources include:

- LSASS-related credential material
- Cached credentials
- Browser credentials
- Credential files
- Password databases
- NTDS.dit
- Kerberos tickets
- Authentication secrets

### SOC perspective

Credential access is particularly important because stolen credentials can enable lateral movement.

A common investigation chain is:

```text
Endpoint Compromise
      ↓
Credential Access
      ↓
Credential Reuse
      ↓
Remote Authentication
      ↓
Lateral Movement
```

Detection should correlate process behavior, privilege level, endpoint telemetry, and subsequent authentication activity.

---

## 7. LSASS and Credential Theft

The Local Security Authority Subsystem Service, commonly known as LSASS, is responsible for important Windows security and authentication functions.

Because authentication secrets can be associated with LSASS, attackers may attempt to access its memory.

A suspicious process accessing LSASS can therefore become an important endpoint investigation signal.

### SOC investigation

Do not stop at:

```text
Process → LSASS
```

Investigate:

- Which process accessed LSASS?
- Which user executed it?
- Was the process elevated?
- What is the executable path?
- Is it signed?
- What was its parent process?
- What happened immediately afterward?
- Did the account authenticate to other systems?

This transforms a single endpoint event into a behavioral investigation.

---

## 8. Kerberoasting

Kerberoasting targets service accounts associated with Kerberos service principals.

A simplified concept is:

```text
Domain User
      ↓
Service Ticket Request
      ↓
Service Account Ticket Material
      ↓
Offline Password Cracking Attempt
```

The attacker attempts to obtain ticket material that can be attacked offline.

### SOC detection perspective

Relevant telemetry may include unusual volumes or patterns of service-ticket requests, particularly when correlated with:

- Unusual user
- Multiple service accounts
- Unusual host
- Unusual timing
- Other credential-access behavior

Event ID 4769 can be important in this investigation.

A single service-ticket request is normal. Detection requires context and baseline.

---

## 9. AS-REP Roasting

AS-REP Roasting can target accounts configured so that Kerberos pre-authentication is not required.

The attacker may obtain authentication material that can be subjected to offline password attacks.

Conceptually:

```text
Target Account
      ↓
No Kerberos Pre-Authentication
      ↓
Authentication Response
      ↓
Offline Password Attack
```

### SOC perspective

The analyst should identify:

- Which account was targeted
- Which host requested the response
- Whether the account configuration is intentional
- Whether the activity matches expected administrative behavior
- Whether additional credential-access activity occurred

---

## 10. Pass-the-Hash

Pass-the-Hash involves using an acquired NTLM hash or equivalent credential material to authenticate without knowing the plaintext password.

Simplified:

```text
Credential Material Obtained
          ↓
NTLM Authentication
          ↓
Remote Access
          ↓
Lateral Movement
```

### Detection perspective

Look for unusual authentication patterns involving:

- Administrative accounts
- Workstation-to-workstation access
- Remote services
- NTLM
- Unusual source and destination relationships

The authentication event should be correlated with endpoint process and network telemetry.

---

## 11. Pass-the-Ticket

Pass-the-Ticket involves abusing stolen Kerberos tickets.

Conceptually:

```text
Kerberos Ticket Obtained
        ↓
Ticket Reused
        ↓
Authentication
        ↓
Access to Service
```

Detection may require correlation between:

- Account
- Host
- Kerberos activity
- Ticket-related telemetry
- Service access
- Process behavior

Because legitimate Kerberos activity can be complex, detection should be based on behavioral anomalies rather than one event.

---

## 12. Golden Ticket

A Golden Ticket attack abuses knowledge of the domain's Kerberos trust material to forge Ticket Granting Tickets.

At a high level:

```text
Domain Kerberos Secret Compromised
             ↓
Forged TGT
             ↓
Authentication as Chosen Identity
             ↓
Potential Domain-Level Access
```

This represents a serious domain-compromise scenario.

### SOC perspective

Investigation may involve:

- Unusual Kerberos ticket properties
- Unexpected account identifiers
- Unusual ticket lifetime
- Authentication from unexpected systems
- Domain-controller telemetry
- Endpoint evidence
- Credential compromise indicators

A Golden Ticket cannot normally be confirmed from one generic authentication event.

---

## 13. Silver Ticket

A Silver Ticket attack involves forging a Kerberos service ticket for a specific service.

Conceptually:

```text
Service Account Secret Compromised
            ↓
Forged Service Ticket
            ↓
Target Service Access
```

The scope differs from a Golden Ticket because the forged ticket is associated with a specific service rather than the broader domain authentication trust represented by the TGT.

### SOC perspective

Investigate:

- Target service
- Service account
- Source host
- Kerberos activity
- Account context
- Service access
- Endpoint process behavior

---

## 14. DCSync

DCSync is a technique in which an attacker abuses directory replication privileges to request credential-related information as though they were a legitimate domain controller replication partner.

Conceptually:

```text
Compromised Privileged Identity
          ↓
Replication Permissions
          ↓
Directory Replication Request
          ↓
Credential Material
```

This is highly significant because it can expose domain credential information.

### Detection perspective

Monitor for unexpected directory replication behavior and investigate:

- Requesting account
- Source host
- Privileges
- Domain-controller context
- Replication-related events
- Subsequent credential use

---

## 15. NTDS.dit

Domain controllers maintain the Active Directory database in the **NTDS.dit** file.

It contains critical directory information.

An attacker who obtains unauthorized access to domain-controller credential data can potentially compromise many domain identities.

### SOC perspective

Access to domain-controller database material should be treated as high-value evidence.

Investigation should consider:

- Which process accessed the data
- Which account performed the action
- Whether the host is a legitimate domain controller
- File access and process telemetry
- Subsequent credential activity

---

## 16. Privilege Escalation Through Group Membership

Attackers may attempt to add accounts to privileged groups.

Example:

```text
Compromised User
      ↓
Group Membership Modification
      ↓
Privileged Group
      ↓
Elevated Access
```

Relevant Windows Security events can include group membership changes.

### SOC investigation

Determine:

- Member added
- Group modified
- Account that performed the modification
- Source system
- Timestamp
- Authorization
- Activity immediately afterward

A legitimate administrator can generate the same event, so change-management context matters.

---

## 17. Account Manipulation

Attackers may manipulate accounts to maintain access.

Possible actions include:

- Creating accounts
- Enabling disabled accounts
- Changing passwords
- Modifying group membership
- Modifying account attributes
- Adding alternative authentication mechanisms

Useful Security Event IDs include:

- 4720 — User account created
- 4722 — User account enabled
- 4723 — Password change attempt
- 4724 — Password reset attempt
- 4725 — User account disabled
- 4726 — User account deleted
- 4738 — User account changed

The exact event fields should be examined during investigation.

---

## 18. Group Policy Abuse

Because Group Policy can affect many systems, attackers with sufficient privileges may attempt to abuse it for:

- Persistence
- Configuration changes
- Security-control modification
- Script execution
- Broad endpoint changes

A simplified attack path is:

```text
Privileged Access
      ↓
GPO Modification
      ↓
Policy Replication
      ↓
Multiple Systems Affected
```

### SOC perspective

Investigate:

- Who changed the GPO?
- What setting changed?
- Which OU or systems are affected?
- Was the change authorized?
- When did it occur?
- Did endpoint behavior change afterward?

---

## 19. Lateral Movement

Lateral movement occurs when an attacker moves from one compromised system to another.

Common Windows technologies include:

- RDP
- SMB
- WinRM
- Remote services
- Administrative shares
- Remote administration tools

Example:

```text
WORKSTATION01
      ↓
Compromised Account
      ↓
SERVER01
      ↓
SERVER02
      ↓
Domain Controller
```

### SOC investigation

Correlate:

- Source host
- Destination host
- Account
- Authentication type
- Logon type
- Process
- Network connection
- Time sequence

This is where endpoint and network telemetry become particularly valuable.

---

## 20. Remote Desktop Protocol

RDP provides graphical remote access to Windows systems.

It is legitimate in many environments but can also be abused after credential compromise.

A SOC analyst should investigate unusual RDP activity by checking:

- Source IP
- Source host
- Destination host
- User
- Logon type
- Time
- Previous authentication failures
- Process activity after logon

A remote logon should be evaluated against expected administrative behavior.

---

## 21. SMB and Administrative Shares

SMB supports file and printer sharing and several Windows administrative functions.

Administrative shares can include resources such as:

- ADMIN$
- C$
- IPC$

Attackers may abuse legitimate Windows mechanisms for remote administration and lateral movement.

### SOC perspective

Investigate unusual access by correlating:

```text
Source Host
    ↓
Account
    ↓
SMB Connection
    ↓
Destination
    ↓
File / Process Activity
```

---

## 22. WinRM and PowerShell Remoting

Windows Remote Management can be used for legitimate administration and remote PowerShell execution.

It can also become a lateral-movement mechanism after account compromise.

A useful investigation chain is:

```text
Remote Authentication
      ↓
WinRM / PowerShell
      ↓
Command Execution
      ↓
Child Process
      ↓
Network Activity
```

PowerShell logging and process telemetry can provide additional evidence.

---

## 23. Detection Engineering Approach

An effective AD detection should not simply ask:

> Did Event ID X occur?

Instead, ask:

> What behavior does this event represent, and what additional context would make the behavior suspicious?

For example:

```text
4624 Successful Logon
        +
Unusual Source Host
        +
Privileged Account
        +
Rare Destination
        +
Suspicious Process
        =
Higher Investigation Priority
```

This is a more robust approach than relying on a single event.

---

## 24. Correlation Across Telemetry

A SOC may receive evidence from multiple sources:

| Source | Useful Evidence |
|---|---|
| Windows Security | Authentication and account events |
| Sysmon | Process, network and file telemetry |
| PowerShell logs | Script execution |
| DNS logs | Name resolution |
| Firewall logs | Network connections |
| EDR | Endpoint behavior |
| SIEM | Correlation and detection |
| Domain Controller | AD authentication and directory activity |

The investigation should connect these sources into one timeline.

---

## 25. Example Detection Scenario

Suppose a normal employee account generates the following sequence:

```text
Multiple failed logons
        ↓
Successful logon from unusual workstation
        ↓
PowerShell execution
        ↓
Remote authentication
        ↓
Access to administrative share
```

A SOC analyst should not immediately conclude that the account is compromised.

Instead, investigate:

1. Was the user expected to work from that workstation?
2. Were the failed logons caused by a legitimate application?
3. Is PowerShell execution normal for this user?
4. Which destination was accessed?
5. Which process initiated the remote connection?
6. Did the account access other systems?
7. Did any privilege changes occur?
8. Does the same behavior appear elsewhere?

This avoids both false positives and premature conclusions.

---

## 26. Practical Detection Lab

### Lab 1 — Authentication Baseline

Generate normal domain authentication activity and record:

- User
- Source
- Destination
- Logon type
- Timestamp
- Authentication package

Build a baseline before attempting detection.

### Lab 2 — Failed Authentication

Generate controlled failed authentication attempts using test accounts.

Observe relevant Security events and compare them with successful authentication.

### Lab 3 — Account Lockout

In a controlled lab, trigger a test account lockout and investigate the surrounding authentication events.

Identify the source of the failed attempts.

### Lab 4 — Group Membership Change

Add a test account to an authorized lab group and inspect the resulting Security telemetry.

Document:

- Modified group
- Added member
- Actor
- Timestamp

### Lab 5 — Remote Logon

Use an authorized test account to perform a normal remote logon between lab systems.

Correlate the authentication event with source and destination hosts.

### Lab 6 — PowerShell Correlation

Execute a benign PowerShell command in the lab and correlate:

```text
User
 ↓
Process Creation
 ↓
PowerShell
 ↓
Command
 ↓
Network Activity
```

### Lab 7 — Detection Rule Design

Create a conceptual detection for unusual privileged authentication.

Example logic:

```text
IF
    privileged_account = true
AND
    source_host is unusual
AND
    destination_host is unusual
AND
    authentication occurs outside normal pattern
THEN
    generate investigation alert
```

The exact thresholds should be tuned to the organization's baseline.

---

## 27. MITRE ATT&CK Mapping

The techniques discussed during Day 8 relate to areas such as:

- T1078 — Valid Accounts
- T1110 — Brute Force
- T1003 — OS Credential Dumping
- T1558 — Steal or Forge Kerberos Tickets
- T1021 — Remote Services
- T1098 — Account Manipulation
- T1484 — Domain or Tenant Policy Modification
- T1069 — Permission Groups Discovery
- T1018 — Remote System Discovery

Technique mapping should be based on observed behavior and available evidence.

---

## 28. SOC Investigation Workflow

When an AD-related alert arrives, use a repeatable workflow:

### Phase 1 — Validate

Determine whether the alert represents actual observed activity.

### Phase 2 — Identify

Find:

- Account
- Host
- IP
- Process
- Destination
- Event

### Phase 3 — Correlate

Connect:

```text
Authentication
      +
Process
      +
Network
      +
Directory
      +
Timeline
```

### Phase 4 — Scope

Search for the same indicators across other systems.

### Phase 5 — Assess

Determine whether the behavior is consistent with legitimate activity or requires escalation.

### Phase 6 — Respond

Follow the organization's incident-response procedure, which may include account containment, host isolation, credential reset, or other approved actions.

---

## 29. Key Concepts Learned

- Valid Accounts
- Password spraying
- Brute-force attacks
- Account lockout investigation
- Credential access
- LSASS credential theft concepts
- Kerberoasting
- AS-REP Roasting
- Pass-the-Hash
- Pass-the-Ticket
- Golden Ticket
- Silver Ticket
- DCSync
- NTDS.dit
- Account manipulation
- Privileged group abuse
- Group Policy abuse
- Lateral movement
- RDP
- SMB
- WinRM
- PowerShell remoting
- AD detection engineering
- Telemetry correlation
- MITRE ATT&CK mapping

---

## 🔍 SOC Analyst Perspective

The biggest lesson from Active Directory attack detection is that **individual events rarely tell the entire story**.

Consider this sequence:

```text
Failed Authentication
        ↓
Successful Authentication
        ↓
PowerShell
        ↓
Credential Access
        ↓
Remote Authentication
        ↓
Administrative Share
        ↓
Additional Host
```

Each event can have a legitimate explanation in isolation.

The SOC analyst's job is to correlate the events, establish a timeline, understand the identity involved, determine the source and destination, identify privilege changes, and investigate the broader scope.

The operational mindset should be:

> **Do not investigate the event in isolation. Investigate the behavior chain.**

That approach is fundamental to detecting identity-based attacks in enterprise environments.

---

## 📌 Day 8 Outcome

Day 8 established the defensive foundation for understanding Active Directory attack paths.

The focus was on **valid accounts, password spraying, brute force, credential access, Kerberos abuse, credential theft, privilege escalation, account manipulation, Group Policy abuse, and Windows lateral movement**.

The next stage will move into **Wazuh architecture**, connecting endpoint telemetry and detection concepts with a practical SIEM/XDR monitoring platform.

**Status: ✅ Completed**
