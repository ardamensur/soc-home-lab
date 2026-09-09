# SOC Home Lab

A hands-on security operations lab built on VirtualBox. It brings together a SIEM, a network IDS, vulnerability management, and an Active Directory environment, with custom detection rules mapped to MITRE ATT&CK. The goal is to practise detection engineering the way it is done in a real SOC: generate an attack, catch it, and prove the catch.

## Architecture

Four virtual machines on an isolated internal network (192.168.10.0/24):

| VM | Role | OS |
|----|------|-----|
| Wazuh Manager | SIEM (Wazuh) + Suricata IDS | Ubuntu Server 24.04 |
| Kali | Attacker + Nessus scanner | Kali Linux |
| Windows Endpoint | Domain-joined workstation + Sysmon | Windows 10 Pro |
| Domain Controller | Active Directory + DNS (lab.local) | Windows Server 2022 |

Each Windows host and the Kali box runs a Wazuh agent that ships logs to the manager. The manager also runs Suricata, so the same traffic is seen at both the network and endpoint layers.

## Detection capabilities

| Layer | Tool | What it does |
|-------|------|--------------|
| Network IDS | Suricata | 52k+ ET Open rules plus 3 custom rules, feeds alerts into the SIEM via eve.json |
| Endpoint | Wazuh + Sysmon | 12 custom detection rules on Windows and Linux |
| Active Directory | Wazuh agent on the DC | Custom rules for domain brute force and account lockout |
| Vulnerability mgmt | Nessus Essentials | Scan, remediate, and verify workflow |

## MITRE ATT&CK coverage

Custom rules are mapped to the following techniques:

- T1046 Network Service Scanning
- T1110 Brute Force (SSH, SMB, and Active Directory domain accounts)
- T1078 Valid Accounts
- T1136.001 Create Account: Local Account
- T1059 / T1059.001 Command and Scripting Interpreter / PowerShell
- T1055 Process Injection
- T1562.001 Impair Defenses: Disable or Modify Tools
- T1548.003 Abuse Elevation Control Mechanism: Sudo

## Key results

**Network vs endpoint source attribution.** When an SMB brute force was launched from Kali, the Windows event log recorded the source as 127.0.0.1, which is useless for tracing the attacker. Suricata, watching at the packet level, recorded the real source IP (192.168.10.10). Correlating both layers in Wazuh gave full visibility that neither source had on its own.

**Active Directory brute force detection.** A custom Wazuh rule correlates repeated failed domain logons (Event 4625) on the Domain Controller into a single high severity alert mapped to T1110, instead of leaving an analyst to eyeball dozens of raw events.

**Vulnerability remediation loop.** A Nessus scan flagged SMB signing as not required (a man in the middle risk). After enforcing SMB signing through Group Policy, a rescan confirmed the finding was gone. Detect, fix, verify.

## Repository layout

- `detection-rules/` custom Wazuh and Suricata rules
- `attack-scenarios/` step by step write ups of each attack and how it was detected
- `vulnerability-management/` the Nessus scan and remediation walkthrough
- `architecture/` network topology and design notes
- `screenshots/` dashboard and alert evidence

## Note

This is a personal learning lab on an isolated network. All IP addresses are private (RFC1918) and all credentials shown in screenshots are lab only and have been rotated.
