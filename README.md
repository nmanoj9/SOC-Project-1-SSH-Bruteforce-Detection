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

## Log Sources

### Linux Authentication Logs

* `/var/log/auth.log`

### Events Observed

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

## Attack Simulation Using Metasploit

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

## Successful Login Simulation

### Configure Valid Credentials

```bash
set PASSWORD msfadmin
run
```

### Result

* Successful authentication achieved
* SSH session established
* Authentication logs recorded successful access

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

### Observed Events

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

Authentication logs were transferred to Splunk for analysis.

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

### SPL Query

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

### Purpose

* Enable real-time brute-force detection
* Generate automated alerts
* Improve incident response capabilities

---

## Visualization

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

### Figure 1

Authentication Log showing failed and successful SSH login attempts.

### Figure 2

Splunk visualization displaying brute-force detection activity.

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

## Project Status

**Completed**
