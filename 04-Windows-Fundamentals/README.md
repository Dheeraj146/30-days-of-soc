# Day 4 — Windows Fundamentals

## 🎯 Objective

The objective of Day 4 is to build a strong practical foundation in **Windows operating systems from a SOC analyst and security investigation perspective**.

Windows is one of the most important operating-system environments in enterprise networks. Workstations, laptops, application servers, domain controllers, file servers, and business applications commonly run Windows. A SOC analyst therefore needs to understand how Windows manages users, processes, services, files, networking, authentication, privileges, configuration, and system activity.

This day focuses on understanding the operating system before moving into deeper topics such as Windows Event Logs, Sysmon, Active Directory attacks, detection engineering, and incident response.

The goal is not simply to memorize commands. The goal is to understand **what happens inside Windows, where evidence is generated, how an analyst can collect that evidence, and how different artifacts can be correlated during an investigation**.

---

## 1. What Is Windows?

Windows is a family of operating systems developed by Microsoft. Modern Windows systems used in organizations include Windows 10, Windows 11, and Windows Server editions.

Windows provides services for:

- User management
- Process execution
- Memory management
- File and storage management
- Networking
- Authentication
- Security controls
- Application execution
- Device management
- System configuration

From a SOC perspective, Windows is especially important because enterprise environments generate large amounts of Windows security telemetry.

A compromised Windows endpoint can provide evidence about:

- The user involved
- Processes executed
- Network connections
- Files created or modified
- Services started
- Authentication activity
- Privilege changes
- PowerShell execution
- Scheduled tasks
- Persistence mechanisms

---

## 2. Windows Architecture

A simplified Windows architecture is:

~~~text
User Applications
        ↓
Windows APIs / Subsystems
        ↓
Executive Services
        ↓
Windows Kernel
        ↓
Hardware
~~~

Windows broadly separates activity into **user mode** and **kernel mode**.

### User mode

Applications normally execute in user mode with restricted privileges.

Examples include:

- Web browsers
- Office applications
- PowerShell
- Command Prompt
- Security tools
- Custom applications

### Kernel mode

Kernel-mode components have highly privileged access to system resources.

They are responsible for areas such as:

- Process and thread management
- Memory management
- Hardware interaction
- Device drivers
- Low-level security operations

### SOC relevance

A malicious application normally begins execution in user mode, but attackers may attempt privilege escalation or exploitation of kernel-level vulnerabilities. Understanding the distinction helps analysts interpret process and security telemetry.

---

## 3. Windows Kernel

The Windows kernel is the core component that manages fundamental operating-system operations.

It coordinates:

- CPU scheduling
- Memory
- Processes
- Threads
- Device drivers
- Hardware interaction
- System calls

Applications request operating-system functionality through Windows APIs and system mechanisms rather than directly controlling hardware.

### Security relevance

Many security events ultimately involve operations performed by or through the operating system. Process creation, file access, network activity, authentication, and service execution can all produce telemetry that security tools collect.

---

## 4. Windows User Accounts

Windows supports multiple types of accounts:

- Local user accounts
- Local administrator accounts
- Domain user accounts
- Domain administrator accounts
- Service accounts
- Built-in system accounts

Useful commands include:

~~~cmd
whoami
whoami /user
whoami /groups
whoami /priv
~~~

These commands help determine the current identity, security identifier, group memberships, and privileges.

### SOC relevance

Identity is one of the most important dimensions of a security investigation.

When an alert occurs, analysts often need to establish:

~~~text
Which user?
      ↓
Which host?
      ↓
Which process?
      ↓
Which action?
      ↓
Which resource?
~~~

---

## 5. Security Identifier — SID

Windows identifies security principals using **Security Identifiers (SIDs)**.

A SID looks similar to:

~~~text
S-1-5-21-...
~~~

A SID can identify:

- Users
- Groups
- Computers
- Other security principals

A username can change, but the SID is the security identity used by Windows for authorization decisions.

### SOC relevance

Security logs may contain usernames and SIDs. Understanding SIDs helps analysts correlate authentication and authorization activity even when account naming becomes confusing.

---

## 6. Local Accounts vs Domain Accounts

A **local account** exists on a particular Windows machine.

A **domain account** is managed centrally through Active Directory.

For example:

~~~text
Local:
COMPUTER01\Alice

Domain:
CORP\Alice
~~~

The same username can exist locally and in a domain but represent different security principals.

