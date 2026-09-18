# Day 3 — Linux Fundamentals

## 🎯 Objective

The objective of Day 3 is to build a practical Linux foundation from a **SOC analyst and security investigation perspective**. Linux systems are widely used for servers, cloud workloads, security appliances, containers, network services, and security tools. A SOC analyst therefore needs to understand how Linux is structured, how users and processes operate, where security-relevant logs are stored, how network services are identified, and how command-line tools can be used to investigate suspicious activity.

This day focuses on understanding **what Linux components do, how they work together, what evidence they produce, and why that evidence matters during a security investigation**.

---

## 1. What Is Linux?

Linux is an open-source operating-system kernel. In practical usage, the term Linux commonly refers to a complete operating-system distribution built around the Linux kernel and user-space software.

Common distributions include:

- Ubuntu
- Debian
- Fedora
- Red Hat Enterprise Linux
- Rocky Linux
- AlmaLinux
- Kali Linux
- Arch Linux

A Linux system normally contains the kernel, system libraries, command-line utilities, services, configuration files, package-management tools, and applications.

### SOC relevance

Linux systems frequently host web servers, databases, authentication services, cloud workloads, containers, SIEM components, and security tools. During an incident, a SOC analyst may need to determine:

- Which users are logged in?
- Which processes are running?
- Which services are listening?
- Which network connections are active?
- What commands were executed?
- What authentication events occurred?
- Which files changed?
- What happened immediately before and after an alert?

Linux fundamentals provide the foundation for answering these questions.

---

## 2. Linux Architecture

A simplified Linux architecture is:

`text
Applications
     ↓
Shell / System Utilities
     ↓
System Libraries
     ↓
Linux Kernel
     ↓
Hardware
`

The **kernel** is the core component responsible for process scheduling, memory management, device management, filesystems, networking, and access to hardware resources.

Applications normally do not communicate directly with hardware. They request operating-system services through system calls, which are handled by the kernel.

### Security relevance

A malicious process may attempt to execute commands, access files, create network connections, spawn additional processes, modify configuration, establish persistence, or obtain higher privileges. Understanding Linux architecture helps an analyst understand where such activity occurs and what telemetry may be available.

---

## 3. The Linux Shell

The shell is a command interpreter that allows users to interact with Linux.

Common shells include:

- Bash
- Zsh
- Fish
- Dash

Bash is especially common on Linux servers.

Example:

`bash
echo "Hello SOC"
pwd
whoami
`

The shell interprets the command and requests the required operation from the operating system.

### SOC relevance

Attackers frequently use shells after obtaining access to a Linux host. Shell activity, command history, process trees, authentication records, and terminal-related telemetry can therefore become important investigation evidence.

---

## 4. Linux Filesystem Hierarchy

Linux uses a single directory tree beginning at the root directory:

`text
/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
`

Unlike Windows, Linux does not normally use drive letters such as C:\ to represent the primary filesystem hierarchy.

### Important directories

**/** — Root of the filesystem.

**/home** — Home directories for normal users.

**/root** — Home directory of the root account.

**/etc** — System-wide configuration files. Important examples include /etc/passwd, /etc/group, /etc/ssh/, /etc/systemd/, and /etc/hosts.

**/var** — Variable data such as logs, caches, queues, and application data. /var/log is especially important during investigations.

**/tmp** — Temporary files. Suspicious scripts, archives, or binaries may sometimes appear here.

**/usr** — Many user-space programs, libraries, and shared resources.

**/proc** — Virtual filesystem exposing information about running processes and kernel/system state.

**/dev** — Device files.

**/sys** — Interface to kernel and device information.

Attackers may modify configuration files, create executable files, or establish persistence in several of these locations, so understanding their purpose is essential.

---

## 5. Navigating the Filesystem

The most basic navigation commands are:

`bash
pwd
ls
cd
`

`pwd` displays the current directory.

`ls -l` displays detailed file information such as permissions, ownership, size, and timestamps.

`ls -la` also displays hidden files.

An absolute path starts from the root:

`text
/etc/ssh/sshd_config
`

A relative path is interpreted from the current directory:

`text
./logs/auth.log
`

A SOC analyst should be comfortable navigating the filesystem because investigations often require locating logs, configuration files, scripts, binaries, and suspicious artifacts.

