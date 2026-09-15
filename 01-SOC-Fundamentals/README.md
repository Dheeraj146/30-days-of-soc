# Day 1 — SOC Fundamentals

## Focus

Security Operations Center (SOC), SOC functions and roles, security monitoring, logs and events, alerts and incidents, incident response, SIEM, log management, security tools, and the complete movement of security telemetry from source systems to a SOC analyst.

## Objective

The objective of Day 1 is to understand **how a SOC operates as an end-to-end security function**. The goal is not simply to memorize definitions of SIEM, EDR, IDS, or other security tools. The goal is to understand how **people, processes, and technology** work together to monitor an environment, identify suspicious behavior, investigate alerts, respond to incidents, document findings, and improve security operations.

```text
Network / Cloud / Users / Endpoints / Applications
                         |
                         v
                Logs / Security Telemetry
                         |
                         v
              Collection / Ingestion
                         |
                         v
              Parsing / Normalization
                         |
                         v
                    Enrichment
                         |
                         v
              Storage / Indexing
                         |
                         v
           Detection / Correlation / Analytics
                         |
                         v
                       ALERT
                         |
                         v
                    SOC Analyst
                         |
                       Triage
                         |
                +--------+--------+
                |                 |
             Benign           Suspicious
                |                 |
             Close               v
                         Investigation
                               |
                               v
                            Incident
                               |
                               v
                       Incident Response
                               |
                               v
                    Recovery / Remediation
                               |
                               v
                         Lessons Learned
                               |
                               v
                     Detection Improvement
```

---

# 1. What Is a SOC?

A **Security Operations Center (SOC)** is a centralized security function responsible for monitoring an organization's digital environment, detecting suspicious or malicious activity, investigating security alerts, coordinating incident response, documenting findings, and continuously improving the organization's security posture.

A SOC provides visibility across many parts of an organization, including endpoints, servers, networks, applications, identities, cloud environments, and security controls.

A modern organization can generate an enormous volume of security telemetry. Thousands of endpoints, servers, users, applications, firewalls, VPN gateways, cloud services, and security products may all produce events. It is not practical for analysts to manually inspect every event. The SOC therefore uses technology and processes to collect, process, prioritize, and investigate the activity that matters.

A SOC is not simply a room containing analysts and dashboards. It is an operational capability that continuously asks questions such as:

- What is happening in the environment?
- Which activity is expected and which is suspicious?
- Which alerts require immediate attention?
- Is the alert a true positive or false positive?
- Which user, endpoint, server, application, or network segment is involved?
- What happened before the suspicious activity?
- What happened after it?
- How far has an attacker progressed?
- What evidence supports the conclusion?
- What is the scope and potential impact?
- What response is appropriate?
- How can the same activity be detected more effectively in the future?

## 1.1 Three Pillars of a SOC

### People

People perform investigation, decision-making, response, communication, and security engineering. Typical personnel include SOC analysts, incident responders, threat hunters, detection engineers, security engineers, malware analysts, digital forensics specialists, and SOC managers.

### Processes

Processes provide repeatable procedures for monitoring, triage, escalation, investigation, incident classification, containment, eradication, recovery, documentation, and post-incident improvement.

### Technology

Technology provides telemetry, detection, analysis, automation, and response capabilities. Common technologies include SIEM, SOAR, EDR/XDR, IDS/IPS, firewalls, VPN systems, identity platforms, threat-intelligence platforms, vulnerability-management systems, and log-management platforms.

**Important:** No single security product provides complete visibility. A SOC becomes more effective when telemetry from multiple layers is correlated and interpreted together.

---

# 2. Key Functions of a SOC

The major SOC functions studied today are **continuous monitoring, threat detection, incident response, reporting and documentation, and continuous improvement**.

## 2.1 Continuous Monitoring

Continuous monitoring means maintaining ongoing visibility into security-relevant activity across the organization's environment.

A SOC may monitor:

- Endpoints and workstations
- Servers
- Active Directory and identity systems
- Network devices
- Firewalls
- VPN gateways
- DNS infrastructure
- Applications
- Databases
- Cloud services
- Email systems
- Security products

The purpose is not for analysts to manually inspect every event. Enterprise environments can generate huge volumes of telemetry. Instead, the SOC uses automated collection, filtering, parsing, correlation, detection rules, and analytics to surface activity that requires human attention.

### Examples of monitored activity

- Repeated failed authentication attempts
- Successful authentication after many failures
- Authentication from an unusual location
- Creation of a new user account
- Privilege or group-membership changes
- Suspicious PowerShell execution
- Unusual process creation
- Suspicious parent-child process relationships
- Unexpected outbound network connections
- DNS requests to suspicious domains
- Malware detections
- Modification of critical files
- Firewall allow/deny activity
- VPN authentication activity
- Cloud administrative changes

### Why continuous monitoring matters

Attackers can operate outside normal business hours. If an account is compromised at night and there is no monitoring, the attacker may have significant time to perform reconnaissance, establish persistence, move laterally, or access sensitive information.

Continuous monitoring reduces the time between malicious activity and detection. The SOC therefore contributes to reducing the time an attacker can operate without being discovered.

---

