# Metasploitable2 — SSH brute-force lab (recorded evidence)
**Date:** Nov 7, 2025  
**Author:** Brendan Briody

## Objective
Demonstrate reconnaissance and a controlled SSH brute-force on an isolated home lab, capture network traffic, and apply a simple mitigation (iptables) to block the attacker. All activity performed on isolated Host-Only network.

## Topology
- **Kali (attacker)** — 192.168.56.2  
- **Metasploitable2 (target)** — 192.168.56.3  
- VirtualBox host-only network (isolated)

## Tools & commands used
- `nmap -sS -sV -Pn -oN nmap_metasploitable.txt 192.168.56.3` — reconnaissance  
- `tcpdump -i eth0 -w ssh_bruteforce.pcap tcp port 22` — capture SSH traffic  
- `hydra` (attempted) — password guessing; compatibility issue with target host-key  
- `sshpass` loop — controlled password attempts compatible with target host-key  
- `iptables -A INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP` — temporary mitigation

## Key results / findings
- Nmap discovered multiple intentionally vulnerable services (ssh, ftp, telnet, smb, mysql, etc.). See `nmap_metasploitable.txt`.  
- Controlled login attempts produced `Accepted password` entries in `/var/log/auth.log` (see `auth_log_snippet.txt`).  
- `ssh_bruteforce.pcap` contains captured SSH handshake + payload bytes showing the connection.  
- Blocking Kali from Metasploitable via iptables (DROP on port 22) produced connection timeouts from Kali — validating mitigation.

## Files included
- `nmap_metasploitable.txt` — scan output (human readable)  
- `ssh_bruteforce.pcap` — tcpdump capture of SSH traffic (pcap)  
- `hydra_ssh_out.txt` — hydra output (if any)  
- `auth_log_snippet.txt` — relevant `/var/log/auth.log` snippet showing accepted/failed attempts  
- `README.md` — this file

## Notes & next steps
- **Safety:** All testing was done on an isolated Host-Only network. Do not run these scans against systems you do not own.  
- Reproduce steps in a lab report on GitHub with supporting screenshots (optional).  
- Next labs: demonstrate detection (log correlation), install `fail2ban` on target and re-test, and create a GitHub lab report with step-by-step commands + timestamps.