---

## 6. Essential File Operations

Create a file:

`bash
touch test.txt
`

Create a directory:

`bash
mkdir investigation
`

Copy:

`bash
cp source.txt destination.txt
`

Move or rename:

`bash
mv old.txt new.txt
`

Remove:

`bash
rm file.txt
`

Remove a directory recursively:

`bash
rm -r directory
`

Destructive commands must be used carefully during investigations because deleting files can destroy evidence.

---

## 7. Reading Files

Common commands include:

`bash
cat file.txt
less file.txt
head file.txt
tail file.txt
`

For a log that is actively being written:

`bash
tail -f /var/log/auth.log
`

This follows new lines as they are appended.

### SOC relevance

Reading logs in real time is useful when reproducing activity in a lab or validating whether an authentication attempt, service event, or other action generates the expected telemetry.

---

## 8. Linux File Permissions

Linux uses permissions to control access to files and directories.

Example:

`text
-rwxr-xr--
`

The permission portion can be understood as:

`text
-   rwx   r-x   r--
    │     │     │
    │     │     └── Others
    │     └──────── Group
    └────────────── Owner
`

The three basic permissions are:

- **r** — read
- **w** — write
- **x** — execute

Permissions can also be represented numerically:

- Read = 4
- Write = 2
- Execute = 1

Therefore:

`text
755
`

means:

- Owner = 7 = read + write + execute
- Group = 5 = read + execute
- Others = 5 = read + execute

---

## 9. chmod and chown

`chmod` changes permissions.

`bash
chmod 755 script.sh
chmod +x script.sh
`

`chown` changes ownership.

`bash
sudo chown root:root suspicious.sh
`

### SOC relevance

Unexpected executable permissions or ownership changes can be useful investigation signals. An analyst may ask whether a suspicious script was recently made executable, whether it is owned by a privileged account, and whether the change occurred before suspicious execution.

---

## 10. Linux Users

Linux supports multiple user accounts.

Useful commands:

`bash
whoami
id
who
w
groups
`

`whoami` shows the current effective username.

`id` shows the UID, primary group, and supplementary groups.

`who` shows logged-in users.

`w` provides information about logged-in users and their current activity.

### SOC relevance

A security investigation needs to establish **which identity performed an action**. User information is therefore a major part of endpoint timeline reconstruction.

---

## 11. /etc/passwd and /etc/shadow

The file `/etc/passwd` contains account information.

A simplified entry looks like:

`text
dheeraj:x:1000:1000:Dheeraj:/home/dheeraj:/bin/bash
`

The fields include username, password-placeholder field, UID, GID, comment information, home directory, and login shell.

Password hashes are normally stored in `/etc/shadow` on systems using local password authentication.

### SOC relevance

Unexpected accounts, unusual UIDs, modified login shells, or newly created privileged users can be important indicators during an investigation. Access to password hashes must also be treated as sensitive authentication material.

---

## 12. Groups and Privilege

Groups allow users to share permissions.

Useful commands:

`bash
groups
getent group
`

Examples of groups that may have significant privileges include:

- sudo
- adm
- docker
- www-data

The root account has extensive system privileges. Authorized users may use `sudo` to execute commands with elevated privileges.

Example:

`bash
sudo systemctl status ssh
`

During an investigation, analysts should determine:

1. Which account was initially involved?
2. Did it execute privileged commands?
3. Was sudo used?
4. Was a new privileged account created?
5. Were permissions changed?
6. Was persistence established after privilege escalation?

---

## 13. Processes

A process is a running instance of a program.

Useful commands include:

`bash
ps
ps aux
top
htop
`

A process has information such as:

- PID
- PPID
- User
- CPU usage
- Memory usage
- Command line
- Process state

### Parent-child relationships

For example:

`text
sshd
  └── bash
       └── python3
            └── curl
`

The process tree can help explain how an executable was launched.

A SOC analyst should correlate the process tree with user identity, timestamps, command lines, network connections, and other telemetry before deciding whether the behavior is suspicious.

---

## 14. Process IDs and Signals

Every running process has a PID.

Use:

`bash
ps -ef
`

to inspect processes and their relationships.

Linux processes can receive signals such as:

- SIGTERM
- SIGKILL
- SIGHUP
- SIGINT

Example:

`bash
kill <PID>
`