## 2.2 Threat Detection

Threat detection is the process of identifying activity that may represent malicious behavior, compromise, abuse, or a security-policy violation.

Detection can be based on an individual event, but strong detections often depend on **patterns and relationships between multiple events**.

### Signature-based detection

Signature-based detection looks for known patterns associated with known malicious artifacts or behavior. It is useful for known threats but may be less effective when an attacker modifies an artifact or uses a previously unknown technique.

### Rule-based detection

Rule-based detection uses explicit conditions.

```text
IF failed_login_count >= 10
AND same_source_ip
AND within 5 minutes
THEN generate brute-force alert
```

### Correlation-based detection

Correlation combines multiple events to identify a meaningful sequence.

```text
Multiple failed logins
        +
Successful authentication
        +
Privileged activity
        +
Suspicious process execution
        +
Suspicious outbound connection
        ↓
Potential account compromise
```

### Behavioral detection

Behavioral detection looks for activity that differs from an established or expected pattern.

### Threat-intelligence-based detection

Observed indicators such as IP addresses, domains, URLs, and file hashes can be compared with threat-intelligence information.

### Anomaly detection

Anomaly detection identifies activity that deviates from a baseline. An anomaly is **not automatically malicious**; it still requires context and investigation.

---

## 2.3 Incident Response

Incident response is the structured process used when suspicious activity is confirmed or meets an organization's criteria for a security incident.

It is broader than simply blocking an IP address. The organization must understand the incident, determine scope and impact, contain the threat, eradicate the cause, recover affected systems, document the investigation, and improve defenses.

---

## 2.4 Reporting and Documentation

Security investigations require a reliable evidence trail. Documentation allows another analyst, manager, auditor, or incident-response team to understand what happened and why a particular conclusion was reached.

Analysts should document:

- What happened
- When it happened
- Which users were involved
- Which systems were involved
- Source and destination information
- Relevant logs and alerts
- Evidence collected
- Investigation steps
- Timeline of activity
- Findings
- Scope
- Impact
- Actions performed
- Containment and remediation
- Final classification
- Recommended improvements

Good documentation should explain the analyst's reasoning, not just record the final conclusion.

---

## 2.5 Continuous Improvement

A mature SOC treats incidents, false positives, missed detections, and analyst feedback as opportunities to improve.

The SOC may ask:

- Why did the detection trigger?
- Was the alert accurate?
- Could the activity have been detected earlier?
- Was enough telemetry available?
- Did the alert contain sufficient context?
- Was the escalation path clear?
- Did the response process work correctly?
- Could the same attack bypass the detection again?
- Should the rule be tuned?
- Should a new detection be created?
- Should a response playbook be updated?

```text
Incident / Alert
      ↓
Investigation
      ↓
Findings
      ↓
Lessons Learned
      ↓
Detection / Process Improvement
      ↓
Better Monitoring
      ↓
Better Future Detection
```

---

# 3. How a SOC Works

A SOC operates as an end-to-end pipeline. The process begins when activity occurs in the environment and telemetry is generated; the analyst becomes involved after automated collection and detection have provided a security signal.

```text
1. Activity occurs
        ↓
2. Source generates telemetry
        ↓
3. Telemetry is collected
        ↓
4. Data is parsed
        ↓
5. Fields are normalized
        ↓
6. Context is enriched
        ↓
7. Data is stored / indexed
        ↓
8. Detection and correlation run
        ↓
9. Alert is generated
        ↓
10. Analyst performs triage
        ↓
11. Investigation
        ↓
12. Classification
        ↓
13. Escalation / response
        ↓
14. Recovery / remediation
        ↓
15. Documentation
        ↓
16. Lessons learned
        ↓
17. Detection / process improvement
```

### Example

Suppose an attacker is attempting to compromise an administrator account.

The authentication system records failed login events. The events are collected and sent to a centralized security platform. The platform extracts information such as username, source IP, timestamp, and authentication result.

A detection rule identifies an unusual number of failures within a short time window and generates an alert. The L1 analyst checks whether the source is expected, whether the account is legitimate, whether a successful login occurred, and whether other suspicious activity is present.

If the activity is suspicious, the investigation expands to endpoint, network, VPN, identity, and threat-intelligence data. If evidence supports compromise, the case is escalated and incident-response actions begin.

Therefore, an alert is not the end of SOC activity. **An alert is the beginning of an analyst-driven investigation.**

---

# 4. SOC Roles and Responsibilities

SOC roles are commonly divided into tiers. The exact responsibilities vary between organizations, but the following model provides a useful operational framework.

## 4.1 L1 — Tier 1 SOC Analyst

The L1 analyst is generally the first human layer handling security alerts. The primary responsibility is **rapid and accurate initial triage**.

### Responsibilities

- Monitor incoming alerts
- Review alert severity and detection reason
- Identify source, destination, user, and host
- Review relevant event details
- Check surrounding activity
- Determine whether activity is expected
- Identify obvious false positives
- Collect initial evidence
- Follow documented playbooks
- Escalate suspicious activity
- Document actions and findings

### L1 workflow