### SOC relevance

During an investigation, analysts must distinguish between local and domain identities because the authentication path, privileges, and scope may be different.

This becomes especially important during lateral-movement investigations.

---

## 7. Windows Groups

Groups allow administrators to assign permissions to multiple users.

Examples include:

- Administrators
- Users
- Remote Desktop Users
- Backup Operators
- Event Log Readers

To inspect local groups:

~~~cmd
net localgroup
~~~

To inspect a specific group:

~~~cmd
net localgroup Administrators
~~~

### SOC relevance

Unexpected membership in a privileged group can be a major investigation lead.

If a standard user suddenly becomes a member of the local Administrators group, the analyst should determine who made the change, when it happened, whether it was authorized, and what happened afterward.

---

## 8. Windows Privileges

Windows privileges determine whether a security principal can perform certain sensitive operations.

Examples include privileges associated with:

- Loading drivers
- Debugging processes
- Backing up files
- Restoring files
- Changing system time
- Impersonating security principals

Inspect current privileges with:

~~~cmd
whoami /priv
~~~

### SOC relevance

Privilege escalation investigations frequently require understanding how a process obtained additional permissions.

---

## 9. Access Tokens

Windows uses **access tokens** to represent the security context of a process or thread.

A token can contain:

- User SID
- Group SIDs
- Privileges
- Integrity information
- Other security attributes

When a process attempts to access a protected resource, Windows uses the security context associated with that process to determine whether the requested operation is allowed.

### SOC relevance

Access tokens help explain why one process can perform an action that another process cannot.

This is important when investigating:

- Privilege escalation
- Token theft
- Impersonation
- Administrative activity
- Security-boundary bypass attempts

---

## 10. Windows Integrity Levels

Windows uses integrity levels as part of its security model.

Common levels include:

- Low
- Medium
- High
- System

A typical standard desktop application may run at medium integrity, while an elevated administrator process may run at high integrity.

The System account commonly operates with system-level authority.

### Security relevance

If an application running at a lower integrity level attempts to perform an operation requiring higher privileges, Windows security controls may prevent the action.

SOC analysts can use integrity information as additional context when examining suspicious process activity.

---

## 11. Windows Filesystem

Modern Windows systems commonly use **NTFS** as the filesystem.

The primary system volume is often represented as:

~~~text
C:\
~~~

Important directories include:

~~~text
C:\Windows
C:\Users
C:\Program Files
C:\Program Files (x86)
C:\ProgramData
C:\Windows\System32
C:\Windows\Temp
~~~

The exact directory structure can vary depending on system configuration.

---

## 12. Important Windows Directories

### C:\Windows

Contains Windows operating-system components.

### C:\Windows\System32

Contains many important Windows executables, libraries, drivers, and system components.

Despite the name, System32 exists on both 32-bit and 64-bit Windows installations and contains many 64-bit system components on 64-bit systems.

### C:\Users

Contains user profiles.

Example:

~~~text
C:\Users\Alice
C:\Users\Administrator
~~~

### C:\Program Files

Common location for installed 64-bit applications on 64-bit Windows.

### C:\Program Files (x86)

Common location for 32-bit applications on 64-bit Windows.

### C:\ProgramData

Contains application data shared across users.

### C:\Windows\Temp

Contains temporary system files.

### SOC relevance

Suspicious executables, scripts, archives, and configuration files may appear in temporary directories or user-writable locations. Analysts should investigate unusual files based on context rather than assuming that every file in a temporary directory is malicious.

---

## 13. Windows Registry

The Windows Registry is a hierarchical configuration database used by Windows and applications.

Major registry root keys include:

- HKEY_LOCAL_MACHINE — HKLM
- HKEY_CURRENT_USER — HKCU
- HKEY_CLASSES_ROOT — HKCR
- HKEY_USERS — HKU
- HKEY_CURRENT_CONFIG — HKCC

The registry stores information related to:

- Operating-system configuration
- Applications
- Hardware
- Users
- Services
- Security settings
- Startup configuration

### SOC relevance

The registry is extremely important in Windows investigations because attackers may abuse registry locations for:

- Persistence
- Configuration changes
- Execution
- Security-control modification
- Evidence concealment

---

## 14. Registry Hives

Important registry hive files include:

~~~text
SYSTEM
SOFTWARE
SAM
SECURITY
NTUSER.DAT
~~~

User-specific registry data is commonly associated with the **NTUSER.DAT** hive.

