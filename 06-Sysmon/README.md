# Day 6 — Sysmon

## Objective

Sysmon (System Monitor) is a Microsoft Sysinternals tool that provides detailed Windows endpoint telemetry. It complements normal Windows Event Logs by recording process creation, network connections, DNS queries, file activity, registry activity, process access, driver loading, WMI activity, and other security-relevant behavior.

For a SOC analyst, Sysmon is valuable because it helps answer not only "what happened?" but also "which process caused it, who launched it, what command line was used, what did it connect to, and what happened next?"

---

## 1. What Is Sysmon?

Sysmon is a Windows system service and driver that monitors selected system activity and writes structured events to:

Microsoft-Windows-Sysmon/Operational

Sysmon is a telemetry tool. It is not an antivirus and does not automatically determine that a file is malicious.

The security workflow is:

System activity → Sysmon telemetry → SIEM/detection → SOC investigation

---

## 2. Why SOC Analysts Use Sysmon

Standard Windows Security logging provides important events such as successful logons and process creation. Sysmon can add richer endpoint context.

For example, instead of only seeing that PowerShell started, Sysmon can provide information such as:

- Process image
- Process ID
- Process GUID
- Parent process
- Parent command line
- User
- Integrity level
- Command line
- Hashes
- File metadata

When network telemetry is also enabled, the analyst can connect a process to a destination IP and port.

This makes Sysmon especially useful for threat hunting and incident response.

---

## 3. Sysmon Architecture

A simplified architecture is:

Windows activity
→ Sysmon driver/service
→ Sysmon event
→ Windows Event Log
→ Agent/Forwarder
→ SIEM
→ SOC analyst

Open Event Viewer with:

eventvwr.msc

Then navigate to:

Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational

---

## 4. Important Sysmon Event IDs

| Event ID | Activity |
|---:|---|
| 1 | Process creation |
| 2 | File creation time changed |
| 3 | Network connection |
| 4 | Sysmon service state changed |
| 5 | Process terminated |
| 6 | Driver loaded |
| 7 | Image/DLL loaded |
| 8 | CreateRemoteThread |
| 9 | RawAccessRead |
| 10 | Process access |
| 11 | File created |
| 12 | Registry object added/deleted |
| 13 | Registry value set |
| 14 | Registry key/value renamed |
| 15 | FileCreateStreamHash |
| 16 | Sysmon configuration changed |
| 17 | Named pipe created |
| 18 | Named pipe connected |
| 19 | WMI event filter |
| 20 | WMI event consumer |
| 21 | WMI consumer-to-filter binding |
| 22 | DNS query |
| 23 | File deleted |
| 24 | Clipboard change |
| 25 | Process tampering |
| 26 | File deleted |
| 27+ | Additional events depending on Sysmon version |
| 255 | Sysmon error |

Event availability depends on the installed Sysmon version and configuration.

---

## 5. Event ID 1 — Process Creation

Event ID 1 is one of the most important Sysmon events.

It can provide:

- Process GUID
- Process ID
- Image
- Command line
- Current directory
- User
- Logon ID
- Integrity level
- Hashes
- Parent process
- Parent command line
- Parent image
- File metadata

Example process chain:

explorer.exe
→ winword.exe
→ powershell.exe
→ cmd.exe
→ suspicious.exe

The process tree gives context. A suspicious executable should not be judged only by its filename. The analyst should examine its parent, command line, user, path, hash, signature, and network behavior.

---

## 6. Process ID vs Process GUID

A PID identifies a process while it is running, but PIDs can be reused.

Sysmon Process GUIDs help identify a particular process instance more reliably during correlation.

This is useful when connecting:

Process creation
→ Network connection
→ File creation
→ Process termination

The Process GUID can provide a stronger correlation key than a PID alone.

---

## 7. Command-Line Analysis

The process name alone often provides insufficient context.

For example:

powershell.exe

is ambiguous.

A command line such as:

powershell.exe -File C:\Scripts\maintenance.ps1

provides much more context.

A command line containing encoded data, unusual download parameters, unexpected URLs, or suspicious execution options may deserve investigation.

Command lines should always be interpreted with user, parent process, path, timing, and environment context.

---

## 8. Hashes and Digital Signatures

Sysmon can record file hashes depending on configuration.

Commonly encountered algorithms include:

- SHA1
- SHA256
- MD5
- IMPHASH

