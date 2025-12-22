# Network

## Table of content

- [Network](#network)
  - [Table of content](#table-of-content)
  - [List of commands](#list-of-commands)

## List of commands

Commands are split in linux / windows

- ping: Checks if the destination responds and reports the round-trip time for basic reachability.
- traceroute / tracert: Shows each hop on the path so you can see where packets slow down or stop.
- mtr / pathping: Continuously measures latency and loss per hop to catch intermittent issues.
- ip addr, ip link / ipconfig /all: Prints local IPs, MACs, and interface status so you can verify the machine’s network identity.
- ip route: Reveals the routing table to confirm which gateway and next hop the system will use.
- ip neigh: Displays IP-to-MAC entries to detect duplicates or stale ARP records on the LAN.
- ss -tulpn: Lists listening sockets and PIDs so you can confirm a service is actually bound to the expected port.
- dig: Resolves DNS records to verify the exact IPs clients will connect to.
- curl -I: Fetches only HTTP(S) headers to check status codes, redirects, and cache settings.
- tcpdump / tshark: Captures packets so you can inspect real traffic and validate what’s sent and received.
- iperf3: Measures end-to-end throughput between two hosts to separate bandwidth limits from app issues.
- ssh: Opens a secure shell on the remote machine to run checks and apply fixes directly.
- sftp: Transfers files securely so you can pull logs or push artifacts during an incident.
- nmap: Scans open ports and probes versions to confirm which services are exposed and responding.