```text
Alert received
      ↓
Read alert details
      ↓
Identify entities involved
      ↓
Check related events
      ↓
Expected activity?
   /          \
 Yes          No
  ↓            ↓
Document     Suspicious?
Close          ↓
            Escalate
               ↓
               L2
```

L1 is not simply a "basic" role. High-quality L1 triage prevents unnecessary escalation while ensuring genuine threats move quickly to deeper investigation.

---

## 4.2 L2 — Tier 2 SOC Analyst

L2 handles investigations requiring deeper technical analysis and correlation across multiple sources.

### Responsibilities

- Investigate escalated alerts
- Correlate authentication, endpoint, network, and application data
- Build event timelines
- Determine affected systems and accounts
- Analyze suspicious processes
- Analyze network connections
- Determine attack scope
- Identify likely attack techniques
- Support containment
- Escalate complex incidents to L3 or specialized teams

### Example

L1 receives a brute-force alert. L2 may discover that the attacker eventually authenticated successfully, identify the endpoint used by the account, inspect process execution, examine network connections, and determine whether lateral movement occurred.

---

## 4.3 L3 — Tier 3 / Senior Analyst

L3 analysts generally handle advanced investigations and specialized security operations.

### Responsibilities

- Advanced incident investigation
- Threat hunting
- Detection engineering
- Malware analysis
- Digital forensics
- Complex attack-chain analysis
- Advanced endpoint investigation
- Detection tuning
- Development of new detection rules
- Investigation of sophisticated or persistent threats
- Support for major incidents

L3 work often goes beyond responding to existing alerts. L3 analysts may proactively search for attacker behavior that has not yet generated an alert.

---

## 4.4 SOC Manager

The SOC manager is responsible for operational leadership and overall SOC effectiveness.

### Responsibilities

- Team management
- Work allocation
- Escalation management
- Incident coordination
- SOC procedures
- Metrics and reporting
- Staffing and capability planning
- Security-tool strategy
- Risk coordination
- Compliance coordination
- Stakeholder communication
- Operational maturity

### Example SOC metrics

- Alert volume
- Mean Time to Detect (MTTD)
- Mean Time to Respond (MTTR)
- False-positive rate
- Escalation rate
- Incident volume
- Detection coverage
- SLA compliance
- Investigation backlog

> Exact responsibilities vary between organizations. Some organizations combine tiers or have separate teams for incident response, threat hunting, detection engineering, malware analysis, and digital forensics.

---

# 5. Logs, Events, Alerts, and Incidents

These concepts are closely related but should not be treated as identical.

## 5.1 What Is a Log?

A **log** is recorded information generated by an operating system, application, network device, security control, cloud service, or other technology component.

Examples include:

```text
User authentication succeeded
User authentication failed
Process started
Process terminated
File modified
Firewall connection allowed
Firewall connection denied
DNS query performed
VPN connection established
Service started
Account created
```

Logs preserve historical evidence. Analysts can use them to reconstruct activity, identify patterns, troubleshoot systems, support audits, and investigate incidents.

### Common security-log fields

A security log may contain:

- Timestamp
- Hostname
- Username
- Source IP
- Destination IP
- Source port
- Destination port
- Protocol
- Process name
- Process ID
- Event ID
- Action
- Result
- Status
- Message

The exact fields depend on the source.

---

## 5.2 What Is an Event?

An **event** represents something that happened within a system or environment.

Example:

```text
Event: User failed authentication
Time: 10:15:22
User: alice
Source IP: 10.10.10.25
Result: Failure
```

Conceptually:

- **Event:** the activity that occurred.
- **Log:** the recorded information describing that activity.

In practical SOC conversations the terms may sometimes be used interchangeably, but the distinction is useful when learning how security telemetry works.

---

## 5.3 What Is an Alert?

An **alert** is a security notification generated when a security product or detection system identifies activity that matches a detection condition or otherwise requires analyst attention.

Example:

```text
Observed:
Failed login
Failed login
Failed login
Failed login
Failed login

Detection:
Multiple failures from same source within a time window

Result:
ALERT — Possible brute-force activity
```

An alert represents a **security hypothesis or condition requiring evaluation**.

An alert can be:

- A true positive
- A false positive
- Benign but unusual activity
- Expected administrative activity
- Suspicious activity requiring further investigation

### Critical principle

> **ALERT ≠ INCIDENT**

The analyst must investigate the context before deciding whether the activity represents an incident.

---

## 5.4 What Is an Incident?

A **security incident** is an event or series of events that meets an organization's criteria for requiring a security response.

Example:

```text
Multiple failed logins
        ↓
Successful login from unusual source
        ↓
Suspicious PowerShell execution
        ↓
Credential-access indicators
        ↓
Suspicious outbound connection
        ↓
Evidence supports compromise
        ↓
SECURITY INCIDENT
```

A single suspicious event may require investigation without being classified as an incident. Multiple correlated events can provide much stronger evidence.

---

# 6. Incident Response

**Incident Response (IR)** is the structured approach used to prepare for, detect, analyze, contain, eradicate, and recover from security incidents.

A simplified lifecycle is:

```text
Preparation
    ↓
Detection & Analysis
    ↓
Containment
    ↓
Eradication
    ↓
Recovery
    ↓
Lessons Learned
```

