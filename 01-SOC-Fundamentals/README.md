# Day 1 — SOC Fundamentals

**Focus:** Security Operations Center (SOC) fundamentals, security monitoring, security tooling, log management, and the flow from raw telemetry to alerts and incidents.

## Objective

Understand how a modern SOC operates, what a SOC is responsible for, how security telemetry moves from monitored systems into a SIEM, how analysts work with events and alerts, and how incidents progress through investigation and response.

---

## 1. What is a SOC?

A **Security Operations Center (SOC)** is a centralized security function responsible for continuously monitoring an organization's digital environment, detecting suspicious or malicious activity, investigating security alerts, responding to incidents, and improving the organization's security posture.

A SOC typically brings together:

- People — analysts, engineers, incident responders, threat hunters, and management
- Processes — monitoring, triage, investigation, escalation, response, recovery, and continuous improvement
- Technology — SIEM, EDR/XDR, IDS/IPS, firewalls, VPN infrastructure, SOAR, threat-intelligence platforms, and other security controls

The SOC acts as an operational security layer between an organization's infrastructure and the threats targeting it.

### High-level SOC flow

```text
Network / Cloud / Users / Endpoints / Applications
                        |
                        v
              Logs + Security Telemetry
                        |
                        v
                Log Management / SIEM
                        |
             Collection + Parsing
                        |
              Normalization + Enrichment
                        |
                 Correlation / Rules
                        |
                        v
                     Alerts
                        |
                        v
                   SOC Analyst
                        |
             Triage + Investigation
                        |
             +----------+----------+
             |                     |
          Benign / FP          Suspicious
                                   |
                                   v
                                Incident
                                   |
                                   v
                       Incident Response
                                   |
                                   v
                         Recovery + Lessons
                                   |
                                   v
                         Detection Improvement
```

---

## 2. Key Functions of a SOC

### 2.1 Continuous Monitoring

A SOC monitors systems, networks, applications, identities, endpoints, and cloud environments continuously to identify abnormal or suspicious activity.

Examples include:

- Repeated failed authentication attempts
- Unusual outbound network connections
- Suspicious process execution
- Privilege changes
- Unexpected administrative activity
- Malware detections
- Changes to critical files or configurations

### 2.2 Threat Detection

The SOC analyzes collected telemetry to identify indicators of compromise, malicious behavior, policy violations, and other security threats.

Detection can use:

- Detection rules
- Correlation logic
- Signatures
- Behavioral analytics
- Threat intelligence
- Statistical or anomaly-based methods

### 2.3 Incident Response

When activity is confirmed or strongly suspected to be malicious, the SOC investigates, contains, eradicates, and supports recovery from the incident.

### 2.4 Reporting and Documentation

Security investigations need an evidence trail. Analysts document:

- What happened
- When it happened
- Which assets and accounts were involved
- What evidence was found
- What actions were taken
- What the impact was
- What remediation is required

### 2.5 Continuous Improvement

A mature SOC uses lessons from incidents, false positives, investigations, and operational metrics to improve detections, processes, playbooks, and security controls.

---

## 3. How a SOC Works

A SOC is not simply a dashboard where analysts watch alerts. It is an operational pipeline.

```text
Telemetry
   ↓
Collection
   ↓
Parsing / Normalization
   ↓
Enrichment
   ↓
Detection / Correlation
   ↓
Alert Generation
   ↓
Triage
   ↓
Investigation
   ↓
Incident Classification
   ↓
Response
   ↓
Recovery
   ↓
Lessons Learned
   ↓
Detection / Process Improvement
```

The goal is to turn large volumes of raw technical data into **actionable security information**.

---

## 4. SOC Roles

### L1 — Tier 1 SOC Analyst

The first operational layer.

Typical responsibilities:

- Monitor security alerts
- Perform initial triage
- Validate whether an alert appears legitimate
- Identify obvious false positives
- Gather basic contextual information
- Escalate suspicious or confirmed incidents
- Follow documented playbooks

The L1 analyst is generally focused on **alert handling and initial investigation**.

