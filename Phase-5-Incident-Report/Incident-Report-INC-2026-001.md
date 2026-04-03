# Incident Report — INC-2026-001

**Classification:** Confidential  
**Status:** Resolved (Controlled Lab Simulation)  
**Analyst:** Charles  
**Date:** April 3, 2026  

---

## Incident Summary

| Field | Details |
|---|---|
| **Incident ID** | INC-2026-001 |
| **Date Detected** | April 3, 2026 |
| **Time Detected** | 15:06:37 CDT |
| **Severity** | Critical (Level 12) |
| **Attack Type** | Brute Force — SSH Authentication |
| **Status** | Resolved |
| **Analyst** | Charles |

---

## Executive Summary

On April 3, 2026 at 15:06:37 CDT, the Wazuh SIEM detected a Critical severity 
brute-force authentication attack targeting the SSH service on the Ubuntu server 
(ubuntu-VMware-Virtual-Platform). The attack originated from 127.0.0.1 and 
consisted of repeated failed SSH login attempts against the user account "ubuntu". 

The attack was detected by a custom Wazuh detection rule (ID 100001) authored 
specifically to identify brute-force patterns — triggering when 8 or more 
authentication failures occur within a 120 second window. No successful 
authentication was achieved. The attack was a controlled simulation conducted 
using Hydra as part of a SOC home lab exercise.

---

## Timeline of Events

| Time (CDT) | Event |
|---|---|
| 15:00:48 | First SSH authentication failures detected by Wazuh rule 2502 |
| 15:06:35 | Hydra simultaneous attack floods SSH with 8+ failures |
| 15:06:37 | Custom rule 100001 fires — Level 12 Critical alert generated |
| 15:11:00 | Analyst confirmed alert in Wazuh dashboard |
| 15:12:00 | Full alert details reviewed and documented |

---

## Technical Analysis

| Field | Value |
|---|---|
| **Attack Type** | SSH Brute Force |
| **Source IP** | 127.0.0.1 (localhost — simulated attacker) |
| **Target Host** | ubuntu-VMware-Virtual-Platform |
| **Target User** | ubuntu |
| **Target Service** | SSH (OpenSSH) |
| **Log Source** | /var/log/auth.log |
| **Tool Used** | Hydra (THC) |
| **Total Failures** | 8+ within 120 seconds |
| **Successful Logins** | 0 |

---

## Detection Analysis

### Rules Triggered

| Rule ID | Level | Description | Trigger |
|---|---|---|---|
| 5760 | 5 | sshd: authentication failed | Individual failure |
| 2502 | 10 | syslog: User missed the password more than one time | Repeated failures |
| 100001 | 12 | Charles-SOC: Brute Force Attack Detected | 8+ failures in 120s |

### MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Technique ID** | T1110 |
| **Technique** | Brute Force |
| **Sub-technique** | T1110.001 — Password Guessing |
| **Tactic** | Credential Access |
| **Tactic** | Lateral Movement (T1021.004 — SSH) |

### Compliance Frameworks Triggered Automatically

| Framework | Control |
|---|---|
| PCI-DSS | 11.4 |
| GDPR | IV_35.7.d |
| NIST 800-53 | AU.14, AC.7 |
| HIPAA | 164.312.b |

---

## Detection Method

The attack was detected through two layers:

1. **Built-in Wazuh rules** — Rules 5760 and 2502 detected individual 
authentication failures as they occurred, mapping them to MITRE T1110 
automatically.

2. **Custom detection rule 100001** — A custom rule authored by the analyst 
during this exercise monitored rule 2502 firing repeatedly and escalated to 
Critical (Level 12) when the threshold of 8 failures within 120 seconds was 
crossed. This rule demonstrates detection engineering — the ability to write 
targeted rules that reduce noise and surface only meaningful threat patterns.

---

## Impact Assessment

| Category | Assessment |
|---|---|
| **Data Compromise** | None |
| **Unauthorized Access** | Not achieved — 0 successful logins |
| **Service Disruption** | None |
| **Business Impact** | None (controlled lab environment) |
| **Persistence Established** | No |

---

## Root Cause

Simulated brute-force attack executed using Hydra against the local SSH service. 
In a real environment this pattern indicates a threat actor systematically 
attempting to gain unauthorized access through credential guessing — one of the 
most common initial access techniques used by attackers.

---

## Containment & Recommendations

In a real production environment the following response actions would be taken:

| Priority | Action |
|---|---|
| **Immediate** | Block source IP at firewall level |
| **Immediate** | Lock targeted user account pending investigation |
| **Short-term** | Implement account lockout policy after 5 failed attempts |
| **Short-term** | Restrict SSH access to known IPs or VPN only |
| **Short-term** | Disable password authentication — enforce SSH key pairs only |
| **Long-term** | Deploy Multi-Factor Authentication on all remote access |
| **Long-term** | Configure Wazuh Active Response to auto-block offending IPs |

---

## Detection Engineering Notes

Custom rule 100001 was authored during this exercise to demonstrate detection 
engineering capability. The rule logic:

- Monitors parent rule 2502 (repeated authentication failures)
- Triggers at frequency threshold of 8 events within 120 seconds  
- Escalates severity to Level 12 (Critical)
- Maps to MITRE T1110 (Brute Force)
- Triggers compliance alerts for PCI-DSS and GDPR automatically

This approach mirrors real SOC detection engineering workflows where analysts 
tune and write custom rules to reduce alert fatigue and surface high-confidence 
detections.

---

## Evidence

| Item | Description |
|---|---|
| Screenshot 1 | Wazuh dashboard — authentication failure spike |
| Screenshot 2 | Wazuh dashboard — MITRE ATT&CK detection (Password Guessing, Brute Force, SSH) |
| Screenshot 3 | Alert triage — expanded event showing source IP, rule details, full log |
| Screenshot 4 | Alert triage — rule compliance mapping (GDPR, HIPAA, NIST, PCI-DSS) |
| Screenshot 5 | Terminal — rule 100001 firing in alerts.json |
| Screenshot 6 | Dashboard — rule 100001 visible with Level 12 and custom description |
| Screenshot 7 | Full alert detail — rule.id 100001, all fields confirmed |

---

## Lessons Learned

- Sysmon on Windows endpoints provides significantly richer telemetry than 
  default Windows logging
- Custom detection rules allow analysts to reduce noise and build high-confidence 
  alerts tailored to the environment
- Wazuh automatically maps alerts to MITRE ATT&CK, GDPR, HIPAA, PCI-DSS and 
  NIST — reducing manual compliance work
- SSH exposed without key-based authentication represents a significant attack 
  surface even on internal networks

---

*This report was produced as part of a SOC Home Lab project demonstrating 
end-to-end threat detection and incident response capability using Wazuh 4.8 
SIEM on Ubuntu 24.04 with a Windows endpoint monitored via Wazuh agent and 
Sysmon.*