Forceful termination:

`bash
kill -9 <PID>
`

During a live incident, terminating a process can destroy volatile evidence. Analysts should therefore follow the incident-response procedure before taking disruptive actions.

---

## 15. systemd and Services

Modern Linux distributions commonly use **systemd** as the system and service manager.

Useful commands:

`bash
systemctl status ssh
systemctl start ssh
systemctl stop ssh
systemctl restart ssh
`

List services:

`bash
systemctl list-units --type=service
`

List enabled services:

`bash
systemctl list-unit-files --type=service
`

### SOC relevance

Attackers may establish persistence through malicious or modified services. Analysts should investigate newly created services, unexpected enabled services, modified service definitions, suspicious service executables, and unusual service accounts.

---

## 16. Linux Networking

Important commands include:

`bash
ip addr
ip route
ip neigh
ss
`

`ip addr` shows interfaces and IP addresses.

`ip route` shows the routing table.

`ip neigh` shows neighbor information.

`ss` displays sockets and network connections.

Example:

`bash
ss -tulnp
`

This can show listening TCP/UDP sockets and associated processes when permissions allow.

### SOC relevance

Unexpected listening services and unusual outbound connections can become important investigation leads.

---

## 17. Network Sockets and Listening Ports

A service listening on a port is not automatically malicious.

For example:

`text
0.0.0.0:22
`

may indicate that SSH is listening on all IPv4 interfaces.

The analyst should determine:

- Which process owns the socket?
- Which user runs it?
- Is the service expected?
- When did it start?
- Is the port externally reachable?
- Are there authentication attempts against it?

This prevents a single technical indicator from being incorrectly treated as proof of compromise.

---

## 18. SSH

SSH provides secure remote access to Linux systems.

Example:

`bash
ssh user@192.168.1.10
`

SSH is widely used for legitimate administration but is also frequently targeted by attackers.

SOC analysts should understand:

- Successful logins
- Failed logins
- Source IP addresses
- User accounts
- SSH configuration
- SSH keys
- Brute-force patterns
- Unusual remote sessions

SSH evidence is particularly useful when constructing an authentication timeline.

---

## 19. Linux Logs

Logs are among the most important sources of Linux security evidence.

Common locations include:

`text
/var/log/
`

Depending on distribution and configuration, logs may include:

- Authentication events
- System events
- Kernel events
- Application events
- Web-server events
- Package-management events

Modern Linux systems may also use **systemd-journald**, queried with `journalctl`.

The exact filenames and logging architecture vary by distribution.

---

## 20. journalctl

`journalctl` queries the systemd journal.

Examples:

`bash
journalctl
journalctl -b
journalctl -p warning
journalctl -u ssh
journalctl --since "1 hour ago"
`

These queries can help investigate system events, service activity, authentication behavior, and other events.

### SOC relevance

A timeline can be built by correlating journal entries with authentication, service starts/stops, process execution, network activity, and configuration changes.

---

## 21. Authentication Logs

Depending on distribution and configuration, authentication events may be available through:

`text
/var/log/auth.log
`

or:

`text
/var/log/secure
`

Common events of interest include:

- Failed logins
- Successful logins
- sudo activity
- SSH activity
- Account changes

Example:

`bash
grep "Failed password" /var/log/auth.log
`

The exact source should always be verified on the target system.

---

## 22. grep and Regular Expressions

`grep` searches text for patterns.

Examples:

`bash
grep "error" application.log
grep -i "failed" application.log
grep -R "suspicious" /var/log/
`

Regular expressions allow analysts to search structured patterns.

Example:

`bash
grep -E '([0-9]{1,3}\.){3}[0-9]{1,3}' file.log
`

This can locate strings resembling IPv4 addresses.

### SOC relevance

Large log files may contain thousands or millions of entries. Search tools allow analysts to rapidly isolate events relevant to an investigation.

A pattern match is not automatically malicious; it must be interpreted in context.

---

## 23. find

`find` searches files and directories using attributes such as name, type, size, owner, and timestamps.

Examples:

`bash
find /tmp -type f
find /tmp -type f -name "*.sh"
find /var/tmp -type f -mtime -1
`

### SOC relevance

Attackers may create scripts, binaries, archives, or configuration files during an intrusion. Searching temporary locations and recently modified files can help locate potential artifacts.

