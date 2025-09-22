# Task 1: Network Reconnaissance with Nmap & Wireshark

## Objective
Scan the local network to find open ports and analyze any DNS service behavior using Nmap and Wireshark/tshark. Document findings, risks, and mitigations.

---

## Files included
- `README.md` 
- `scan_results.txt` — raw Nmap output(s)
- `nmap_dns_checks.txt` — Nmap NSE script output for DNS checks
- `capture_full.pcapng` 
- `dns_capture.pcapng` 
- `screenshot1.png`, `screenshot2.png` — Wireshark screenshots

---

## What I did 
1. Identified local network range using `ipconfig` / `ifconfig`.
2. Ran an Nmap TCP SYN scan of the local network and saved results:

   ```
   nmap -sS 192.168.**.***/24 -oN scan_results.txt
``

3. Ran DNS-specific checks and version detection on hosts with port 53 open:

   ```
   nmap -sV --script=dns-zone-transfer,dns-recursion -p 53 192.168.**.*** -oN nmap_dns_checks.txt
   ```

4. Captured network traffic with Wireshark while running the scans and saved a filtered DNS capture: `dns_capture.pcapng`.

---

## Key finding 

* Host: `192.168.**.***`
* Open port: `53/tcp` — DNS service detected (example output: `53/tcp open domain`).
* Nmap service detection suggests: `ISC BIND 9.16.48 (Debian Linux)`.

**Interpretation:** The host is running a DNS server. TCP/53 is expected for zone transfers and large responses; UDP/53 is used for normal queries.

---

## Evidence attached

* `nmap_dns_checks.txt` — shows `53/tcp open domain` and `nmap` script results (zone transfer / recursion tests).
* `dns_capture.pcapng` — packet capture showing DNS packets during the tests.
* `screenshot1.png` — Wireshark view of the TCP stream containing DNS records (use Follow → TCP Stream).

---

## Risks observed and mitigation (fill with your findings)

### Risk: Zone transfer allowed (AXFR)

* **Risk description:** If AXFR is allowed to unauthorized hosts, an attacker can obtain full DNS zone content (all DNS records) revealing hostnames and internal structure.
* **Mitigation:** Restrict `allow-transfer` to specific secondary DNS server IPs, require TSIG for transfers, and firewall TCP/53 to known IPs only.

### Risk: Open resolver / recursion allowed for external clients

* **Risk description:** If the DNS server answers recursive queries from arbitrary IPs, it can be abused for cache poisoning or amplification attacks.
* **Mitigation:** Disable recursion for external clients. Use ACLs to allow recursion only for internal networks. Monitor query patterns and rate-limit.

### Risk: Unpatched DNS software

* **Risk description:** Older DNS software may contain remote-exploitable vulnerabilities.
* **Mitigation:** Upgrade DNS software to supported versions, apply security patches and enable logging/monitoring.

### Risk: Information leakage via DNS

* **Risk description:** DNS records (MX, SRV, PTR) may expose service locations and internal naming.
* **Mitigation:** Review and prune unnecessary records; use split-horizon DNS if internal records must remain private.

---

## Commands I used (record these exact commands in your report)

* Nmap scans:

  ```bash
  # SYN scan on the network
  nmap -sS 192.168.**.***/24 -oN scan_results.txt

  # Version detection + DNS NSE scripts
  nmap -sV --script=dns-zone-transfer,dns-recursion -p 53 192.168.**.*** -oN nmap_dns_checks.txt
  ```


* Wireshark / tshark captures:

  ```bash
  # capture 5 minutes on interface wlan0
  sudo tshark -i wlan0 -a duration:300 -w capture_full.pcapng

  # export DNS-only pcap from full capture
  tshark -r capture_full.pcapng -Y "dns" -w dns_capture.pcapng
  ```

---


## Example placeholder `scan_results.txt`

```
# nmap -sS 192.168.**.***/24 -oN scan_results.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-22 20:02 IST
Nmap scan report for 192.168.**.*
Host is up (0.0016s latency).
All 1000 scanned ports on 192.168.**.* are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
MAC Address: 00:50:56:C0:**.** (VMware)

Nmap scan report for 192.168.**.*
Host is up (0.00056s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
53/tcp open  domain
MAC Address: 00:50:56:F0:**.** (VMware)

Nmap scan report for 192.168.**.**
Host is up (0.00037s latency).
All 1000 scanned ports on 192.168.**.** are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
MAC Address: 00:50:56:E8:**:** (VMware)

Nmap scan report for 192.168.**.**
Host is up (0.000013s latency).
All 1000 scanned ports on 192.168.**.** are in ignored states.
Not shown: 1000 closed tcp ports (reset)

Nmap done: 256 IP addresses (4 hosts up) scanned in 11.88 seconds
```

---

