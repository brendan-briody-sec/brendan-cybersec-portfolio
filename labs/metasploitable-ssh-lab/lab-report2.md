# Lab Report — Metasploitable2 SSH brute-force (Nov 7, 2025)

**Author:** Brendan Briody
**Date:** 2025-11-07
**Repo path:** `labs/metasploitable-ssh-lab/`
**Status:** ✅ Complete

---

## TL;DR

Small, repeatable home‑lab demonstrating discovery, a controlled SSH authentication exercise, packet capture, log correlation, a temporary mitigation, and evidence collection. All activity performed on isolated VirtualBox Host‑Only network with permission (personal lab).

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

## Objective

Demonstrate reconnaissance and a controlled SSH authentication test against a purposely vulnerable VM (Metasploitable2), capture network traffic for evidence, extract relevant logs for correlation, apply a temporary network mitigation, and prepare a GitHub‑ready evidence package.

## Scope & Safety

* **Scope:** Isolated VirtualBox Host‑Only network. Kali (attacker) → Metasploitable2 (target).
* **Safety reminder:** Do **not** run these steps against any system you do not own or have explicit permission to test. All commands below were executed on isolated VMs under my control.

## Prerequisites

* VirtualBox with Host‑Only network `vboxnet0` configured.
* Kali Linux VM (attacker).
* Metasploitable2 VM (target).
* Tools: `nmap`, `tcpdump`, `wireshark`/`tshark`, `hydra`, `sshpass`, `git`.

## Topology & IPs

```
Kali (attacker)        192.168.56.2
Metasploitable2 (target)192.168.56.3
Network: VirtualBox Host-Only (isolated)
```

---

## Artifacts included in `labs/metasploitable-ssh-lab/`

* `lab-report.md` — this file (improved layout).
* `nmap_metasploitable.txt` — full Nmap output.
* `ssh_bruteforce.pcap` — tcpdump capture (open in Wireshark).
* `auth_log_snippet.txt` — `/var/log/auth.log` excerpt from target.
* `hydra_ssh_out.txt` — Hydra output and errors.

---

## Step‑by‑step procedure

> All commands were executed on Kali unless otherwise noted.

### 0) Preperation (Import VM's + Snapshots)  
Imported Kali and Metasploitable2 into VirtualBox. Configured both VMs to use a Host-Only adapter (`vboxnet0`) so the environment is isolated. Booted both VMs and verify IPs: Kali should be `192.168.56.2`, Metasploitable `192.168.56.3`. Took snapshots BEFORE testing: Kali snapshot `kali-clean-base`; Metasploitable snapshot `msf-clean-base`. The Snapshots let me return to a known baseline if anything breaks.

```bash
# Imported VMs and confirmed network
# Took snapshots BEFORE testing
# Example snapshot names used: Kali: kali-clean-base and Metasploitable: msf-clean-base
```

### 1) Verify connectivity (Kali)
Ran: `ip a` and `ping -c 3 192.168.56.3`. Expected: `3 packets transmitted, 3 received, 0% packet loss` — this confirmed host-only connectivity between the attacker and target.

```bash
ip a            # confirm interface
ping -c 3 192.168.56.3
```

Expected: 3 transmitted, 3 received, 0% loss.

### 2) Reconnaissance — Nmap (Kali)

```bash
sudo nmap -sS -sV -Pn -oN ~/Downloads/nmap_metasploitable.txt 192.168.56.3
```

Save `nmap_metasploitable.txt` into the evidence folder.

### 3) Start packet capture (Kali)

```bash
sudo tcpdump -i eth0 -w ~/Downloads/ssh_bruteforce.pcap tcp port 22
# Ctrl+C when done
```

Note: confirm `eth0` is the correct interface for the host-only adapter.

### 4) Controlled SSH authentication test (Hydra) — observed host-key error

```bash
sudo hydra -l msfadmin -P ~/Documents/lab-home/small_wordlist.txt \
  ssh://192.168.56.3 -t 4 -f -o ~/Downloads/hydra_ssh_out.txt -V
```

If you see a *kex* / host-key algorithm mismatch (modern client vs legacy server), Hydra/ssh may fail. Save `hydra_ssh_out.txt` as evidence.

### 5) Controlled workaround & remote auth log extraction

Use `sshpass` with explicit host-key options **only** in this isolated lab to fetch auth logs for correlation:

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

**Security note:** do not commit secrets; collect only necessary log snippets and rotate/revoke any tokens used.

### 6) Apply temporary mitigation (on Metasploitable2)

Run this on the target VM (or via the SSH session above):