### L2 — Tier 2 SOC Analyst

Handles more complex investigations that require deeper analysis.

Typical responsibilities:

- Investigate escalated alerts
- Correlate activity across multiple systems
- Analyze authentication, endpoint, network, and application telemetry
- Determine attack scope and potential impact
- Perform deeper threat analysis
- Recommend or initiate containment actions according to organizational procedures

### L3 — Tier 3 / Senior Analyst

Handles advanced investigations and specialized security operations.

Typical responsibilities:

- Advanced incident investigation
- Threat hunting
- Detection engineering
- Malware and forensic analysis
- Complex attack-chain analysis
- Improving detection coverage
- Developing advanced investigation techniques
- Supporting major incident response

### SOC Manager

Responsible for operational leadership and overall SOC effectiveness.

Typical responsibilities:

- Team management
- Incident escalation management
- SOC processes and procedures
- Metrics and reporting
- Staffing and capability planning
- Risk and compliance coordination
- Tooling and operational strategy
- Stakeholder communication

> In real organizations, role boundaries vary. Titles and responsibilities depend on the organization's size, operating model, and maturity.

---

## 5. Logs, Events, Alerts, and Incidents

These terms are related but should not be treated as interchangeable.

### Log

A **log** is a recorded piece of information generated by a system, application, network device, security control, or service.

Examples:

```text
User authentication succeeded
User authentication failed
Process started
Firewall connection allowed
Firewall connection denied
DNS query performed
VPN connection established
```

### Event

An **event** represents something that happened in a system or environment. A log record is often the stored representation of that event.

For example:

```text
Event: A user failed authentication
Time: 10:15:22
User: alice
Source IP: 10.10.10.25
```

### Alert

An **alert** is generated when security tooling identifies activity that matches a detection condition or otherwise requires analyst attention.

Example:

```text
5 failed logins for the same account within 60 seconds
                 ↓
             Detection rule
                 ↓
               ALERT
```

An alert is **not automatically an incident**. An analyst must investigate the context and determine its significance.

### Incident

A **security incident** is a confirmed or sufficiently credible security event that requires response according to the organization's incident-management criteria.

Example:

```text
Multiple failed logins
        ↓
Successful login from unusual source
        ↓
Suspicious PowerShell execution
        ↓
Outbound connection to suspicious IP
        ↓
Confirmed compromise
        ↓
SECURITY INCIDENT
```

### Incident Response

**Incident response (IR)** is the structured process used to prepare for, detect, analyze, contain, eradicate, and recover from security incidents.

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

### Disaster

A **disaster** is a major disruptive event that significantly affects business operations or critical services. A cyberattack can contribute to a disaster, but not every security incident is a disaster.

Disaster recovery focuses on restoring critical business services and technology after a major disruption.

---

## 6. SIEM — Security Information and Event Management

A **SIEM** centralizes security-relevant telemetry and provides capabilities for collection, storage, search, analysis, correlation, alerting, and investigation.

### How a SIEM processes data

```text
Data Sources
    |
    +--> Windows
    +--> Linux
    +--> Firewalls
    +--> IDS/IPS
    +--> EDR
    +--> VPN
    +--> Cloud
    +--> Applications
    |
    v
Log Collection
    |
    v
Parsing
    |
    v
Normalization
    |
    v
Enrichment
    |
    v
Storage / Indexing
    |
    v
Rules / Correlation / Analytics
    |
    v
Alerts
    |
    v
SOC Investigation
```

### 6.1 Collection

The SIEM receives logs and telemetry from multiple sources.

Collection can happen through agents, collectors, APIs, network protocols, cloud integrations, or other ingestion mechanisms.

### 6.2 Parsing

Raw logs often arrive as unstructured text or vendor-specific formats. Parsing extracts useful fields.

Example:

```text
Raw:
Failed login for user admin from 10.10.10.50

Parsed:
username = admin
source_ip = 10.10.10.50
action = failed_login
```

### 6.3 Normalization

Different products can describe the same activity differently. Normalization maps data into a consistent structure so detection logic can work across sources.