### SOC relevance

Registry artifacts can provide historical evidence about configuration and user activity. During forensic investigations, analysts may acquire registry hives and analyze them using specialized forensic tools.

---

## 15. Windows Processes

A process is an executing instance of a program.

Useful commands include:

~~~cmd
tasklist
tasklist /v
~~~

PowerShell:

~~~powershell
Get-Process
~~~

A process can have:

- Process ID (PID)
- Parent process
- Executable path
- User/security context
- Memory usage
- Threads
- Network activity

### SOC relevance

Process analysis is central to endpoint detection and response.

An analyst may investigate:

- What executable is running?
- Where is it located?
- Which user launched it?
- What is its parent?
- What command line was used?
- What child processes did it create?
- What network connections did it make?

---

## 16. Parent and Child Processes

Processes commonly form parent-child relationships.

For example:

~~~text
explorer.exe
     ↓
powershell.exe
     ↓
cmd.exe
     ↓
whoami.exe
~~~

The process tree provides context about execution.

A command such as whoami.exe can be completely legitimate. However, the surrounding context matters.

For example:

~~~text
winword.exe
    ↓
powershell.exe
    ↓
cmd.exe
    ↓
suspicious.exe
~~~

could deserve investigation depending on the environment and observed behavior.

The process tree should never be treated as proof of malicious activity by itself. It is an important contextual signal.

---

## 17. Windows Services

Windows services are background components designed to perform functions without requiring a user to interact with them directly.

Useful commands:

~~~cmd
sc query
~~~

PowerShell:

~~~powershell
Get-Service
~~~

A specific service can be queried with:

~~~cmd
sc query <service_name>
~~~

### SOC relevance

Attackers may create or modify services to establish persistence or execute code with elevated privileges.

Analysts should investigate:

- Newly created services
- Unusual service names
- Unexpected service executable paths
- Services running under unusual accounts
- Recently modified service configurations

---

## 18. Windows Task Scheduler

Task Scheduler allows programs or scripts to execute automatically according to triggers.

Tasks may run:

- At startup
- At logon
- At a scheduled time
- When an event occurs
- According to other configured triggers

Command-line inspection:

~~~cmd
schtasks /query /fo LIST /v
~~~

### SOC relevance

Scheduled tasks are commonly used for legitimate automation, but they can also be abused for persistence.

An analyst should inspect:

- Task name
- Author
- Trigger
- Action
- Executable path
- Run-as account
- Creation or modification information

---

## 19. Windows Command Prompt

Command Prompt, or CMD, is the traditional Windows command-line interface.

Useful commands include:

~~~cmd
dir
cd
type
copy
move
del
whoami
hostname
ipconfig
tasklist
netstat
~~~

The command line is important for both system administration and security investigations.

### SOC relevance

Attackers often use command-line tools because they are already present on Windows systems. This is commonly discussed as **living off the land**, where legitimate system utilities are abused for malicious purposes.

The use of a built-in utility is not automatically malicious. Analysts must examine context, parent process, command line, user, destination, and timing.

---

## 20. PowerShell

PowerShell is a powerful command-line shell and scripting environment built into modern Windows.

Examples:

~~~powershell
Get-Process
Get-Service
Get-ChildItem C:\Windows
Get-NetTCPConnection
~~~

PowerShell can interact with:

- Files
- Processes
- Services
- Registry
- Networking
- Windows APIs
- Remote systems

### SOC relevance

PowerShell is heavily used by administrators and developers, but it is also frequently abused during attacks.

SOC analysts commonly investigate:

- PowerShell process creation
- Command-line arguments
- Script execution
- Encoded commands
- Download activity
- Child processes
- Network connections
- PowerShell logging

Detailed PowerShell detection will be covered later in the challenge.

---

## 21. Windows Networking

Important commands include:

~~~cmd
ipconfig /all
route print
arp -a
netstat -ano
~~~

PowerShell provides additional commands:

~~~powershell
Get-NetIPConfiguration
Get-NetTCPConnection
Get-NetRoute
~~~

### SOC relevance

Network investigation often requires correlating:

~~~text
IP Address
    ↓
Port
    ↓
PID
    ↓
Process
    ↓
User
~~~

For example, netstat -ano can show a connection and its PID. The PID can then be correlated with the process using Task Manager or:

~~~cmd
tasklist /fi "PID eq <PID>"
~~~

This is a practical endpoint-investigation technique.