## 6.1 Preparation

Preparation occurs before an incident.

It includes:

- Incident-response procedures
- Escalation contacts
- Logging and monitoring
- Backups
- Response playbooks
- Forensic capabilities
- Security tools
- Communication procedures
- Defined responsibilities

Preparation ensures that the organization has the capabilities required to respond before a crisis occurs.

## 6.2 Detection and Analysis

The organization identifies suspicious activity and determines what actually happened.

Analysts may examine:

- SIEM alerts
- Windows/Linux logs
- Authentication activity
- Endpoint telemetry
- Firewall logs
- DNS logs
- Network traffic
- Cloud activity
- Threat intelligence
- Process execution
- File activity

The objective is to establish facts, determine scope, and understand the attack path.

## 6.3 Containment

Containment limits the attacker's ability to continue operating and prevents additional damage.

Possible actions include:

- Isolating an endpoint
- Disabling a compromised account
- Revoking sessions or tokens
- Blocking malicious infrastructure
- Restricting network access
- Removing exposed credentials

Containment must consider business impact. Response actions should follow organizational procedures and authorization requirements.

## 6.4 Eradication

Eradication removes the malicious presence or underlying cause.

Examples include:

- Removing malware
- Removing persistence mechanisms
- Resetting compromised credentials
- Removing unauthorized accounts
- Patching exploited vulnerabilities
- Correcting insecure configurations

## 6.5 Recovery

Recovery returns systems and services to a trusted operational state. Systems should continue to be monitored after recovery to identify recurrence.

## 6.6 Lessons Learned

After an incident, the organization reviews what happened and identifies improvements to technology, detections, procedures, and training.

---

# 7. Incident vs Disaster

A **security incident** and a **disaster** are not the same concept.

A security incident is generally an event or set of events requiring security investigation and response.

A disaster is a major disruptive event that significantly affects critical business operations or services.

For example:

```text
Compromised workstation
        ↓
Security Incident
```

A large ransomware attack that encrypts critical systems, disrupts business operations, and requires major recovery efforts could contribute to a disaster scenario.

### Disaster Recovery

Disaster recovery focuses on restoring critical technology and business services after a major disruption. SOC and incident-response teams may contribute during cyber-related disasters, but disaster recovery is broader than SOC operations.

---

# 8. SIEM — Security Information and Event Management

A **SIEM** is a security platform that centralizes security-relevant telemetry and provides capabilities for collection, storage, search, analysis, correlation, detection, alerting, and investigation.

Without centralized monitoring, an analyst might need to inspect many independent systems:

```text
Windows logs
Linux logs
Firewall logs
VPN logs
DNS logs
EDR telemetry
Cloud logs
Application logs
```

A SIEM brings relevant telemetry into a centralized platform where it can be searched, correlated, analyzed, and used for detection.

### What a SIEM helps analysts investigate

- Which user performed the activity?
- Which endpoint was involved?
- What source IP generated the event?
- What destination did the endpoint contact?
- Did the same source contact other systems?
- Was there a successful login after failed attempts?
- Did suspicious process execution occur afterward?
- Are there related alerts?
- Does the activity match known threat indicators?

---

# 9. How a SIEM Collects and Processes Data

This is one of the most important concepts from Day 1.

```text
                       DATA SOURCES
                            |
       +--------------------+--------------------+
       |                    |                    |
    Windows               Linux             Firewall
    Servers                Hosts              / VPN
       |                    |                    |
       +--------------------+--------------------+
                            |
                            v
                    DATA COLLECTION
                            |
                            v
                         PARSING
                            |
                            v
                      NORMALIZATION
                            |
                            v
                        ENRICHMENT
                            |
                            v
                    STORAGE / INDEXING
                            |
                            v
               CORRELATION / DETECTION
                            |
                            v
                          ALERT
                            |
                            v
                     SOC ANALYST
                            |
                            v
                      INVESTIGATION
```

## 9.1 Data Collection

The first step is receiving telemetry from security-relevant sources.

Potential sources include:

- Windows Event Logs
- Linux system logs
- Application logs
- Firewalls
- IDS/IPS
- EDR
- VPN systems
- DNS infrastructure
- Cloud audit logs
- Identity providers
- Databases

Collection may use agents, collectors, APIs, network protocols, cloud integrations, or other ingestion mechanisms.

The quality of the final investigation depends heavily on whether the required telemetry is collected in the first place.

---

## 9.2 Parsing

Raw logs often arrive as text or vendor-specific structured data. **Parsing extracts individual fields** from the incoming data.

Raw:

```text
Failed login for user admin from 10.10.10.50
```

Parsed:

```text
username  = admin
source_ip = 10.10.10.50
action    = failed_login
```

Parsing is important because detection engines need structured information. It is easier to build flexible detection logic using fields such as username, source IP, action, and result than relying only on an entire message string.

---

## 9.3 Normalization

Different vendors often use different names for the same concept.

```text
Vendor A: src_ip
Vendor B: sourceAddress
Vendor C: client_ip
```

A normalized representation could use:

```text
source.ip
```

Normalization makes cross-source searches and detections easier because analysts and detection rules can reason about consistent concepts.

