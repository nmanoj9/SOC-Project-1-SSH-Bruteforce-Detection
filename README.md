# SOC Project 1 – SSH Brute Force Detection Using Splunk

## Project Overview

This project demonstrates the detection and investigation of an SSH brute-force attack using Splunk SIEM. The objective was to simulate unauthorized access attempts in a controlled lab environment, collect authentication logs, ingest them into Splunk, and create correlation-based detections to identify malicious activity.

---

## Objectives

* Simulate an SSH brute-force attack in a controlled lab environment
* Monitor authentication logs for failed and successful login attempts
* Ingest Linux authentication logs into Splunk
* Develop correlation-based detection logic
* Create Splunk alerts for brute-force detection
* Visualize attack activity using Splunk dashboards
* Investigate attacker behavior and persistence activity

---
## Repository Structure

This repository includes:

- README.md – Project overview
- Documentation/ – Detailed investigation notes
- Queries/ – Splunk SPL searches used during the investigation
- Screenshots/ – Investigation evidence and dashboard images

---

## Lab Environment

| Component               | Description       |
| ----------------------- | ----------------- |
| Attacker Machine        | Kali Linux        |
| Target Machine          | Metasploitable 2  |
| SIEM Platform           | Splunk Enterprise |
| Virtualization Platform | Oracle VirtualBox |
| Network Configuration   | NAT Network       |

---

## Tools Used

* Kali Linux
* Metasploitable 2
* Splunk Enterprise
* Metasploit Framework
* Oracle VirtualBox

---

# Lab Architecture

```
                 +----------------------+
                 |    Kali Linux        |
                 | (Attacker Machine)   |
                 +----------+-----------+
                            |
                     SSH Brute Force
                            |
                            v
                 +----------------------+
                 |   Metasploitable 2   |
                 |    (SSH Server)      |
                 +----------+-----------+
                            |
                     Authentication Logs
                     (/var/log/auth.log)
                            |
                            v
                 +----------------------+
                 |  Splunk Enterprise   |
                 |  Search & Reporting  |
                 | Detection & Analysis |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |   SOC Investigation  |
                 | Detection Rule       |
                 | Dashboard            |
                 | MITRE ATT&CK         |
                 +----------------------+
```
---

### Architecture Overview

The lab consisted of a Kali Linux attacker machine and a Metasploitable 2 target system running an SSH service. Authentication logs generated during the simulated brute-force attack were collected from the target and analyzed using Splunk Enterprise. Correlation searches, dashboards, and detection logic were used to identify repeated authentication failures and successful logins from the same source IP address.

---

---

# Investigation Workflow

The investigation followed a structured SOC workflow to identify, validate, and investigate SSH brute-force activity.

```
Kali Linux Attack
        │
        ▼
SSH Authentication Logs
 (/var/log/auth.log)
        │
        ▼
Log Collection
        │
        ▼
Splunk Log Ingestion
        │
        ▼
Authentication Analysis
(Failed + Successful Logins)
        │
        ▼
Brute Force Detection
        │
        ▼
Alert Generation
        │
        ▼
MITRE ATT&CK Mapping
        │
        ▼
SOC Investigation Findings
```

### Workflow Overview

The simulated SSH brute-force attack generated authentication events that were collected from the Linux authentication log (`/var/log/auth.log`). These logs were ingested into Splunk Enterprise, where searches and correlation logic were used to identify repeated failed authentication attempts followed by a successful login. Detection rules, visualizations, and MITRE ATT&CK mapping were then used to document the investigation.

---

## Log Sources

### Linux Authentication Logs

* `/var/log/auth.log`

### Evidence Collected

* Failed Password Attempts
* Successful SSH Logins
* SSH Session Activity
* User Account Creation Activity

---

## Network Verification

Before launching the attack, connectivity between the attacker and target systems was verified.

### Command

```bash
ping <target_ip>
```

### Purpose

* Verify network communication
* Confirm target accessibility
* Validate lab environment setup

---

## SSH Brute Force Attack Simulation

### Launch Metasploit

```bash
msfconsole
```

### Load SSH Brute Force Module

```bash
use auxiliary/scanner/ssh/ssh_login
```

### Configure Target

```bash
set RHOSTS <target_ip>
set USERNAME msfadmin
set PASSWORD wrongpass
```

### Execute Attack

```bash
run
```

### Result

* Multiple failed SSH login attempts generated
* Authentication logs populated with attack activity

---