SHA256 is commonly preferred for modern file identification.

Hashes help analysts:

- Correlate the same file across endpoints
- Search threat-intelligence sources
- Track known artifacts
- Compare files

A hash is an identifier, not a verdict.

Digital signatures can also provide useful context. A valid signature does not automatically prove that software is safe, and an unsigned file is not automatically malicious.

---

## 9. Event ID 3 — Network Connection

Event ID 3 records network connections when enabled.

Useful fields can include:

- Source IP
- Source port
- Destination IP
- Destination port
- Protocol
- Process ID
- Process GUID
- User
- Image

Example:

10.10.10.25:52344
→ 203.0.113.50:443
→ PID 4820
→ powershell.exe

This lets an analyst connect network activity directly to an endpoint process.

---

## 10. Network-to-Process Correlation

A core Sysmon workflow is:

Network connection
→ Process GUID/PID
→ Process creation
→ Command line
→ User
→ Parent process
→ File/hash

Suppose a workstation connects to an unfamiliar external IP. The analyst can determine which process made the connection, who launched it, what command line was used, how it started, and whether the executable was recently created.

This is one of the most useful endpoint investigation techniques.

---

## 11. Event ID 22 — DNS Query

Event ID 22 records DNS queries when enabled.

It can provide:

- Query name
- Query type
- Query status
- Process
- PID
- Process GUID
- User

A useful correlation is:

Process
→ DNS query
→ destination IP
→ network connection

DNS telemetry can support investigation of suspicious domains, command-and-control infrastructure, unexpected external services, and unusual query behavior.

A domain should not be classified as malicious based only on its appearance.

---

## 12. Event ID 11 — File Created

Event ID 11 records file creation when configured.

Useful fields can include:

- File path
- File name
- Creating process
- User
- Timestamp
- Hash information

Example:

powershell.exe
→ creates
→ C:\Users\Alice\AppData\Local\Temp\test.exe

The analyst can then determine whether the created file was subsequently executed, connected to the network, or used for persistence.

A file in a temporary directory is not automatically malicious.

---

## 13. File Deletion and Timestamp Changes

Sysmon can provide visibility into file deletion and file creation-time changes.

File deletion may be relevant when an attacker removes tools or artifacts after execution.

File creation-time modification may be relevant to timestomping investigations.

A useful timeline is:

File created
→ Process executed
→ Network activity
→ File deleted

The analyst should correlate the file event with the responsible process and user.

---

## 14. Event ID 5 — Process Terminated

Event ID 5 records process termination.

This helps establish process lifetimes.

For example:

10:00:00 Process created
10:00:03 Network connection
10:00:20 Process terminated

Combining creation, network, and termination events can produce a much clearer endpoint timeline.

---

## 15. Event ID 6 — Driver Loaded

Event ID 6 records driver loading.

Drivers operate at a highly privileged level.

Investigate:

- Driver name
- Path
- Hash
- Signature
- Company
- Load time
- Associated software

Many legitimate security and hardware products install drivers, so a driver event should be investigated using context rather than treated as malicious automatically.

---

## 16. Event ID 7 — Image Loaded

Event ID 7 provides visibility into image/DLL loading when configured.

It can support investigations involving:

- Unexpected DLLs
- DLL side-loading
- Suspicious modules
- Sensitive-process activity

Image-load monitoring can generate significant telemetry, so configuration and filtering are important.

---

## 17. Event ID 8 — CreateRemoteThread

Event ID 8 records CreateRemoteThread activity.

Remote thread creation can be legitimate, but it can also appear in process-injection techniques.

Investigate:

- Source process
- Target process
- Source user
- Target process
- Start address
- Image
- Hash
- Related process activity

Never treat one event as automatic proof of injection.

---

## 18. Event ID 10 — Process Access

Event ID 10 records process-access activity when configured.

It can be valuable for investigating suspicious access to other processes.

Investigate:

- Source process
- Target process
- Granted access
- User
- Source Process GUID
- Target Process GUID
- Image paths

Security products and legitimate administration tools can also generate process-access events, so contextual analysis is essential.

---

## 19. Registry Events 12, 13 and 14

Sysmon can monitor registry activity.

Important events include:

- 12 — Registry object added/deleted
- 13 — Registry value set
- 14 — Registry key/value renamed

Registry telemetry can support investigations involving:

- Persistence
- Configuration changes
- Security-control modification
- Application behavior

For example, changes to Windows startup Run keys may deserve investigation.

---

## 20. Event ID 15 — FileCreateStreamHash

Event ID 15 records file stream hashing information associated with NTFS alternate data streams.

Alternate Data Streams have legitimate uses, but they can also be abused to store hidden information.

This is a specialized forensic and threat-hunting signal and should be interpreted with other endpoint evidence.

---

## 21. Event ID 16 — Sysmon Configuration Changed

Event ID 16 records Sysmon configuration changes.

This is important because configuration determines what telemetry is collected.

An unexpected configuration change should lead to questions such as:

- Who changed it?
- When?
- What changed?
- Was the change authorized?
- Did logging visibility decrease?

Protecting telemetry is part of maintaining detection capability.

---

## 22. Named Pipes and WMI

Sysmon can monitor named-pipe activity through Events 17 and 18.

It can also monitor WMI activity through Events 19, 20, and 21.

Named pipes and WMI are legitimate Windows technologies, but attackers may abuse them for execution, persistence, communication, or lateral movement.

These events are most valuable when correlated with process, user, network, and persistence telemetry.

---

## 23. Event ID 22 — DNS Query Investigation

A practical DNS investigation might look like:

powershell.exe
→ DNS query
→ suspicious-example.com
→ network connection
→ external IP

The analyst should determine:

- Which process performed the query?
- Which user owns the process?
- Was the domain expected?
- Did a network connection follow?
- What command line launched the process?
- Was a file created before the connection?

This turns a DNS indicator into an endpoint investigation.

---

## 24. Event ID 25 — Process Tampering

Event ID 25 records certain process-tampering behavior.

Process tampering can be associated with techniques intended to modify process state or evade security controls.

Investigate:

- Process involved
- User
- Parent process
- Target process
- Timing
- Related process-access events
- Other endpoint telemetry

---

## 25. Sysmon Configuration

Sysmon uses configuration rules to determine which events are collected and filtered.

A simplified configuration concept is:

<Sysmon>
  <EventFiltering>
    <ProcessCreate onmatch="include">
      ...
    </ProcessCreate>
    <NetworkConnect onmatch="include">
      ...
    </NetworkConnect>
  </EventFiltering>
</Sysmon>

The exact schema depends on the Sysmon version.

A good configuration balances:

Visibility + Performance + Storage + Detection requirements

Logging everything can produce excessive data, high SIEM ingestion costs, and investigation noise.

---

## 26. Installing Sysmon in a Lab

For an authorized Windows lab:

1. Obtain Sysmon from Microsoft Sysinternals.
2. Extract the package.
3. Open an elevated PowerShell or Command Prompt.
4. Review your configuration.
5. Install Sysmon.
6. Verify the Sysmon service.
7. Open Event Viewer.
8. Confirm the Sysmon Operational channel.
9. Generate benign activity.
10. Verify that expected events appear.

A typical installation command is:

sysmon64.exe -accepteula -i <config.xml>

Always verify the exact syntax for the installed version.

---

## 27. Verify the Sysmon Service

A common service query is:

sc query Sysmon64

Then verify:

Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational

If the service is installed but events are absent, investigate service status, configuration, permissions, filtering, and the event channel.

---

## 28. Updating Configuration

A typical configuration update uses:

sysmon64.exe -c <config.xml>

After changing the configuration:

- Verify the configuration
- Confirm the service remains operational
- Generate controlled test activity
- Verify expected events
- Check that important telemetry was not unintentionally filtered

---

## 29. Querying Sysmon with PowerShell

Recent Sysmon events:

Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -MaxEvents 20

Process creation:

Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=1} -MaxEvents 20

Network connections:

Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=3} -MaxEvents 20

DNS queries:

Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=22} -MaxEvents 20

---

## 30. Sysmon Event Correlation

The biggest advantage of Sysmon is correlation.

A single process may produce:

Event 1 → Process Created
Event 22 → DNS Query
Event 3 → Network Connection
Event 11 → File Created
Event 5 → Process Terminated

The Process GUID can help associate related events.

This can create a detailed endpoint timeline.

---

## 31. Example Investigation

Suppose an alert reports that a workstation connected to an unfamiliar external IP.

### Step 1 — Find Event 3

Identify destination IP, destination port, process, PID, and Process GUID.