---

## 9.4 Enrichment

Enrichment adds useful context that was not present in the original event.

Possible enrichment includes:

- User identity
- Asset owner
- Asset criticality
- Geographic information
- IP reputation
- Domain reputation
- Threat-intelligence matches
- Vulnerability information
- Identity context
- Business context

Example:

```text
Original event:
Source IP = 203.0.113.50

Enriched context:
Source IP = 203.0.113.50
Threat intelligence = Suspicious
Target asset = Critical server
Asset owner = Security team
```

Enrichment helps analysts prioritize alerts and make better decisions with less manual context gathering.

---

## 9.5 Storage and Indexing

Collected telemetry must be stored so it can be searched during detection and investigation.

Common searchable fields include:

- Timestamp
- Username
- Source IP
- Destination IP
- Hostname
- Event ID
- Process name
- Domain
- File hash
- Action
- Result

Indexing allows analysts and detection systems to locate relevant records efficiently.

Retention requirements depend on organizational requirements, regulatory obligations, investigation needs, storage capacity, and the type of telemetry.

---

## 9.6 Correlation

Correlation connects related events together.

A single failed login may be normal. A large number of failures followed by a successful login, privileged activity, suspicious process execution, and an external connection is much more significant.

```text
10 failed logins
      +
Successful login
      +
Privileged activity
      +
Suspicious process
      +
Suspicious network connection
      ↓
Potential account compromise
```

Correlation allows the SOC to reason about an **attack sequence** instead of treating every event as an isolated record.

---

## 9.7 Detection and Alert Generation

Detection logic evaluates processed telemetry against defined conditions.

Example:

```text
IF
    failed_login_count >= 10
AND
    same_source_ip
AND
    within 5 minutes

THEN
    generate brute-force alert
```

A useful alert should ideally provide enough context for the analyst to begin investigation, such as the affected account, host, source, timestamps, event count, detection reason, and related evidence.

---

# 10. SIEM vs Log Management

These technologies are related, but their primary objectives differ.

## 10.1 Log Management System

A log-management system primarily manages the lifecycle of logs.

Typical functions include:

- Collecting logs
- Transporting logs
- Storing logs
- Indexing logs
- Searching logs
- Retaining logs
- Managing availability

The central question is:

> **Can we reliably collect, store, search, and retain our logs?**

## 10.2 SIEM

A SIEM includes centralized log-management capabilities but adds security-focused capabilities such as:

- Detection rules
- Correlation
- Security analytics
- Alerting
- Threat-intelligence integration
- Investigation workflows
- Security monitoring
- Security context

The central question becomes:

> **What security-relevant activity is occurring, and does the available telemetry indicate a threat?**

### Simplified comparison

```text
LOG MANAGEMENT
Collect → Transport → Store → Search → Retain

SIEM
Collect → Parse → Normalize → Enrich
        ↓
Store → Correlate → Detect → Alert
        ↓
Investigate → Respond
```

Modern platforms increasingly combine log-management and SIEM functionality, so the boundary is not always strict.

---

# 11. How Logs Can Be Transferred

Logs must move from the source system to a centralized location before they can be analyzed centrally.

Common mechanisms include:

1. Log forwarding
2. Agent-based collection
3. Network-based logging
4. API-based collection
5. File-based collection
6. Intermediate collectors
7. Cloud-native integrations

The appropriate method depends on the source system, network architecture, security requirements, reliability requirements, and capabilities of the receiving platform.

## 11.1 Log Forwarding

Log forwarding means a source system sends its log data to another system responsible for receiving or processing it.

```text
Endpoint / Server
       |
       | Log forwarding
       v
Collector / SIEM
```

### Benefits

- Centralized visibility
- Easier searching
- Centralized detection
- Easier investigation
- Reduced need for manual inspection of individual systems

## 11.2 Agent-Based Collection

An agent is software installed on an endpoint that collects selected telemetry and forwards it to a centralized platform.

```text
Windows / Linux Endpoint
          |
       Security Agent
          |
          v
     Collector / SIEM
```

Agents can collect:

- Operating-system logs
- Application logs
- Security events
- Process information
- File activity
- Endpoint telemetry

### Advantages

- Direct endpoint visibility
- Centralized configuration
- Controlled collection
- Useful for local log sources
- Endpoint-specific security telemetry

### Considerations

Agents consume endpoint resources and must be securely configured, maintained, upgraded, and monitored.

## 11.3 Network-Based Logging

Network devices and security appliances can send logs across the network to a central collector. A common example is **Syslog**.

```text
Firewall
   |
   | Syslog
   v
Log Collector
   |
   v
SIEM
```

Typical sources include:

- Firewalls
- Routers
- Switches
- VPN gateways
- IDS/IPS devices
- Network appliances

## 11.4 API-Based Collection

Cloud and SaaS platforms frequently expose security events through APIs.

```text
Cloud / SaaS / Security Platform
              |
             API
              |
              v
        Collector / SIEM
```

Important considerations include:

- API authentication
- Permissions
- Rate limits
- Polling intervals
- Pagination
- Event delays
- Duplicate events
- API failures

## 11.5 File-Based Collection / Log Copy