## Authentication Validation

### Configure Valid Credentials

```bash
set PASSWORD msfadmin
run
```

### Result

* Successful authentication achieved
* SSH session established
* Authentication logs recorded successful access

A successful SSH authentication was intentionally performed after the failed login attempts to generate both failed and successful authentication events for correlation analysis in Splunk.

---

## Log Monitoring

### Log File

```bash
/var/log/auth.log
```

### Monitoring Command

```bash
sudo tail -f /var/log/auth.log
```

### Evidence Collected

* Failed password attempts
* Accepted password events
* SSH session activity
* Authentication records

---

## Persistence Simulation

To simulate attacker persistence after successful access:

```bash
sudo adduser hackersuser
```

### Purpose

* Simulate post-compromise activity
* Demonstrate persistence through account creation
* Generate additional security-relevant log events

---

## Log Collection and Transfer

After the attack simulation and authentication validation, the Linux authentication log (`/var/log/auth.log`) was securely transferred to the Splunk Enterprise instance for centralized log analysis and investigation.

### Command

```bash
scp -o HostKeyAlgorithms=+ssh-rsa msfadmin@<target_ip>:~/auth.log .
```

### Purpose

* Secure log transfer
* Centralized log analysis
* SIEM ingestion preparation

---

## Splunk Log Ingestion

### Steps Performed

1. Open Splunk Web Interface
2. Navigate to **Settings → Add Data**
3. Select **Upload**
4. Upload `auth.log`
5. Configure Index as **main**
6. Complete ingestion process

### Verification Searches

```spl
index=main "Failed password"
```

```spl
index=main "Accepted password"
```

---

## Detection Logic

### Detection Search

```spl
index=main ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count(eval(searchmatch("Failed password"))) as failed,
count(eval(searchmatch("Accepted password"))) as success
by src_ip
| where failed > 5 AND success > 0
```

### Detection Methodology

This query:

* Extracts source IP addresses
* Counts failed login attempts
* Counts successful logins
* Correlates authentication activity
* Identifies brute-force attacks resulting in successful access

### Detection Condition

* More than five failed login attempts
* Followed by a successful login
* From the same source IP

---

## Alert Configuration

### Configuration

* Saved SPL query as Alert
* Trigger Condition: Number of Results > 0
* Time Range: Last 15 Minutes

### Verification Objectives

* Enable real-time brute-force detection
* Generate automated alerts
* Improve incident response capabilities

---

## Detection Dashboard

### Dashboard Component

Bar Chart

### Metrics

* X-Axis: Source IP Address
* Y-Axis: Number of Authentication Attempts

### Outcome

* Clear attacker identification
* Attack intensity visualization
* Improved investigation workflow

---

## Key Findings

* Multiple failed SSH authentication attempts were generated from the attacker system.
* Successful login was achieved after repeated failed attempts.
* Source IP responsible for the activity was identified through log analysis.
* Splunk correlation searches successfully detected brute-force behavior.
* Persistence activity was simulated through local user account creation.
* Real-time alerting was configured to identify future brute-force attacks.

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Evidence                                                     |
| ------------ | -------------- | ------------------------------------------------------------ |
| T1110        | Brute Force    | Multiple failed SSH login attempts                           |
| T1078        | Valid Accounts | Successful authentication after repeated failures            |
| T1136        | Create Account | Creation of local user account during persistence simulation |

---

## Screenshot References

Figure 1

Authentication Log
(Screenshots/01_attack_execution.png)

Figure 2

Splunk Detection Dashboard
(Screenshots/04_dashboard_visualization.png)

---

## Skills Demonstrated

* Splunk SIEM
* Security Monitoring
* Log Analysis
* Threat Detection
* Incident Investigation
* Attack Simulation
* Detection Engineering
* Alert Creation
* Event Correlation
* Linux Authentication Log Analysis

---

## Real-World Relevance

This project demonstrates a common SOC use case used to identify brute-force attacks against SSH services. Security analysts frequently investigate authentication anomalies, detect account compromise attempts, build correlation-based detections, and respond to potential unauthorized access attempts.

---

## Conclusion

This project demonstrates practical SOC analyst skills by simulating a real-world SSH brute-force attack and implementing detection mechanisms using Splunk SIEM. Through authentication log analysis, event correlation, alert creation, persistence simulation, and investigation methodology, the project highlights how security analysts detect, investigate, and respond to unauthorized access attempts in enterprise environments.

---