---

## 22. Windows Firewall

Windows includes a host-based firewall that controls network traffic according to configured rules.

PowerShell can inspect firewall configuration:

~~~powershell
Get-NetFirewallProfile
Get-NetFirewallRule
~~~

### SOC relevance

Firewall configuration can help analysts determine whether:

- A service is exposed
- A rule was recently created
- An unexpected port was allowed
- Security controls were disabled
- An application received network access

Firewall state and rules should be interpreted alongside network telemetry.

---

## 23. Windows Hosts File and DNS

The Windows hosts file can locally map hostnames to IP addresses.

Common location:

~~~text
C:\Windows\System32\drivers\etc\hosts
~~~

Example:

~~~text
192.168.1.20 internal-server.local
~~~

Useful DNS commands include:

~~~cmd
nslookup example.com
ipconfig /displaydns
ipconfig /flushdns
~~~

### SOC relevance

Attackers may modify the hosts file to redirect traffic or interfere with name resolution. DNS activity can also reveal suspicious domains, command-and-control infrastructure, or unexpected external services.

---

## 24. Windows Event Logging

Windows generates events for many types of activity, including:

- User logon
- Process creation
- Service activity
- System changes
- Security-policy changes
- Application errors

Event Viewer can be launched with:

~~~text
eventvwr.msc
~~~

PowerShell can query events:

~~~powershell
Get-WinEvent -LogName System -MaxEvents 20
~~~

### SOC relevance

Windows Event Logs are a major source of security telemetry.

The next challenge day will focus specifically on Windows Event Logs, including important Security events, Event IDs, logon activity, and investigation workflows.

---

## 25. Windows Event Log Files

Windows Event Logs are commonly stored under:

~~~text
C:\Windows\System32\winevt\Logs\
~~~

Common logs include:

- Security
- System
- Application
- Microsoft-Windows-PowerShell/Operational
- Microsoft-Windows-Sysmon/Operational, when Sysmon is installed

The exact logs available depend on system configuration.

---

## 26. Windows Authentication

Windows authentication can involve different mechanisms depending on the environment.

Common concepts include:

- Local authentication
- Domain authentication
- NTLM
- Kerberos

In a standalone system, a local account may authenticate against the local security authority.

In an Active Directory environment, domain authentication can involve domain controllers and Kerberos.

### SOC relevance

Authentication telemetry is critical for detecting:

- Brute-force attempts
- Password spraying
- Unusual logons
- Lateral movement
- Privilege abuse
- Compromised accounts

Active Directory authentication will be covered in greater depth later.

---

## 27. Windows Remote Access

Common Windows remote-access technologies include:

- RDP
- SMB
- WinRM
- Remote PowerShell

These technologies have legitimate administrative uses but can also be abused by attackers.

An unexpected remote login followed by PowerShell execution and access to multiple internal systems could warrant investigation.

The analyst should correlate identity, source IP, destination host, process execution, and authentication events.

---

## 28. Windows Security Controls

Windows includes multiple security mechanisms, depending on edition and configuration.

Examples include:

- Microsoft Defender Antivirus
- Windows Firewall
- User Account Control (UAC)
- Windows Security
- SmartScreen
- Application-control technologies
- Credential protections

### SOC relevance

Security controls generate useful telemetry and can also become attack targets.

For example, disabling security software or modifying firewall rules may be a meaningful event during an investigation.

---

## 29. User Account Control — UAC

UAC helps limit the ability of applications to perform elevated operations without user approval or an appropriate elevation mechanism.

A standard desktop process and an elevated administrator process can operate under different security contexts.

### SOC relevance

Attackers may attempt to bypass or abuse elevation mechanisms to obtain higher privileges.

When investigating privilege escalation, analysts should correlate:

- User identity
- Process creation
- Integrity level
- Parent process
- Requested privileges
- Resulting administrative activity

---

## 30. Windows Startup and Persistence

Applications can automatically execute when Windows starts or when a user logs in.

Potential persistence mechanisms include:

- Startup folders
- Registry Run keys
- Services
- Scheduled tasks
- Group Policy
- Other Windows mechanisms

Example registry locations:

~~~text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
~~~

### SOC relevance

Unexpected entries in startup locations can indicate persistence.

Analysts should investigate:

- What executable is referenced?
- Who created the entry?
- When was it created?
- Where is the executable located?
- Is it signed?
- What process launched it?

---

## 31. Windows Temporary Directories