```bash
sudo iptables -A INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP
sudo iptables -L INPUT -n -v --line-numbers
# To remove later:
# sudo iptables -D INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP
# or delete by line-number using iptables -L ... then -D INPUT <num>
```

### 7) Correlate PCAP & auth logs

* Open the pcap in Wireshark: `wireshark ~/Downloads/ssh_bruteforce.pcap` and inspect SSH KEX / auth packets.
* Summarize quickly with tshark:

```bash
tshark -r ~/Downloads/ssh_bruteforce.pcap -T fields \
  -e frame.number -e _ws.col.Time -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport | head -n 50
```

* Match timestamps in `auth_log_snippet.txt` (e.g., `Nov 7 16:24:02 ... Accepted password for msfadmin from 192.168.56.2`) with pcap frame times to form an evidence chain.

### 8) Collect evidence into repo folder (Kali)

```bash
cd ~/Downloads/brendan-cybersec-portfolio
mkdir -p labs/metasploitable-ssh-lab
cp ~/Downloads/nmap_metasploitable.txt labs/metasploitable-ssh-lab/
cp ~/Downloads/ssh_bruteforce.pcap labs/metasploitable-ssh-lab/
cp ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt labs/metasploitable-ssh-lab/
cp ~/Downloads/hydra_ssh_out.txt labs/metasploitable-ssh-lab/
# add lab-report.md to same folder
```

### 9) Commit & push

```bash
git add labs/metasploitable-ssh-lab
git commit -m "Add Metasploitable SSH lab evidence (Nov 7 2025)"
git push -u origin main
```

If using HTTPS, use a temporary PAT and revoke it after use. Prefer SSH key auth for long-term workflows.

### 10) Cleanup & restore baseline

* Restore VirtualBox snapshots: `msf-clean-base`, `kali-clean-base`.
* Or gracefully shut down and reset VMs.

---

## Evidence correlation & verification

* Use timestamps (UTC/local) to link pcap frames ↔ auth log entries ↔ nmap/hydra outputs.
* Generate file hashes for non-sensitive artifacts and include them in the repo (verifies integrity):

```bash
sha256sum labs/metasploitable-ssh-lab/ssh_bruteforce.pcap
sha256sum labs/metasploitable-ssh-lab/auth_log_snippet.txt
sha256sum labs/metasploitable-ssh-lab/nmap_metasploitable.txt
```

Record the outputs in a small `evidence_hashes.txt` file so future reviewers can verify artifacts.

---

## Troubleshooting (quick)

* **Hydra: kex/host‑key algorithm error** — legacy server keys (ssh‑rsa/dss) rejected by modern client. In‑lab workaround: add `-o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa` or use `sshpass` to fetch logs. Do **not** apply on external systems.
* **sshpass missing** — `sudo apt update && sudo apt install -y sshpass` (requires internet or NAT access).
* **iptables -D failing** — list with `sudo iptables -L INPUT -n --line-numbers` then delete by index.
* **VirtualBox input capture** — click VM window or press Host Key (macOS default: Left ⌘).
* **Git auth failing** — use PAT (HTTPS) or SSH keys; check remote URL `git remote -v`.

---

## Lessons learned & recommendations

* Keep forensic chain-of-custody minimal but auditable: timestamped evidence, file hashes, and a short README describing how artifacts were created.
* Prefer non-interactive proofs (pcap + log snippet) over screenshots alone — they are more verifiable.
* Avoid committing secrets; use short‑lived PATs and revoke after push.
* When publicizing labs, consider redacting sensitive local paths and credentials from the commits (or keep evidence in a private repo until scrubbed).

---

## Appendix — Useful commands, screenshots tips

**Generate evidence hashes**

```bash
sha256sum labs/metasploitable-ssh-lab/* > labs/metasploitable-ssh-lab/evidence_hashes.txt
```

**Quick screenshot commands (Kali GUI / Metasploitable)**

* In a GUI session, use the screenshot tool or `gnome-screenshot -a` to select area.
* Or capture terminal output to a file: `script -c "nmap ..." nmap_output.txt` then open and screenshot.

**One‑line summary commit message**

```
Add Metasploitable SSH lab evidence (Nov 7 2025): nmap, pcap, auth-log, hydra-output
```

**Suggested `.gitignore`**

* If you keep raw downloads elsewhere, add appropriate entries so you only commit intended artifacts. Example: `Downloads/` or `*.pcap` if you choose to keep pcaps out of public repo.

---

### If you want, I can:

* produce a single ready‑to‑paste GitHub `lab-report.md` that *replaces* your current file (I already formatted it above), or
* create a one‑page printable checklist that you can copy into a lab notebook.

---

*End of improved layout.*
