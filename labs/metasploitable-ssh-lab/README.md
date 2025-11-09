# Metasploitable2 — SSH brute-force lab (Nov 7, 2025)

**Date:** Nov 7, 2025  
**Author:** Brendan Briody

## Objective
Demonstrate reconnaissance and a controlled SSH brute-force on an isolated home lab, capture network traffic, and apply a simple mitigation (iptables) to block the attacker. All activity performed on an isolated Host-Only network.

## Topology
- **Kali (attacker)** — 192.168.56.2  
- **Metasploitable2 (target)** — 192.168.56.3  
- VirtualBox host-only network (isolated)

## Summary
This mini-lab covers scanning the target to discover services, attempting a controlled SSH authentication test (observing a host-key algorithm compatibility issue), capturing packet traffic for evidence, extracting auth log evidence from the target, and applying an iptables block as an immediate mitigation. Snapshots were taken before testing and all work was performed on an isolated local network.

---

## Tools & commands (high-level)
- `nmap -sS -sV -Pn -oN nmap_metasploitable.txt 192.168.56.3` — reconnaissance  
- `tcpdump -i eth0 -w ssh_bruteforce.pcap tcp port 22` — capture SSH traffic  
- `hydra` (attempted) — password guessing; compatibility issue with host-key algorithms  
- `sshpass` + explicit SSH options — controlled login attempts / evidence extraction  
- `iptables -A INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP` — temporary mitigation

---

## Key results / findings
- Nmap discovered multiple intentionally vulnerable services (ssh, ftp, telnet, smb, mysql, etc.). See `nmap_metasploitable.txt`.  
- Controlled login attempts produced `Accepted password` entries in `/var/log/auth.log` (see `auth_log_snippet.txt`).  
- `ssh_bruteforce.pcap` contains captured SSH handshake + payload bytes showing the connection.  
- Blocking Kali from Metasploitable via iptables (DROP on port 22) produced connection timeouts from Kali — validating mitigation.

---

## Evidence files (in this folder)
- `README.md` — this summary (you are here).  
- `lab-report.md` — **detailed step-by-step** lab report with commands, troubleshooting, findings (recommended to add).  
- `nmap_metasploitable.txt` — full Nmap output used for service discovery.  
- `ssh_bruteforce.pcap` — tcpdump capture of the SSH traffic; open in Wireshark for analysis.  
- `auth_log_snippet.txt` — extracted `/var/log/auth.log` lines from the target (shows SSH/sudo entries).  
- `hydra_ssh_out.txt` — Hydra attempts & notes about the host-key compatibility error.

---

## Quick findings
- Nmap revealed multiple open services (SSH, ftp, telnet, MySQL, Tomcat, etc.). See `nmap_metasploitable.txt`.
- A host-key algorithm compatibility issue prevented direct Hydra brute-forcing; explicit SSH options resolved the mismatch for lab testing. See `hydra_ssh_out.txt`.
- Packet capture + auth log snippet provide a correlated evidence chain: traffic in `ssh_bruteforce.pcap` maps to events in `auth_log_snippet.txt`.
- Temporary iptables mitigation successfully dropped SSH connections from the attacker host IP (used for demonstration).

---

## Ethics & lab hygiene
- All testing performed only on my own VMs on a VirtualBox host-only network.  
- Snapshots were taken before tests and restored as needed.  
- No real-world or third-party hosts were targeted.

---

## Notes & next steps
- **Safety:** All testing was done on an isolated Host-Only network. Do not run these scans against systems you do not own.  
- Reproduce steps in a lab report on GitHub with supporting screenshots (optional).  
- Next labs: demonstrate detection (log correlation), install `fail2ban` on target and re-test, and create a GitHub lab report with step-by-step commands + timestamps.