Common temporary locations include:

~~~text
C:\Windows\Temp
%TEMP%
%TMP%
~~~

User-specific temporary data may also exist under the user's profile.

### SOC relevance

Malware, scripts, installers, archives, and temporary payloads may appear in these locations.

However, temporary directories contain large amounts of legitimate activity. Analysts should use file type, timestamps, process lineage, hash, network activity, and execution evidence to determine whether an artifact deserves investigation.

---

## 32. Windows File Metadata and Hashing

PowerShell can inspect file metadata:

~~~powershell
Get-Item C:\path\file.exe | Format-List *
~~~

A SHA-256 hash can be calculated with:

~~~powershell
Get-FileHash C:\path\file.exe -Algorithm SHA256
~~~

Analysts may examine:

- File size
- Creation time
- Modification time
- Last access time
- Owner
- Attributes
- File version
- Digital signature
- Hash

### SOC relevance

File hashes are useful for identifying known artifacts and correlating files across endpoints, but a hash alone does not establish maliciousness.

---

## 33. Windows Executables and DLLs

Windows commonly uses executable formats such as:

- EXE
- DLL
- SYS
- MSI

A DLL is a Dynamic Link Library containing reusable code and resources. Executables can load DLLs to provide functionality.

### Security relevance

Malware can abuse legitimate executable and DLL mechanisms. Analysts may investigate:

- Unusual executable paths
- Unsigned binaries
- Recently created DLLs
- DLL loading behavior
- Suspicious parent-child relationships
- Execution from user-writable directories

---

## 34. Services, Drivers, and Privileged Components

Windows services and drivers can operate with significant privileges.

Drivers run in kernel mode and interact closely with hardware and the operating system.

### Security relevance

Malicious or vulnerable drivers can provide attackers with powerful capabilities. Driver-related investigation is therefore important in advanced endpoint security.

However, legitimate security software also installs drivers. Analysts must verify publisher, signature, installation time, path, associated software, and behavior.

---

## 35. Example Windows Investigation Scenario

Suppose a SOC alert indicates that a Windows workstation connected to a suspicious external IP.

### Step 1 — Identify the host

~~~cmd
hostname
~~~

### Step 2 — Identify the user and privileges

~~~cmd
whoami
whoami /groups
whoami /priv
~~~

### Step 3 — Identify network connections

~~~cmd
netstat -ano
~~~

Find the suspicious destination and associated PID.

### Step 4 — Identify the process

~~~cmd
tasklist /fi "PID eq <PID>"
~~~

Determine the executable associated with the connection.

### Step 5 — Investigate process lineage

Determine the parent process, child processes, command line, user context, and execution path.

### Step 6 — Investigate the file

Check:

- File path
- Hash
- Signature
- Timestamps
- Reputation
- Whether the file was recently created

### Step 7 — Investigate persistence

Check:

- Services
- Scheduled tasks
- Startup locations
- Registry Run keys

### Step 8 — Review Windows Event Logs

Correlate endpoint activity with authentication and process-related events.

### Step 9 — Establish a timeline

~~~text
User Logon
    ↓
Suspicious File Creation
    ↓
Process Execution
    ↓
Network Connection
    ↓
Persistence
~~~

### Step 10 — Determine scope

Search whether the same user, hash, destination IP, domain, or process appears on other endpoints.

This is the foundation of Windows endpoint investigation.

---

## 36. Windows Investigation Evidence

A SOC analyst may correlate evidence from:

| Evidence | What it can tell you |
|---|---|
| User account | Identity involved |
| SID | Security principal identity |
| Process | What program executed |
| PID | Specific process instance |
| Parent process | How execution started |
| Command line | How the program was invoked |
| Network connection | Where the process communicated |
| File hash | File-content identifier |
| File metadata | Artifact timeline/context |
| Service | Background execution/persistence |
| Scheduled task | Automated execution |
| Registry | Configuration/persistence |
| Event Logs | System/security activity |
| Defender telemetry | Security detections |
| Firewall telemetry | Network control activity |

No single artifact should normally be treated as the complete story. Strong investigations correlate multiple evidence sources.

---

## 🧪 Practical Labs

### Lab 1 — Windows System Reconnaissance

Run:

~~~cmd
hostname
whoami
whoami /user
whoami /groups
whoami /priv
systeminfo
~~~

Document the host, current identity, groups, privileges, operating-system information, and security context.

