# Nmap Port Scan Detection

## Objective

Detect network reconnaissance (a port scan) at the network layer with
Suricata, before any actual attack begins.

## Attack

A SYN scan from the Kali box against hosts on the lab network:

    sudo nmap -sS 192.168.10.1
    sudo nmap -sS -Pn 192.168.10.20

A SYN scan sends a stream of connection requests without completing the
handshake, which is a classic first step in reconnaissance: find out which
ports are open before choosing an attack.

## Detection

A custom Suricata rule (SID 1000001) counts SYN packets from a single source
and alerts when the rate crosses a threshold:

    alert tcp any any -> $HOME_NET any (msg:"LAB CUSTOM - Nmap SYN Scan Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 10; classtype:attempted-recon; sid:1000001; rev:1; metadata:mitre_technique_id T1046, mitre_tactic reconnaissance;)

The alert is written to eve.json and picked up by Wazuh, so reconnaissance
shows up in the same SIEM view as the later attack stages. The threshold
approach keeps a single stray connection from being flagged, while a real
scan (dozens of SYNs in seconds) trips the rule immediately.

## Why this matters

Catching the scan gives a SOC an early warning. In a real timeline, the scan
comes first, then the brute force or exploit. Seeing the recon lets an analyst
connect the later events to the same source and understand the attack as one
story rather than isolated alerts.

## MITRE ATT&CK

- T1046 Network Service Scanning (Reconnaissance / Discovery)