For example:

```text
Vendor A: src_ip
Vendor B: sourceAddress
Vendor C: client_ip

Normalized field:
source.ip
```

### 6.4 Enrichment

Additional context can be attached to an event, such as:

- Asset information
- User information
- GeoIP data
- Threat-intelligence matches
- Domain reputation
- Vulnerability context
- Identity information

### 6.5 Correlation and Detection

The SIEM can correlate multiple events to identify patterns that are more meaningful than individual events.

Example:

```text
10 failed logins
      +
Successful login
      +
New privileged process
      +
Suspicious outbound connection
      ↓
Potential account compromise
```

### 6.6 Alert Generation

When detection logic matches, the SIEM creates an alert containing relevant context for analysts.

The SOC then performs triage and investigation.

---

## 7. SIEM vs Log Management

### Log Management System

A log management platform primarily focuses on collecting, transporting, storing, indexing, searching, and retaining logs.

### SIEM

A SIEM builds on centralized log management by adding security-focused capabilities such as:

- Detection rules
- Correlation
- Security analytics
- Alerting
- Investigation workflows
- Threat intelligence integration
- Security monitoring

The boundary is not absolute. Modern platforms increasingly combine log management, SIEM, analytics, and response capabilities.

---

## 8. How Logs Are Transferred

Logs can move from a source to a centralized platform through several mechanisms.

### 8.1 Log Forwarding

A source forwards logs to a centralized collector or SIEM.

```text
Endpoint
   |
   | Log forwarding
   v
Collector / SIEM
```

Examples include agent-based forwarding and network-based log forwarding.

### 8.2 Agent-Based Collection

A lightweight agent runs on the endpoint and reads selected logs before securely forwarding them to a central platform.

```text
Windows / Linux Endpoint
        |
      Agent
        |
        v
   SIEM / Collector
```

Advantages include endpoint visibility and controlled collection.

### 8.3 Network-Based Logging

Some network and security devices can send logs to a central collector using standardized logging mechanisms such as Syslog.

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

### 8.4 API-Based Collection

A SIEM or collector can periodically retrieve events from a service using an API.

```text
Cloud / SaaS / Security Platform
             |
            API
             |
             v
            SIEM
```

### 8.5 Log Copy / File-Based Collection

Some systems write logs to files that can be copied, synchronized, or collected by an agent or scheduled process.

```text
Application Log File
        |
        v
Collector / Agent
        |
        v
SIEM
```

The exact method depends on the data source, platform, network architecture, security requirements, and operational constraints.

---

## 9. Security Tools in a SOC

A SOC normally uses multiple security technologies. Each provides a different control or visibility layer.

### SIEM

Centralized security telemetry, correlation, detection, alerting, search, and investigation.

### SOAR — Security Orchestration, Automation and Response

Automates repetitive SOC workflows and coordinates actions across security tools.

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
If malicious
    ↓
Block IP / Isolate Host / Create Ticket
```

### EDR — Endpoint Detection and Response

Provides endpoint telemetry, detection, investigation, and response capabilities.

Typical visibility includes:

- Processes
- Files
- Network connections
- User activity
- Persistence mechanisms
- Endpoint security events

### XDR — Extended Detection and Response

Extends detection and response beyond a single endpoint data source by correlating telemetry across multiple security domains such as endpoints, identity, email, network, and cloud.

### IDS — Intrusion Detection System

Detects suspicious or malicious network or host activity and generates alerts.

### IPS — Intrusion Prevention System

Performs detection and can actively block or prevent detected malicious traffic.

### Firewall

Controls network traffic based on defined security policies.

Common policy dimensions include:

- Source
- Destination
- Port
- Protocol
- Application
- Identity

### VPN — Virtual Private Network

Provides an encrypted or otherwise protected communication channel between endpoints and networks, depending on the VPN technology and configuration.

From a SOC perspective, VPN logs can provide valuable authentication, source, destination, and connection telemetry.

---

## 10. Putting the Security Tools Together

A simplified enterprise security architecture can look like this:

```text
                         Internet
                            |
                         Firewall
                            |
                    +-------+-------+
                    |               |
                   IDS/IPS        VPN
                    |               |
                    +-------+-------+
                            |
                 Enterprise Network
                            |
        +-------------------+-------------------+
        |                   |                   |
     Endpoints           Servers             Cloud
        |                   |                   |
       EDR              Sysmon/Logs         Cloud Logs
        |                   |                   |
        +-------------------+-------------------+
                            |
                            v
                    Log Collector / Agent
                            |
                            v
                           SIEM
                            |
              +-------------+-------------+
              |             |             |
           Alerts       Analytics     Investigation
              |
              v
             SOC
              |
              v
            SOAR
              |
       Response Actions