### Lab 2 — Windows Filesystem Investigation

Explore:

~~~text
C:\Windows
C:\Windows\System32
C:\Users
C:\ProgramData
C:\Windows\Temp
~~~

Identify the purpose of each directory and locate several legitimate Windows executables.

### Lab 3 — Process Investigation

Run:

~~~cmd
tasklist
tasklist /v
~~~

Then inspect processes with PowerShell:

~~~powershell
Get-Process
~~~

Identify process names, PIDs, resource usage, and process relationships.

### Lab 4 — Network-to-Process Correlation

Run:

~~~cmd
netstat -ano
~~~

Identify a TCP connection and note its PID.

Then:

~~~cmd
tasklist /fi "PID eq <PID>"
~~~

Understand how a network connection can be associated with a process.

### Lab 5 — Service Investigation

Run:

~~~cmd
sc query
~~~

or:

~~~powershell
Get-Service
~~~

Select several services and determine their name, status, startup configuration, purpose, and associated executable where available.

### Lab 6 — Scheduled Task Investigation

Run:

~~~cmd
schtasks /query /fo LIST /v
~~~

Identify scheduled tasks and examine their triggers and actions.

### Lab 7 — Registry Exploration

Open:

~~~text
regedit
~~~

In an authorized lab, inspect the structure of HKLM, HKCU, HKU, HKCR, and HKCC.

Do not modify registry settings unless the lab specifically requires it.

### Lab 8 — File Hash and Metadata Investigation

Use:

~~~powershell
Get-FileHash C:\Windows\System32\notepad.exe -Algorithm SHA256
Get-Item C:\Windows\System32\notepad.exe | Format-List *
~~~

Document the hash and available metadata.

### Lab 9 — Windows Event Viewer Introduction

Open:

~~~text
eventvwr.msc
~~~

Explore:

- Windows Logs
- Application
- Security
- System

The purpose of this lab is familiarization. Detailed event analysis will be covered on Day 5.

---

## 🔍 SOC Analyst Perspective

Windows fundamentals become valuable when an alert needs to be transformed into an investigation.

Suppose the SIEM reports:

~~~text
Windows Endpoint
       ↓
Outbound connection
       ↓
External IP:443
       ↓
PID 4820
~~~

A SOC analyst should be able to work backward:

~~~text
PID 4820
   ↓
Process
   ↓
Executable
   ↓
Command line
   ↓
Parent process
   ↓
User
   ↓
Logon event
   ↓
File activity
   ↓
Persistence
~~~

The analyst can then work forward again:

~~~text
Initial access
     ↓
User activity
     ↓
Process execution
     ↓
Network communication
     ↓
Persistence
     ↓
Potential lateral movement
~~~

This bidirectional reasoning is important because security investigations are rarely solved by looking at one log entry.

The core principle is **correlation**.

A suspicious process becomes more meaningful when combined with:

- An unexpected user
- An unusual parent process
- A recently created executable
- A suspicious network connection
- A new scheduled task
- A new service
- A relevant authentication event

This is how endpoint telemetry becomes an incident narrative.

---

## 📌 Key Concepts Learned

By completing Day 4, the following Windows fundamentals were covered:

- Windows architecture
- User mode and kernel mode
- Windows kernel
- Local and domain accounts
- Security Identifiers (SIDs)
- Windows groups
- Privileges
- Access tokens
- Integrity levels
- NTFS
- Windows filesystem hierarchy
- Important Windows directories
- Windows Registry
- Registry hives
- Windows processes
- PIDs and process relationships
- Windows services
- Task Scheduler
- Command Prompt
- PowerShell fundamentals
- Windows networking
- Network-to-process correlation
- Windows Firewall
- Hosts file
- DNS
- Windows Event Logging
- Windows authentication
- Remote access technologies
- Environment variables
- Temporary directories
- File metadata and hashing
- EXE/DLL concepts
- Drivers and privileged components
- Windows security controls
- UAC
- Startup and persistence mechanisms
- Windows endpoint investigation methodology

---

## 📌 Day 4 Outcome

Day 4 established the Windows operating-system foundation required for SOC monitoring and endpoint investigation.

The focus was on understanding how Windows manages **identities, privileges, processes, services, files, registry configuration, networking, authentication, scheduled execution, and security controls**.

These concepts will directly support the next stage of the challenge, where we move deeper into **Windows Event Logs and security-relevant Event IDs**.

**Status: ✅ Completed**
