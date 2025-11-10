# Lab Report — Metasploitable2 SSH brute-force (Nov 7, 2025)

**Author:** Brendan Briody  
**Date:** Nov 7, 2025  
**Repo path:** `labs/metasploitable-ssh-lab/`

---

## TL;DR

Small, repeatable home‑lab demonstrating discovery, a controlled SSH authentication exercise, packet capture, log correlation, a temporary mitigation, and evidence collection. All activity performed on isolated VirtualBox Host‑Only network with permission (personal lab).

This is a single, straight-through, step-by-step lab report for an isolated VirtualBox host-only environment with Kali (attacker) `192.168.56.2` and Metasploitable2 (target) `192.168.56.3`. The objective: demonstrate scanning, a controlled SSH authentication exercise, network capture, log extraction, a temporary mitigation, evidence collection, and documentation for this portfolio. All actions were performed on VMs I controled inside a Host-Only virtual network. Safety reminder: I did not run these steps against systems I do not own or have explicit permission to test.

---

## Table of contents

1. [Objective](#objective)
2. [Scope & Safety](#scope--safety)
3. [Prerequisites](#prerequisites)
4. [Topology & IPs](#topology--ips)
5. [Artifacts included](#artifacts-included)
6. [Step‑by‑step procedure](#step-by-step-procedure)
7. [Evidence correlation & verification](#evidence-correlation--verification)
8. [Mitigation & cleanup](#mitigation--cleanup)
9. [Troubleshooting (quick)](#troubleshooting-quick)
10. [Lessons learned & recommendations](#lessons-learned--recommendations)
11. [Appendix: Useful commands / hashes / screenshots tips](#appendix-useful-commands--hashes--screenshots-tips)

---

<a id="objective"></a>

## Objective

Demonstrate reconnaissance and a controlled SSH authentication test against a purposely vulnerable VM (Metasploitable2), capture network traffic for evidence, extract relevant logs for correlation, apply a temporary network mitigation, and prepare a GitHub‑ready evidence package.

<a id="scope--safety"></a>

## Scope & Safety

* **Scope:** Isolated VirtualBox Host‑Only network. Kali (attacker) → Metasploitable2 (target).
* **Safety reminder:** Do **not** run these steps against any system you do not own or have explicit permission to test. All commands below were executed on isolated VMs under my control.

<a id="prerequisites"></a>

## Prerequisites

* VirtualBox with Host‑Only network `vboxnet0` configured.
* Kali Linux VM (attacker).
* Metasploitable2 VM (target).
* Tools: `nmap`, `tcpdump`, `wireshark`/`tshark`, `hydra`, `sshpass`, `git`.

<a id="topology--ips"></a>

## Topology & IPs

```
Kali (attacker)          192.168.56.2
Metasploitable2 (target) 192.168.56.3
Network: VirtualBox Host-Only (isolated)
```

---

<a id="artifacts-included"></a>

## Artifacts included in `labs/metasploitable-ssh-lab/`

* `lab-report.md` — this file (improved layout).
* `nmap_metasploitable.txt` — full Nmap output.
* `ssh_bruteforce.pcap` — tcpdump capture (open in Wireshark).
* `auth_log_snippet.txt` — `/var/log/auth.log` excerpt from target.
* `hydra_ssh_out.txt` — Hydra output and errors.

---

<a id="step-by-step-procedure"></a>

## Step‑by‑step procedure

> All commands were executed on Kali and Metasploitable unless otherwise noted.

### 0) Preperation (Import VM's + Snapshots)

Imported Kali and Metasploitable2 into VirtualBox. Configured both VMs to use a Host-Only adapter (`vboxnet0`) so the environment is isolated. Booted both VMs and verify IPs: Kali should be `192.168.56.2`, Metasploitable `192.168.56.3`. Took snapshots BEFORE testing: Kali snapshot `kali-clean-base`; Metasploitable snapshot `msf-clean-base`. The Snapshots let me return to a known baseline if anything breaks.

```bash
# Imported VMs and confirmed network
# Took snapshots BEFORE testing
# Example snapshot names used: Kali: kali-clean-base and Metasploitable: msf-clean-base
```

### 1) Verified Connectivity (Kali)

Ran: `ip a` and `ping -c 3 192.168.56.3`. Expected: `3 packets transmitted, 3 received, 0% packet loss` — this confirmed host-only connectivity between the attacker and target.

```bash
ip a            # confirm interface
ping -c 3 192.168.56.3
```

### 2) Reconnaissance — Nmap (Kali)

Ran a SYN + version scan and saved output: `sudo nmap -sS -sV -Pn -oN ~/Downloads/nmap_metasploitable.txt 192.168.56.3`. Saved `nmap_metasploitable.txt` as evidence. Example useful output lines: `22/tcp open ssh OpenSSH 4.7p1` and many legacy services (ftp, telnet, mysql, tomcat, bindshell 1524).

```bash
sudo nmap -sS -sV -Pn -oN ~/Downloads/nmap_metasploitable.txt 192.168.56.3
```

### 3) Started Packet Capture — TCPDump (Kali)

Started capture prior to authentication attempts: `sudo tcpdump -i eth0 -w ~/Downloads/ssh_bruteforce.pcap tcp port 22` and pressed Ctrl+C to stop after tests. On stop, I saw `^C47 packets captured`. The file `ssh_bruteforce.pcap` is the capture to open in Wireshark for analysis. Note: confirmed `eth0` is the correct interface for the host-only adapter.

```bash
sudo tcpdump -i eth0 -w ~/Downloads/ssh_bruteforce.pcap tcp port 22
# Ctrl+C when done
```

### 4) Controlled SSH authentication test (Hydra) — Observed host-key error

Used Hydra command: `sudo hydra -l msfadmin -P ~/Documents/lab-home/small_wordlist.txt ssh://192.168.56.3 -t 4 -f -o ~/Downloads/hydra_ssh_out.txt -V`. Hydra failed with `kex error : no match for method server host key algo: server [ssh-rsa,ssh-dss], client [ssh-ed25519,ecdsa-sha2-...]` this indicates a host-key algorithm mismatch: Metasploitable (old OpenSSH) advertises legacy host-key types that modern clients may reject by default. Saved `hydra_ssh_out.txt` as evidence of the attempt and error.

```bash
sudo hydra -l msfadmin -P ~/Documents/lab-home/small_wordlist.txt \
  ssh://192.168.56.3 -t 4 -f -o ~/Downloads/hydra_ssh_out.txt -V
```

### 5) Controlled Workaround & Remote auth log extraction (Kali)

Because modern tools may reject old host keys, used `sshpass` with explicit options in this isolated lab to fetch auth logs for correlation:
`mkdir -p ~/Downloads/lab_evidence_metasploitable`
`sshpass -p 'msfadmin' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o PreferredAuthentications=password -o PubkeyAuthentication=no -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.56.3 'sudo tail -n 120 /var/log/auth.log' > ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt`
This saves `auth_log_snippet.txt` locally; it contains recent `sshd` and `sudo` entries to correlate with the pcap. Note: `sshpass` and the `ssh` options are used here only in an isolated controlled environment. Do not expose credentials or secrets in public repos.

```bash
mkdir -p ~/Downloads/lab_evidence_metasploitable
sshpass -p 'msfadmin' ssh -o StrictHostKeyChecking=no \
  -o UserKnownHostsFile=/dev/null \
  -o PreferredAuthentications=password \
  -o PubkeyAuthentication=no \
  -o HostKeyAlgorithms=+ssh-rsa \
  -o PubkeyAcceptedKeyTypes=+ssh-rsa \
  msfadmin@192.168.56.3 'sudo tail -n 120 /var/log/auth.log' \
  > ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt
```

**Security note:** do not commit secrets; collected only necessary log snippets and rotate/revoke any tokens used.

### 6) Applied temporary mitigation (Iptables ran on Metasploitable2)

On the target, I applied a DROP rule to block the attacker's IP for SSH:
`sudo iptables -A INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP`
Verified: `sudo iptables -L INPUT -n -v --line-numbers`. I saw a `DROP` entry blocking `192.168.56.2` on port 22. To remove later: `sudo iptables -D INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP` or if that fails, `sudo iptables -L INPUT -n --line-numbers` then `sudo iptables -D INPUT <line-number>`.

```bash
sudo iptables -A INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP
sudo iptables -L INPUT -n -v --line-numbers
# To remove later:
# sudo iptables -D INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP
# or delete by line-number using iptables -L ... then -D INPUT <num>
```

### 7) Correlate PCAP & auth logs

Opened the capture and matched timestamps with auth log entries: used Wireshark GUI `wireshark ~/Downloads/ssh_bruteforce.pcap` to inspect packets, or summarize with `tshark -r ~/Downloads/ssh_bruteforce.pcap -T fields -e frame.number -e _ws.col.Time -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport | head -n 50`. Inspected auth log: `less ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt`. Looked for lines such as `Nov 7 16:24:02 metasploitable sshd[5308]: Accepted password for msfadmin from 192.168.56.2 port 56816 ssh2` and matched those timestamps to the pcap (SSH banner, key exchange, auth attempts) to form an evidence chain.

```bash
tshark -r ~/Downloads/ssh_bruteforce.pcap -T fields \
  -e frame.number -e _ws.col.Time -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport | head -n 50
```

* Match timestamps in `auth_log_snippet.txt` (e.g., `Nov 7 16:24:02 ... Accepted password for msfadmin from 192.168.56.2`) with pcap frame times to form an evidence chain.

### 8) Collect Local Evidence into repo folder (Kali)

Copied evidence into my repository lab folder: `cd ~/Downloads/brendan-cybersec-portfolio` then `mkdir -p labs/metasploitable-ssh-lab` and `cp ~/Downloads/lab_evidence_metasploitable/* labs/metasploitable-ssh-lab/` and `cp ~/Downloads/nmap_metasploitable.txt ~/Downloads/ssh_bruteforce.pcap ~/Downloads/hydra_ssh_out.txt labs/metasploitable-ssh-lab/`. Verified the folder contains: `lab-report.md` (this file), `nmap_metasploitable.txt`, `ssh_bruteforce.pcap`, `auth_log_snippet.txt`, `hydra_ssh_out.txt`.

```bash
cd ~/Downloads/brendan-cybersec-portfolio
mkdir -p labs/metasploitable-ssh-lab
cp ~/Downloads/nmap_metasploitable.txt labs/metasploitable-ssh-lab/
cp ~/Downloads/ssh_bruteforce.pcap labs/metasploitable-ssh-lab/
cp ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt labs/metasploitable-ssh-lab/
cp ~/Downloads/hydra_ssh_out.txt labs/metasploitable-ssh-lab/
# add lab-report.md to same folder
```

### 9) Commit & pushed to Github (Kali)

From the repo root: `git add labs/metasploitable-ssh-lab` then `git commit -m "Add Metasploitable SSH lab evidence (Nov 7 2025)"` then `git push -u origin main`. When pushing via HTTPS, used a GitHub Personal Access Token (PAT) with minimal `repo` scope in place of my password. Revoked the PAT after the push for security hygiene. See [Evidence correlation & verification](#evidence-correlation--verification) section for more.

```bash
git add labs/metasploitable-ssh-lab
git commit -m "Add Metasploitable SSH lab evidence (Nov 7 2025)"
git push -u origin main
```

### 10) Cleanup & restore baseline

After evidence collection and pushing, restored the clean snapshots in VirtualBox: opened VirtualBox Manager, selected the VM → Snapshots → chose `msf-clean-base` (or `kali-clean-base`) → Restore. Alternatively, gracefully shut down each VM with `sudo poweroff` or `sudo shutdown -h now`. If the VM console is unresponsive, toggled the VirtualBox Host Key (macOS default: Left ⌘) to capture/release keyboard and mouse.

---

<a id="evidence-correlation--verification"></a>

## Evidence correlation & verification

* `nmap_metasploitable.txt` — full Nmap scan output (service/version discovery).
* `ssh_bruteforce.pcap` — tcpdump capture of SSH traffic (open with Wireshark).
* `auth_log_snippet.txt` — snippet of `/var/log/auth.log` from the target for correlation.
* `hydra_ssh_out.txt` — Hydra run output and observed error messages.
* `lab-report.md` — this file.

---

<a id="mitigation--cleanup"></a>

## Troubleshooting (common issues)

* **Hydra: kex/host‑key algorithm error** — legacy server keys (ssh‑rsa/dss) rejected by modern client. In‑lab workaround: add `-o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa` or use `sshpass` to fetch logs. Do **not** apply on external systems.
* **sshpass missing** — `sudo apt update && sudo apt install -y sshpass` (requires internet or NAT access).
* **iptables -D failing** — list with `sudo iptables -L INPUT -n --line-numbers` then delete by index.
* **VirtualBox input capture** — click VM window or press Host Key (macOS default: Left ⌘).
* **Git auth failing** — use PAT (HTTPS) or SSH keys; check remote URL `git remote -v`.

---

<a id="lessons-learned--recommendations"></a>

## Reflection

This lab demonstrates the ability to build isolated attacker/target environments, perform safe reconnaissance with Nmap, collect and correlate network and host evidence (pcap + auth log), troubleshoot real compatibility issues (legacy SSH host-key algorithms vs modern clients), and apply a practical mitigation (iptables). The artifacts and write-up are reproducible and provide a clear hands-on example of practical troubleshooting and evidence-based thinking — exactly what junior SOC/analyst roles value.

---

<a id="appendix-useful-commands--hashes--screenshots-tips"></a>

## Appendix — Useful commands, screenshots tips

* produce a single ready‑to‑paste GitHub `lab-report.md` that *replaces* your current file (I already formatted it above), or
* create a one‑page printable checklist that you can copy into a lab notebook.

---