Some applications write logs to local files. An agent or collector can monitor those files and forward new records.

```text
Application
    |
    v
Log File
    |
    v
Agent / Collector
    |
    v
SIEM
```

Important considerations include log rotation, permissions, duplicate records, partial writes, file changes, and retention.

## 11.6 Collector-Based Architecture

Organizations may place an intermediate collector between data sources and the SIEM.

```text
Windows ──┐
Linux ────┤
Firewall ─┤
VPN ──────┤
Cloud ────┤
Apps ─────┘
      |
      v
Central Log Collector
      |
      v
SIEM
      |
      v
SOC
```

Collectors can provide buffering, filtering, routing, transformation, and centralized ingestion management.

---

# 12. Security Tools Used in a SOC

A SOC uses multiple security technologies because each technology provides a different type of visibility or control.

## 12.1 SIEM

**Security Information and Event Management** centralizes security telemetry and provides search, correlation, detection, alerting, analytics, and investigation capabilities.

## 12.2 SOAR

**Security Orchestration, Automation and Response** automates repetitive security workflows and coordinates actions across multiple security products.

Example:

```text
SIEM Alert
    ↓
SOAR Playbook
    ↓
Extract IP
    ↓
Threat Intelligence Lookup
    ↓
Is IP malicious?
   /        \
 Yes        No
  ↓           ↓
Block IP    Continue Investigation
  ↓
Create Ticket / Notify Analyst
```

SOAR is useful for predictable, repetitive processes. Automated response should be carefully designed because an incorrect action can affect legitimate business activity.

## 12.3 EDR

**Endpoint Detection and Response (EDR)** provides endpoint visibility, detection, investigation, and response capabilities.

Depending on the product, telemetry may include:

- Processes
- Parent-child process relationships
- Files
- Network connections
- User activity
- Persistence mechanisms
- Registry activity
- Command execution
- Endpoint security events

EDR is valuable during endpoint investigations because it can provide detailed information about what happened on a host.

## 12.4 XDR

**Extended Detection and Response (XDR)** extends detection and response across multiple security domains.

Depending on the platform, telemetry can include:

- Endpoints
- Identity
- Email
- Network
- Cloud
- Applications

The objective is to correlate security signals across multiple layers rather than investigate each security domain independently.

## 12.5 IDS

An **Intrusion Detection System (IDS)** identifies suspicious or malicious activity and generates alerts.

```text
Network Traffic
      ↓
     IDS
      ↓
Suspicious Pattern
      ↓
    ALERT
```

The primary purpose is detection and alerting.

## 12.6 IPS

An **Intrusion Prevention System (IPS)** can detect suspicious or malicious traffic and actively prevent or block it according to its configuration.

```text
IDS → Detect → Alert

IPS → Detect → Prevent / Block
```

Modern network-security platforms can combine IDS and IPS capabilities.

## 12.7 Firewall

A firewall controls network traffic according to security policies.

Policies may consider:

- Source IP
- Destination IP
- Source port
- Destination port
- Protocol
- Application
- User or identity
- Network zone

Firewall logs can provide evidence about:

- Allowed connections
- Denied connections
- Connection attempts
- Source and destination addresses
- Ports and protocols
- Scanning activity
- Suspicious outbound traffic

Firewall telemetry is therefore valuable for SIEM correlation and incident investigation.

## 12.8 VPN

A **Virtual Private Network (VPN)** provides a protected communication channel between users/devices and networks, depending on the VPN architecture.

From a SOC perspective, VPN logs can provide:

- Authentication attempts
- Successful authentication
- Failed authentication
- User identity
- Source IP
- Connection time
- Session duration
- Assigned address
- Gateway information

VPN telemetry can be particularly useful when investigating stolen credentials or unusual remote access.

---

# 13. How the Security Tools Work Together

A SOC should be viewed as an integrated security ecosystem rather than a collection of unrelated products.

```text
                           INTERNET
                               |
                               v
                           FIREWALL
                               |
                         +-----+-----+
                         |           |
                       IDS/IPS      VPN
                         |           |
                         +-----+-----+
                               |
                       ENTERPRISE NETWORK
                               |
          +--------------------+--------------------+
          |                    |                    |
      ENDPOINTS              SERVERS              CLOUD
          |                    |                    |
         EDR              OS / App Logs         Cloud Logs
          |                    |                    |
          +--------------------+--------------------+
                               |
                               v
                    LOG COLLECTOR / AGENTS
                               |
                               v
                              SIEM
                               |
               +---------------+---------------+
               |               |               |
             Alerts         Analytics      Investigation
               |                               |
               +---------------+---------------+
                               |
                               v
                              SOC
                               |
                               v
                             SOAR
                               |
                               v
                        RESPONSE ACTIONS
```

### Why integration matters

Suppose a firewall reports an outbound connection to an unusual IP address. The firewall alone may not provide enough context to determine whether the connection was malicious.

EDR may show that `powershell.exe` created the connection. Identity logs may show which user was logged in. DNS logs may show a suspicious domain lookup immediately before the connection. The SIEM can correlate these observations and provide the analyst with a much stronger investigation trail.

This demonstrates a core SOC principle:

