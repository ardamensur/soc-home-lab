# SMB Brute Force: Network and Endpoint Correlation

## Objective

Detect an SMB brute force attack against a Windows host, and show why a
network sensor and an endpoint sensor together give better visibility than
either one alone.

## Attack

From the Kali box, repeated SMB authentication attempts against the Windows
endpoint using wrong passwords:

    for i in $(seq 1 15); do smbclient //192.168.10.20/C$ -U "Administrator%wrongpass$i" -c exit; done

The account locked out partway through, which is the expected real world
outcome of a brute force against a hardened account.

## Detection

Two independent detections fired for the same attack:

**Endpoint (Wazuh, rule 100004).** Windows logged the failed logons as Event
4625 and the custom rule correlated five or more failures into a brute force
alert (level 10, T1110).

**Network (Suricata, custom SID 1000002).** Suricata saw the same traffic at
the packet level on port 445 and raised its own alert, which was forwarded
into Wazuh through eve.json.

## The key finding

The Windows event log recorded the source of the attack as `127.0.0.1`, which
is a known quirk of how Windows logs certain SMB authentication failures. On
its own, that log is useless for tracing the attacker.

Suricata, watching the wire, recorded the true source address `192.168.10.10`
(the Kali box). Correlating the network alert with the endpoint alert in the
SIEM produced a complete picture: what happened (SMB brute force), on which
host (the endpoint), and where it actually came from (the attacker's real IP).

This is the practical argument for defence in depth at the logging layer: one
source told us *what*, the other told us *who*.


## Evidence

Suricata captured the attack with the real source IP (192.168.10.10), while the Windows event log showed only 127.0.0.1:

![Suricata SMB alert with real source IP](../screenshots/suricata-smb-real-source-ip.png)

Suricata alerts forwarded into the Wazuh SIEM:

![Suricata alerts in Wazuh](../screenshots/suricata-wazuh-alerts.png)

## MITRE ATT&CK

- T1110 Brute Force (Credential Access)
