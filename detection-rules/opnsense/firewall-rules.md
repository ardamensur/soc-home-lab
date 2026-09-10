# OPNsense Firewall Rules

Segmentation rules controlling traffic between the two lab segments:
lab-net (192.168.10.0/24, servers) and user-net (192.168.20.0/24, client and attacker).
OPNsense sits between them (LAN 192.168.10.254, WAN 192.168.20.1).

## WAN interface rules (user-net to lab-net)

Rules are evaluated top to bottom, first match wins.

| # | Action | Protocol | Source | Destination | Log | Purpose |
|---|--------|----------|--------|-------------|-----|---------|
| 1 | Block | ICMP | 192.168.20.0/24 | 192.168.10.5 | yes | Block ping from user-net to the Domain Controller (demo of selective control) |
| 2 | Pass | any | 192.168.20.0/24 | any | yes | Allow user-net to reach lab-net services (DNS, LDAP, Kerberos for domain membership) |

## Source NAT (outbound)

Mode: Hybrid. Traffic from user-net (192.168.20.0/24) heading to lab-net is
translated to the OPNsense LAN address (192.168.10.254), so servers reply to
the firewall and the firewall routes the answer back. This removes the need
for a default gateway on each server.

## Design notes

- The block rule is placed above the pass rule so it matches first. This is
  how a firewall enforces selective access: ICMP is denied while DNS on the
  same source and destination is allowed.
- "Block private networks" and "block bogons" are disabled on WAN, because in
  this lab the WAN side is itself a private segment (user-net), not the internet.
- All rules log, so allowed and denied traffic both show up in the OPNsense
  live log and can be forwarded to the SIEM.