> **Telemetry from different security layers provides context that an individual event cannot provide by itself.**

---

# 14. Example — From Raw Log to Security Incident

Consider a suspected brute-force attack.

## Step 1 — Raw Events

```text
10:15:01  Failed login  user=admin  source=10.10.10.50
10:15:04  Failed login  user=admin  source=10.10.10.50
10:15:07  Failed login  user=admin  source=10.10.10.50
10:15:10  Failed login  user=admin  source=10.10.10.50
```

Each record is an individual authentication event.

## Step 2 — Collection

The authentication system forwards or exposes the events to a collector or SIEM.

## Step 3 — Parsing

The system extracts:

```text
username   = admin
source.ip  = 10.10.10.50
action     = authentication
result     = failure
timestamp  = 10:15:01
```

## Step 4 — Normalization

The data is represented using the fields expected by the security analytics and detection layer.

## Step 5 — Correlation

The detection engine identifies repeated authentication failures from the same source within a defined time window.

## Step 6 — Alert

```text
ALERT: Possible brute-force activity
Source IP: 10.10.10.50
Target account: admin
Failed attempts: 20
Time window: 5 minutes
```

## Step 7 — L1 Triage

The analyst should not immediately declare an incident.

The analyst checks:

- Is the source IP expected?
- Is the account legitimate?
- Is the source internal or external?
- Is the user an administrator?
- Did a successful login occur afterward?
- Are other accounts being attacked?
- Is the source attacking multiple systems?
- What happened immediately before and after the alert?

## Step 8 — Investigation

Additional telemetry can be correlated:

```text
Authentication logs
       +
Endpoint telemetry
       +
Firewall logs
       +
VPN logs
       +
DNS logs
       +
Threat intelligence
```

Suppose the investigation finds:

```text
Multiple failed logins
        ↓
Successful authentication
        ↓
New privileged process
        ↓
Suspicious PowerShell activity
        ↓
Outbound connection to suspicious infrastructure
```

The combined evidence is significantly more concerning than the original failed-login events.

## Step 9 — Incident Decision

If the evidence meets the organization's incident criteria, the alert is escalated or classified as a security incident.

## Step 10 — Response

Depending on the confirmed activity and organizational authorization, response may include:

- Protecting or disabling the compromised account
- Revoking sessions
- Isolating an affected endpoint
- Blocking malicious infrastructure
- Resetting credentials
- Removing persistence
- Investigating additional systems
- Restoring affected systems

## Step 11 — Improvement

After the incident, the SOC can improve:

- Detection thresholds
- Correlation rules
- Threat-intelligence enrichment
- Investigation playbooks
- Alert context
- Endpoint telemetry
- Authentication monitoring

The complete transformation is:

```text
Raw Event
   ↓
Collected Telemetry
   ↓
Parsing
   ↓
Normalization
   ↓
Enrichment
   ↓
Correlation
   ↓
Detection
   ↓
Alert
   ↓
Triage
   ↓
Investigation
   ↓
Incident
   ↓
Response
   ↓
Recovery
   ↓
Improvement
```

---

# 15. Practical SOC Mindset

One of the most important lessons from Day 1 is to think in terms of **evidence, context, and investigation**, rather than simply trusting an alert label.

An analyst should not automatically think:

> "The tool generated a malicious alert, therefore the host is compromised."

Instead, the analyst should ask:

```text
What happened?
      ↓
When did it happen?
      ↓
Who was involved?
      ↓
Which asset was involved?
      ↓
Where did the activity originate?
      ↓
What happened before it?
      ↓
What happened after it?
      ↓
Is this expected behavior?
      ↓
What additional telemetry exists?
      ↓
What evidence supports the detection?
      ↓
What evidence contradicts it?
      ↓
What is the scope?
      ↓
What is the impact?
      ↓
Does it meet incident criteria?
      ↓
What response is required?
```

This investigative mindset is fundamental to SOC operations.

---

# 16. Important Concepts to Remember

## Alert ≠ Incident

An alert is a signal requiring attention. Investigation determines whether the signal represents meaningful malicious activity.

## More Logs ≠ Better Security

Collecting excessive telemetry without a detection and investigation strategy can create noise, storage requirements, and analyst fatigue. The objective is useful and reliable telemetry that supports actual security decisions.

## Context Matters

The same activity can be legitimate or malicious depending on the circumstances.

For example, PowerShell execution could be:

- Normal administration
- Software deployment
- Security tooling
- Malicious execution

The analyst needs context such as user, host, command line, parent process, timing, destination, and surrounding events.

## Correlation Increases Visibility

Multiple related events can reveal an attack sequence that would not be obvious from one event.

## Detection Is Only One Part of SOC Operations

A SOC must also triage, investigate, respond, document, recover, and improve.

---

# 17. Day 1 Knowledge Map