```

No single security product provides complete visibility. The SOC combines telemetry and capabilities from multiple controls to build a broader security picture.

---

## 11. Alert-to-Incident Example

Consider a suspected brute-force attack.

### Step 1 — Raw Events

```text
Failed login — user admin — 10.10.10.50
Failed login — user admin — 10.10.10.50
Failed login — user admin — 10.10.10.50
Failed login — user admin — 10.10.10.50
```

### Step 2 — Detection

The SIEM identifies repeated authentication failures from the same source.

### Step 3 — Alert

```text
ALERT: Possible brute-force activity
Source: 10.10.10.50
Target: admin
Failures: 20
```

### Step 4 — Triage

The analyst asks:

- Is the source expected?
- Is the target account legitimate?
- Did authentication eventually succeed?
- Is the source internal or external?
- Are other accounts affected?
- What happened immediately before and after the alert?

### Step 5 — Investigation

Additional telemetry is correlated from authentication logs, endpoint telemetry, firewall logs, VPN logs, and threat intelligence.

### Step 6 — Incident Decision

If evidence supports malicious activity, the alert is escalated or classified as a security incident according to organizational procedures.

### Step 7 — Response

Potential actions may include account protection, source blocking, endpoint isolation, credential reset, eradication, and recovery—depending on the confirmed attack and authorization model.

### Step 8 — Improvement

The SOC reviews the detection and investigation to determine whether rules, thresholds, enrichment, or playbooks should be improved.

---

## 12. Key Takeaways

1. A SOC combines **people, processes, and technology** to operate security monitoring and response.
2. The SOC lifecycle moves from **telemetry → detection → alert → triage → investigation → incident response → improvement**.
3. A **log** is raw recorded telemetry; an **event** represents activity that occurred; an **alert** requires analyst attention; an **incident** is a confirmed or sufficiently credible security situation requiring response.
4. A SIEM centralizes and processes security telemetry so analysts can search, correlate, detect, and investigate activity.
5. Log processing commonly involves **collection, parsing, normalization, enrichment, storage/indexing, correlation, and detection**.
6. Log transfer can use agents, forwarding, Syslog, APIs, file-based collection, and other integration mechanisms.
7. SIEM, SOAR, EDR, XDR, IDS/IPS, firewalls, VPN systems, and other tools provide complementary visibility and control.
8. An alert should not automatically be treated as an incident. **Context and investigation matter.**
9. A mature SOC continuously improves its detections, processes, and response capabilities.

---

## 13. Day 1 Learning Summary

Today I focused on understanding the SOC as an operational security function rather than just a collection of tools. I studied the SOC's major responsibilities, analyst tiers, the distinction between logs, events, alerts, incidents, incident response, and disasters, and the role of SIEM and other security technologies.

I also studied the end-to-end movement of security telemetry: how logs are collected from endpoints, servers, network devices, applications, and cloud systems; how they are parsed and normalized; how additional context can be added; and how correlation and detection logic can turn raw telemetry into actionable alerts for SOC analysts.

The main operational concept learned today is that **effective SOC operations depend on converting high-volume raw telemetry into reliable, contextualized, and actionable security decisions.**

---

## 14. Reference Material

- Day 1 SOC fundamentals study material and diagrams
- Practical SOC and SIEM concepts studied during the session