---

## 24. File Metadata and Hashes

Useful commands include:

`bash
stat suspicious.bin
file suspicious.bin
sha256sum suspicious.bin
ls -l suspicious.bin
`

These provide information about:

- File type
- Size
- Ownership
- Permissions
- Timestamps
- Cryptographic hash

Hashes can be used to compare a file with known samples or threat-intelligence data.

### Important limitation

A hash identifies a specific file content. It does not by itself prove that the file is malicious. The analyst should correlate the hash with execution context, source, behavior, and other evidence.

---

## 25. Pipes and Redirection

A pipe sends the output of one command into another.

`bash
ps aux | grep ssh
`

The output of `ps aux` becomes input to `grep`.

Output can be redirected to a file:

`bash
ps aux > processes.txt
ps aux >> processes.txt
`

Standard error can also be redirected:

`bash
command 2> errors.txt
`

These features allow analysts to build repeatable command-line investigation workflows.

---

## 26. awk and sed

`awk` is useful for field-based text processing.

`bash
awk '{print $1}' access.log
`

`sed` can filter or transform text.

`bash
sed -n '1,20p' file.txt
`

These tools are particularly useful when extracting fields from large log files.

---

## 27. Command History

Bash may store command history in:

`text
~/.bash_history
`

The history command displays recent commands:

`bash
history
`

### SOC relevance

Command history can provide useful investigative context, but it is **not a complete record of all commands executed**. History may be disabled, cleared, configured differently, or bypassed.

Therefore, command history should be correlated with process telemetry, authentication logs, audit logs, and other evidence.

---

## 28. Cron and Scheduled Tasks

Cron provides scheduled task execution.

Common locations include:

`text
/etc/crontab
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
`

User crontabs can be inspected with:

`bash
crontab -l
`

### SOC relevance

Scheduled tasks can be used legitimately for maintenance and automation, but attackers can also abuse them for persistence.

Investigate unexpected jobs, scripts, commands, execution paths, ownership, and creation or modification times.

---

## 29. Package Management

Common package managers include:

### Debian/Ubuntu

`bash
apt update
apt install <package>
apt list --installed
`

### Red Hat-based systems

`bash
dnf install <package>
dnf list installed
`

Package-management logs can help establish whether software was recently installed or updated.

### SOC relevance

An unexpected package installation can become an important timeline event when correlated with suspicious activity.

---

## 30. Environment Variables

Environment variables provide configuration information to processes and shells.

View them with:

`bash
env
echo $PATH
`

The `PATH` variable determines where the shell searches for executable commands.

### Security relevance

Unexpected PATH modifications can cause a different executable to run when a user invokes a command. This can be relevant to command hijacking and persistence investigations.

---

## 31. Persistence Mechanisms

Linux persistence can be implemented in several ways, including:

- systemd services
- cron jobs
- SSH authorized keys
- shell startup files
- modified configuration
- scheduled tasks
- user accounts
- malicious binaries or scripts

A SOC analyst should investigate persistence after suspicious execution because persistence explains how an attacker may regain access after the initial intrusion.

---

## 32. Linux Security Investigation Workflow

A basic endpoint investigation can follow:

`text
Alert
  ↓
Identify affected host
  ↓
Identify user
  ↓
Review authentication activity
  ↓
Inspect processes
  ↓
Inspect network connections
  ↓
Review logs
  ↓
Search suspicious files
  ↓
Check persistence mechanisms
  ↓
Build timeline
  ↓
Determine scope
  ↓
Document findings
`

The exact workflow depends on the incident and available telemetry.

---

## 33. Example Investigation Scenario

Suppose a Linux server generates an alert for an unusual outbound connection.

### Step 1 — Identify network activity

`bash
ss -tunap
`

Determine which process is associated with the connection.

### Step 2 — Identify the process

`bash
ps -ef
`

Check the PID, user, parent process, and command line.

### Step 3 — Identify the account

`bash
id <username>
`

Review the user's privileges and group memberships.

### Step 4 — Review authentication

Search the appropriate authentication logs or system journal.

### Step 5 — Search for recently modified artifacts

`bash
find /tmp /var/tmp -type f -mtime -1
`

### Step 6 — Check persistence

Review systemd services, cron jobs, shell startup files, SSH keys, and other relevant mechanisms.