### Step 2 — Find Event 1

Identify:

- Image
- Command line
- User
- Parent process
- Hash

### Step 3 — Review Event 22

Determine whether the process performed a DNS lookup immediately before the connection.

### Step 4 — Review Event 11

Determine whether the process created a file.

### Step 5 — Review persistence

Investigate relevant:

- Registry activity
- Services
- Scheduled tasks
- WMI

### Step 6 — Build the timeline

User activity
→ Process creation
→ DNS query
→ Network connection
→ File creation
→ Persistence

This is a practical Sysmon investigation workflow.

---

## 32. Sysmon and MITRE ATT&CK

Sysmon telemetry can support investigation of many ATT&CK technique areas.

Examples include:

| Technique area | Useful telemetry |
|---|---|
| Command and Scripting Interpreter | Process creation and command line |
| PowerShell | Process creation + PowerShell logs |
| Scheduled Task/Job | Task activity |
| Windows Service | Service installation |
| Process Injection | CreateRemoteThread / Process Access |
| Registry Run Keys | Registry events |
| WMI | WMI events |
| Network Communication | Network connection |
| DNS | DNS query |
| Indicator Removal | File deletion / log activity |

Sysmon does not automatically determine the ATT&CK technique. Analysts map observed behavior to techniques using context.

---

## 33. Sysmon and Threat Hunting

Example PowerShell hunt:

PowerShell process
→ Identify parent
→ Review command line
→ Review DNS
→ Review external connections
→ Review file creation

Example service hunt:

New service
→ Check executable
→ Check creating process
→ Check account
→ Check network activity

Sysmon provides the raw telemetry needed for these searches.

---

## 34. Sysmon and SIEM

A typical SOC architecture is:

Windows Endpoint
→ Sysmon
→ Sysmon Operational Log
→ Wazuh / Splunk / Elastic / Sentinel
→ Parsing
→ Detection Rules
→ Alert
→ SOC Analyst

Example detection logic:

Sysmon Event 1
AND PowerShell
AND unusual parent process
AND external network connection

This should be treated as a detection candidate rather than automatic proof of compromise.

---

## 35. Sysmon vs Windows Security Events

| Capability | Windows Security | Sysmon |
|---|---|---|
| Successful logon | Strong | Contextual |
| Failed logon | Strong | Not primary |
| Process creation | 4688 | Event 1 |
| Parent process | Available when configured | Detailed |
| Command line | Requires appropriate auditing | Detailed process telemetry |
| Network connection | Not primary | Event 3 |
| DNS query | Not primary | Event 22 |
| File creation | Limited | Event 11 |
| Registry activity | Limited | Events 12–14 |
| Process access | Limited | Event 10 |
| Image/DLL loading | Limited | Event 7 |
| Driver loading | Limited | Event 6 |

The strongest endpoint investigations generally correlate multiple telemetry sources rather than depending on only one.

---

## 36. False Positives

Detailed telemetry also creates noise.

Examples:

- Security software accessing processes
- Browsers creating temporary files
- Installers modifying registry keys
- Windows services creating files
- Administrators using PowerShell
- Applications generating many DNS requests

Detection logic should consider:

- Parent process
- User
- Path
- Signature
- Hash
- Destination
- Frequency
- Time
- Host role
- Business context

---

## 37. Practical Lab 1 — Verify Sysmon

Run:

sc query Sysmon64

Then open Event Viewer and inspect the Sysmon Operational channel.

Confirm that events are being generated.

---

## 38. Practical Lab 2 — Process Creation

Launch benign programs such as:

notepad.exe
cmd.exe
powershell.exe

Find Event ID 1.

Document:

- Image
- PID
- Process GUID
- Parent Image
- Command Line
- User
- Hash

---

## 39. Practical Lab 3 — Network Connection

Generate normal network traffic in an authorized lab.

Search for Event ID 3.

Document:

- Source IP
- Destination IP
- Destination port
- Protocol
- Image
- PID
- Process GUID

Then correlate the connection with Event ID 1.

---

## 40. Practical Lab 4 — DNS Investigation

Perform a normal DNS lookup:

nslookup example.com

Search Sysmon for Event ID 22.

Document:

- Query name
- Query type
- Process
- PID
- Process GUID
- Timestamp

Then correlate DNS activity with network activity.

---

## 41. Practical Lab 5 — File Creation