```text
                         SOC
                          |
        +-----------------+------------------+
        |                 |                  |
      PEOPLE           PROCESSES          TECHNOLOGY
        |                 |                  |
   L1 / L2 / L3      Monitor / Triage       SIEM
   Manager           Investigate            SOAR
                     Escalate               EDR
                     Respond                XDR
                     Recover                IDS/IPS
                     Document               Firewall
                     Improve                VPN
        |                 |                  |
        +-----------------+------------------+
                          |
                     TELEMETRY
                          |
                    Logs / Events
                          |
                     SIEM Pipeline
                          |
             Parse / Normalize / Enrich
                          |
                  Store / Correlate
                          |
                       Detect
                          |
                        Alert
                          |
                       Analyst
                          |
                    Investigation
                          |
                       Incident
                          |
                  Incident Response
                          |
                  Recovery / Lessons
                          |
                   Detection Improvement
```

---

# 18. Key Takeaways

1. A SOC is an **operational security function**, not merely a software product.
2. A SOC combines **people, processes, and technology**.
3. Continuous monitoring provides visibility across endpoints, servers, identities, networks, applications, and cloud environments.
4. Threat detection can use signatures, rules, correlation, behavioral analytics, threat intelligence, and anomaly detection.
5. L1 analysts generally focus on initial alert triage and escalation.
6. L2 analysts perform deeper investigation and cross-source correlation.
7. L3 analysts handle advanced investigation, threat hunting, detection engineering, malware analysis, and forensics.
8. SOC managers coordinate people, processes, metrics, strategy, and operational effectiveness.
9. A **log** is recorded information describing system or security activity.
10. An **event** represents activity that occurred.
11. An **alert** is a security signal requiring analyst attention.
12. An **incident** is activity that meets the organization's criteria for security response.
13. **An alert should not automatically be treated as an incident.**
14. Incident response provides a structured process for handling security incidents.
15. Disaster recovery focuses on restoring critical operations after major disruption.
16. A SIEM centralizes security telemetry and supports search, correlation, detection, alerting, analytics, and investigation.
17. SIEM processing commonly involves **collection → parsing → normalization → enrichment → storage/indexing → correlation/detection → alerting**.
18. Log management primarily focuses on collection, transport, storage, search, and retention, while SIEM adds security-focused detection and investigation capabilities.
19. Logs can be transferred using agents, forwarding, Syslog/network logging, APIs, file-based collection, collectors, and cloud integrations.
20. EDR provides endpoint visibility and response capabilities.
21. XDR correlates security telemetry across multiple security domains.
22. IDS primarily detects and alerts, while IPS can actively prevent or block activity.
23. Firewalls enforce network-access policies and generate valuable security telemetry.
24. VPN systems generate authentication and connection telemetry useful during investigations.
25. SOAR can automate repetitive investigation and response workflows.
26. No single security product provides complete visibility.
27. Effective SOC operations depend on **context, correlation, evidence, and disciplined investigation**.

---

# 19. Day 1 Learning Summary

Today I studied the fundamentals of Security Operations Center operations and how a SOC functions as an integrated security capability.

I learned that a SOC combines **people, processes, and technology** to continuously monitor an organization's environment, identify suspicious activity, investigate alerts, respond to incidents, document findings, and improve security operations over time.

I studied the responsibilities of L1, L2, L3, and SOC management. L1 focuses primarily on initial alert triage and escalation, L2 performs deeper investigation and correlation, L3 handles advanced investigations and specialized activities such as threat hunting, detection engineering, malware analysis, and digital forensics, while the SOC manager focuses on operational leadership, metrics, processes, staffing, and strategy.

I learned the operational difference between **logs, events, alerts, incidents, incident response, and disasters**. A log is recorded telemetry, an event represents activity that occurred, an alert is a security signal requiring attention, and an incident is activity that meets the organization's criteria for security response. A key lesson is that an alert is a starting point for investigation rather than automatic proof of compromise.

A major part of the day was understanding how a SIEM processes security data. Telemetry is collected from endpoints, servers, applications, network devices, firewalls, VPN systems, identity systems, cloud services, EDR, IDS/IPS, and other sources. The data can then be parsed, normalized, enriched, stored/indexed, and evaluated through detection and correlation logic. When detection criteria are satisfied, an alert is generated and enters the SOC investigation workflow.

I also studied the difference between **log management and SIEM**. Log management focuses primarily on collecting, transporting, storing, indexing, searching, and retaining logs, whereas SIEM adds security-focused correlation, detection, alerting, analytics, and investigation capabilities.

I learned that logs can be transferred through several mechanisms, including agent-based collection, log forwarding, network-based logging such as Syslog, APIs, file-based collection, intermediate collectors, and cloud-native integrations. The method depends on the architecture, source system, security requirements, and capabilities of the receiving platform.

Finally, I studied how SIEM, SOAR, EDR, XDR, IDS, IPS, firewalls, VPN systems, and other security technologies complement one another. The most important operational principle is that **security telemetry becomes significantly more useful when events from different sources are correlated and interpreted in context**.

The overall SOC workflow learned today is:

```text
Monitor
   ↓
Collect
   ↓
Process
   ↓
Detect
   ↓
Alert
   ↓
Triage
   ↓
Investigate
   ↓
Classify
   ↓
Respond
   ↓
Recover
   ↓
Document
   ↓
Improve
```

---

# 20. Reference Material

- Day 1 SOC fundamentals study material and diagrams provided during the learning session.
- Topics covered during the Day 1 practical study session.