### Step 7 — Build a timeline

Correlate:

`text
Login
  ↓
Command execution
  ↓
File creation
  ↓
Process start
  ↓
Network connection
  ↓
Persistence
`

This approach converts isolated host artifacts into an investigation narrative.

---

## 34. Practical SOC Command Set

### Navigation and files

`bash
pwd
ls -la
cd
cp
mv
rm
mkdir
touch
`

### Users and permissions

`bash
whoami
id
who
w
groups
chmod
chown
`

### Processes and services

`bash
ps
top
htop
systemctl
journalctl
`

### Networking

`bash
ip addr
ip route
ip neigh
ss
ping
dig
nslookup
curl
`

### Investigation and text processing

`bash
cat
less
head
tail
grep
grep -E
find
awk
sed
stat
file
sha256sum
`

### Persistence and history

`bash
history
crontab -l
`

---

## 🧪 Practical Labs

### Lab 1 — Linux System Reconnaissance

Collect basic host information:

`bash
hostname
uname -a
whoami
id
uptime
`

Document the operating-system and user context.

### Lab 2 — Filesystem Investigation

Explore:

`text
/etc
/var/log
/tmp
/var/tmp
/home
`

Identify configuration, logging, user-data, and temporary-artifact locations.

### Lab 3 — User and Permission Analysis

In an authorized lab, create a test user and inspect:

`bash
id <user>
groups <user>
ls -l
`

Experiment with read, write, and execute permissions.

### Lab 4 — Process Investigation

Run:

`bash
ps aux
ss -tulnp
`

Identify running processes, listening services, associated users, and available command-line information.

### Lab 5 — Log Investigation

Inspect available authentication and system logs.

For systemd-based systems:

`bash
journalctl --since "1 hour ago"
`

Search for failed authentication events where applicable.

### Lab 6 — File Investigation

Create and investigate a test artifact:

`bash
touch /tmp/soc-test.txt
find /tmp -name "soc-test.txt"
stat /tmp/soc-test.txt
sha256sum /tmp/soc-test.txt
`

Understand how file metadata supports an investigation.

### Lab 7 — Process-to-Network Correlation

Identify a listening service:

`bash
ss -tulnp
`

Then determine the associated process and user.

The objective is to understand:

`text
Network Socket
      ↓
Port
      ↓
Process
      ↓
User
      ↓
Executable
`

This correlation is highly valuable during endpoint investigations.

---

## 🔍 SOC Analyst Perspective

Linux knowledge becomes valuable when raw endpoint activity must be converted into an investigation narrative.

For example, an alert may indicate:

`text
Linux Host → Suspicious External IP:443
`

A SOC analyst should not stop at the IP address. The investigation should ask:

- Which process created the connection?
- Which user owned that process?
- What was the parent process?
- When did the process start?
- Did the user recently authenticate?
- Was a suspicious file created?
- Was a scheduled task or service modified?
- Did the system show other connections to the same infrastructure?
- Are other hosts showing similar behavior?

The objective is to establish a defensible chain of events rather than collecting isolated indicators.

---

## 📌 Key Concepts Learned

By completing Day 3, the following Linux fundamentals were covered:

- Linux architecture
- Kernel and user space
- Shells and Bash
- Linux filesystem hierarchy
- Important Linux directories
- File and directory operations
- File permissions
- chmod and chown
- Users and groups
- /etc/passwd and /etc/shadow
- Root and sudo
- Processes, PIDs, and PPIDs
- Parent-child process relationships
- systemd and services
- Linux networking
- Sockets and listening ports
- SSH
- Linux logging
- systemd journal
- Authentication logs
- grep and regular expressions
- find
- awk and sed
- Pipes and redirection
- Command history
- Cron
- Package management
- Environment variables
- File metadata
- Hashing
- Persistence mechanisms
- Linux endpoint investigation methodology

---

## 📌 Day 3 Outcome

Day 3 established the Linux foundation required for SOC endpoint monitoring and investigation. The focus was on understanding how Linux manages users, files, permissions, processes, services, networking, logs, and persistence mechanisms.

These concepts will become directly relevant in later days when investigating Linux authentication activity, suspicious processes, network connections, persistence, malware behavior, and security alerts.

**Status: ✅ Completed**