Create a benign test file or copy a test file.

Search for Event ID 11.

Document:

- File path
- Creating process
- User
- Timestamp
- Hash information where available

Understand the difference between file creation and file execution.

---

## 42. Practical Lab 6 — Registry Activity

In an authorized lab, make a harmless registry change under a dedicated test key.

Search for:

- Event ID 12
- Event ID 13
- Event ID 14

Document which operation generated which event.

Do not modify security-critical registry locations unless the lab specifically requires it.

---

## 43. Practical Lab 7 — Build a Process Timeline

Select one benign process and identify available events:

Event 1 → Process Created
Event 3 → Network Connection
Event 22 → DNS Query
Event 11 → File Created
Event 5 → Process Terminated

Not every process will generate every event.

The objective is to understand how multiple event types describe one process lifecycle.

---

## 44. Practical Lab 8 — Basic Threat Hunting

Use PowerShell to identify recent Sysmon process-creation events and review those containing powershell.exe.

Example approach:

Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=1} -MaxEvents 100

Review the event details and identify:

- Parent process
- Command line
- User
- Path
- Hash

---

## 45. Practical Lab 9 — Network-to-Process Hunt

Search recent Event ID 3 records.

For each interesting connection:

1. Record destination IP.
2. Record destination port.
3. Record PID/Process GUID.
4. Find the associated Event ID 1.
5. Identify the process.
6. Identify the user.
7. Review the parent process.
8. Review related DNS activity.

This teaches endpoint network correlation.

---

## 46. Practical Lab 10 — Detection Engineering

Design a hypothetical detection:

Sysmon Event 1
+
PowerShell
+
Unexpected Parent Process
+
External Network Connection

Define:

- Detection purpose
- Required telemetry
- Fields
- Filtering conditions
- Expected false positives
- Investigation steps

The exercise is to learn detection design, not to assume that the pattern automatically represents compromise.

---

## 47. Sysmon Investigation Checklist

### Process

- What process?
- What path?
- What hash?
- What signer?
- What command line?

### Identity

- Which user?
- Which logon session?
- What privileges?

### Lineage

- Who launched it?
- What child processes did it create?

### Network

- Which destination?
- Which port?
- Which protocol?
- Which process created the connection?

### DNS

- What domain was queried?
- Which process made the query?

### Files

- What files were created?
- What files were deleted?
- Were timestamps changed?

### Persistence

- Registry?
- Service?
- Scheduled task?
- WMI?

### Scope

- Same hash elsewhere?
- Same destination elsewhere?
- Same user on other hosts?

---

## 48. Key Concepts Learned

By completing Day 6, you should understand:

- What Sysmon is
- Why SOC teams use Sysmon
- Sysmon architecture
- Sysmon Operational channel
- Sysmon configuration
- Event ID 1 — Process Creation
- Event ID 3 — Network Connection
- Event ID 5 — Process Terminated
- Event ID 6 — Driver Loaded
- Event ID 7 — Image Loaded
- Event ID 8 — CreateRemoteThread
- Event ID 10 — Process Access
- Event ID 11 — File Created
- Event IDs 12–14 — Registry activity
- Event ID 15 — FileCreateStreamHash
- Event ID 16 — Configuration Changed
- Event IDs 17–18 — Named Pipes
- Event IDs 19–21 — WMI
- Event ID 22 — DNS Query
- Event ID 23/26 — File Deletion depending on version
- Event ID 24 — Clipboard
- Event ID 25 — Process Tampering
- Process GUID correlation
- Hash and signature analysis
- Process-tree analysis
- Network-to-process correlation
- DNS-to-process correlation
- File-to-process correlation
- SIEM integration
- Threat hunting
- Detection engineering
- MITRE ATT&CK mapping
- False-positive reduction

---

## Day 6 Outcome

Day 6 established Sysmon as a detailed endpoint telemetry layer for SOC operations.

The progression is:

Windows Fundamentals
→ Windows Event Logs
→ Sysmon
→ Detailed Endpoint Telemetry
→ Threat Hunting
→ Detection Engineering
→ SIEM Correlation

The most important skill is not memorizing every Event ID. It is learning to connect **process, user, file, DNS, network, registry, and persistence telemetry into one coherent endpoint investigation**.

**Status: ✅ Completed**

**Next: Day 7 — SIEM Fundamentals and Log Analysis.**
