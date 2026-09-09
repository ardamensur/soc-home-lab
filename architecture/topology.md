# Lab Architecture

## Overview

Four virtual machines on a single VirtualBox host, connected through an
isolated internal network. No lab traffic touches the host network or the
internet except through a controlled NAT adapter used only for package
installation and updates.

## Network

- Internal network (lab-net): `192.168.10.0/24`
- Every VM has a lab-net adapter for lab traffic
- The Wazuh Manager and the Windows hosts also have a NAT adapter for
  outbound updates only

## Hosts

| VM | lab-net IP | Role |
|----|-----------|------|
| Wazuh Manager | 192.168.10.1 | SIEM (Wazuh all-in-one) and Suricata IDS |
| Kali | 192.168.10.10 | Attacker and Nessus scanner |
| Windows Endpoint | 192.168.10.20 | Domain-joined workstation with Sysmon |
| Domain Controller | 192.168.10.5 | Active Directory and DNS (lab.local) |

## Data flow

    [ Kali ]        [ Windows Endpoint ]      [ Domain Controller ]
       |                    |                          |
       |  attacks           | Wazuh agent              | Wazuh agent
       |                    | (Sysmon + Security log)  | (Security log)
       v                    v                          v
    ------------------- lab-net (192.168.10.0/24) -------------------
                            |
                            v
                   [ Wazuh Manager ]
                   - Wazuh agents report here
                   - Suricata sniffs lab-net (promiscuous mode)
                   - eve.json feeds Suricata alerts into Wazuh
                   - single dashboard for network + endpoint + AD

## Design notes

- The Wazuh Manager's lab-net adapter runs in promiscuous mode so Suricata
  can see traffic between other VMs, not just traffic addressed to the
  manager.
- Suricata custom rules use `any -> $HOME_NET` rather than
  `$EXTERNAL_NET -> $HOME_NET`, because in this lab the attacker sits inside
  the monitored network.
- Putting a Wazuh agent on the Domain Controller is deliberate: domain
  authentication is logged on the DC, so that is where AD attacks are caught.

## Software

- Wazuh 4.11.2 (Manager, Indexer, Dashboard)
- Suricata 8.0.6 with ET Open ruleset
- Sysmon with the SwiftOnSecurity configuration
- Nessus Essentials 10.12.4
- Windows Server 2022, Windows 10 Pro, Ubuntu Server 24.04, Kali Linux
