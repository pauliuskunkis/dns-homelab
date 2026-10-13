# DNS Homelab (BIND9 on Debian)

Forwarding/caching DNS server plus an HA pair of authoritative servers for a private zone, built in VirtualBox.

| VM | IP | Role |
|---|---|---|
| DNS-Client | 192.168.1.10 | Client, uses .11 as DNS |
| DNS-CO-SRV | 192.168.1.11 | Cache-only forwarder (ACL, conditional forwarding of intranet.local) |
| DNS-Master-SRV | 192.168.1.100 | Authoritative master for intranet.local (forward + reverse zones) |
| DNS-Slave-SRV | 192.168.1.101 | Authoritative slave (zone transfer from master) |
| Router | 192.168.1.254 | Upstream resolver |

## Flow
Client → .11 → intranet.local → .100 / .101
             → everything else → .254 → internet

## Verified
- Forwarding and caching (tcpdump shows repeat queries answered from cache)
- Zone transfer master → slave
- Forwarder queries either master or slave (HA)

## Troubleshooting notes
- Static IP broke internet: gateway set to .1, real router is .254. Fix: check `ip route` before going static.
- SERVFAIL for intranet.local: DNSSEC validation failed because `local.` doesn't exist publicly. Fix: `validate-except { "intranet.local"; };` on the forwarder.
- Debian 13 has no /var/log/syslog. Use `journalctl -u named`.
- Slave zone files go in /var/cache/bind (BIND needs write access).
