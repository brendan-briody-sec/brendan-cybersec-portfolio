# Lab Report — Metasploitable2 SSH brute-force (Nov 7, 2025)

**Author:** Brendan Briody  
**Date:** Nov 7, 2025  
**Repo path:** `labs/metasploitable-ssh-lab/`

This is a single, straight-through, step-by-step lab report for an isolated VirtualBox host-only environment with Kali (attacker) `192.168.56.2` and Metasploitable2 (target) `192.168.56.3`. The objective: demonstrate scanning, a controlled SSH authentication exercise, network capture, log extraction, a temporary mitigation, evidence collection, and documentation for a portfolio. All actions were performed on VMs I control inside a Host-Only virtual network. Safety reminder: do not run these steps against systems you do not own or have explicit permission to test.

0) PREPARATION (IMPORT VMS + SNAPSHOTS)  
Import Kali and Metasploitable2 into VirtualBox. Configure both VMs to use a Host-Only adapter (`vboxnet0`) so the environment is isolated. Boot both VMs and verify IPs: Kali should be `192.168.56.2`, Metasploitable `192.168.56.3`. Take snapshots BEFORE testing: Kali snapshot `kali-clean-base`; Metasploitable snapshot `msf-clean-base`. Snapshots let you return to a known baseline if anything breaks.

1) VERIFY CONNECTIVITY (RUN ON KALI)  
Run: `ip a` and `ping -c 3 192.168.56.3`. Expected: `3 packets transmitted, 3 received, 0% packet loss` — this confirms host-only connectivity between attacker and target.

2) RECONNAISSANCE — NMAP (RUN ON KALI)  
Run a SYN + version scan and save output: `sudo nmap -sS -sV -Pn -oN ~/Downloads/nmap_metasploitable.txt 192.168.56.3`. Save `nmap_metasploitable.txt` as evidence. Example useful output lines you should see: `22/tcp open ssh OpenSSH 4.7p1` and many legacy services (ftp, telnet, mysql, tomcat, bindshell 1524).

3) START PACKET CAPTURE — TCPDUMP (RUN ON KALI)  
Start capture prior to authentication attempts: `sudo tcpdump -i eth0 -w ~/Downloads/ssh_bruteforce.pcap tcp port 22` and press Ctrl+C to stop after tests. On stop you may see `^C47 packets captured`. The file `ssh_bruteforce.pcap` is the capture to open in Wireshark for analysis.

4) ATTEMPTED SSH BRUTE-FORCE WITH HYDRA (RUN ON KALI) — NOTE OBSERVED ERROR  
Example Hydra command: `sudo hydra -l msfadmin -P ~/Documents/lab-home/small_wordlist.txt ssh://192.168.56.3 -t 4 -f -o ~/Downloads/hydra_ssh_out.txt -V`. If Hydra fails with `kex error : no match for method server host key algo: server [ssh-rsa,ssh-dss], client [ssh-ed25519,ecdsa-sha2-...]` this indicates a host-key algorithm mismatch: Metasploitable (old OpenSSH) advertises legacy host-key types that modern clients may reject by default. Save `hydra_ssh_out.txt` as evidence of the attempt and error.

5) CONTROLLED WORKAROUND + REMOTE LOG EXTRACTION (RUN ON KALI)  
Because modern tools may reject old host keys, use `sshpass` with explicit options in this isolated lab to fetch auth logs for correlation:  
`mkdir -p ~/Downloads/lab_evidence_metasploitable`  
`sshpass -p 'msfadmin' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o PreferredAuthentications=password -o PubkeyAuthentication=no -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.56.3 'sudo tail -n 120 /var/log/auth.log' > ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt`  
This saves `auth_log_snippet.txt` locally; it contains recent `sshd` and `sudo` entries to correlate with the pcap. Note: `sshpass` and the `ssh` options are used here only in an isolated controlled environment. Do not expose credentials or secrets in public repos.

6) APPLY TEMPORARY MITIGATION — IPTABLES (RUN ON METASPLOITABLE2)  
On the target, apply a DROP rule to block the attacker's IP for SSH:  
`sudo iptables -A INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP`  
Verify: `sudo iptables -L INPUT -n -v --line-numbers`. You should see a `DROP` entry blocking `192.168.56.2` on port 22. To remove later: `sudo iptables -D INPUT -s 192.168.56.2 -p tcp --dport 22 -j DROP` or if that fails, `sudo iptables -L INPUT -n --line-numbers` then `sudo iptables -D INPUT <line-number>`.

7) CORRELATE PCAP AND AUTH LOGS  
Open the capture and match timestamps with auth log entries: use Wireshark GUI `wireshark ~/Downloads/ssh_bruteforce.pcap` to inspect packets, or summarize with `tshark -r ~/Downloads/ssh_bruteforce.pcap -T fields -e frame.number -e _ws.col.Time -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport | head -n 50`. Inspect auth log: `less ~/Downloads/lab_evidence_metasploitable/auth_log_snippet.txt`. Look for lines such as `Nov 7 16:24:02 metasploitable sshd[5308]: Accepted password for msfadmin from 192.168.56.2 port 56816 ssh2` and match those timestamps to the pcap (SSH banner, key exchange, auth attempts) to form an evidence chain.

8) COLLECT LOCAL EVIDENCE AND PREPARE FOR COMMIT (RUN ON KALI)  
Copy evidence into your repository lab folder: `cd ~/Downloads/brendan-cybersec-portfolio` then `mkdir -p labs/metasploitable-ssh-lab` and `cp ~/Downloads/lab_evidence_metasploitable/* labs/metasploitable-ssh-lab/` and `cp ~/Downloads/nmap_metasploitable.txt ~/Downloads/ssh_bruteforce.pcap ~/Downloads/hydra_ssh_out.txt labs/metasploitable-ssh-lab/`. Verify the folder contains: `lab-report.md` (this file), `nmap_metasploitable.txt`, `ssh_bruteforce.pcap`, `auth_log_snippet.txt`, `hydra_ssh_out.txt`.

9) COMMIT & PUSH TO GITHUB (RUN ON KALI)  
From the repo root: `git add labs/metasploitable-ssh-lab` then `git commit -m "Add Metasploitable SSH lab evidence (Nov 7 2025)"` then `git push -u origin main`. When pushing via HTTPS, use a GitHub Personal Access Token (PAT) with minimal `repo` scope in place of your password. Revoke the PAT after the push for security hygiene.

10) CLEANUP & RESTORE BASELINE  
After evidence collection and pushing, restore the clean snapshots in VirtualBox: open VirtualBox Manager, select the VM → Snapshots → choose `msf-clean-base` (or `kali-clean-base`) → Restore. Alternatively, gracefully shut down each VM with `sudo poweroff` or `sudo shutdown -h now`. If the VM console is unresponsive, toggle the VirtualBox Host Key (macOS default: Left ⌘) to capture/release keyboard and mouse.

Quick troubleshooting (common issues)  
- Hydra host-key algorithm error: If you see `kex error : no match for method server host key algo`, the target offers legacy key types (ssh-rsa / ssh-dss). In-lab use `-o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa` or `sshpass` for controlled commands; do not apply these fixes against systems you do not control.  
- `sshpass` missing: run `sudo apt update && sudo apt install -y sshpass` on Kali (requires temporary internet/NAT).  
- `iptables -D` failing with "bad rule": list rules `sudo iptables -L INPUT -n --line-numbers` then delete by index with `sudo iptables -D INPUT <line-number>`.  
- VirtualBox VM keyboard/mouse not captured: click the VM window or press the Host Key (macOS default: Left ⌘) to toggle input capture; restore snapshot if VM state is aborted.  
- No characters shown when typing password: Terminal suppresses echo — type the password and press Enter even though nothing displays.  
- Git push failing with authentication: use a PAT (not account password) or set up SSH keys for Git.

Evidence files included in this folder  
- `nmap_metasploitable.txt` — full Nmap scan output (service/version discovery).  
- `ssh_bruteforce.pcap` — tcpdump capture of SSH traffic (open with Wireshark).  
- `auth_log_snippet.txt` — snippet of `/var/log/auth.log` from the target for correlation.  
- `hydra_ssh_out.txt` — Hydra run output and observed error messages.  
- `lab-report.md` — this file.

Hiring-facing summary (one paragraph)  
This lab demonstrates the ability to build isolated attacker/target environments, perform safe reconnaissance with Nmap, collect and correlate network and host evidence (pcap + auth log), troubleshoot real compatibility issues (legacy SSH host-key algorithms vs modern clients), and apply a practical mitigation (iptables). The artifacts and write-up are reproducible and provide a clear hands-on example of practical troubleshooting and evidence-based thinking — exactly what junior SOC/analyst roles value.

End of file — paste this markdown into `labs/metasploitable-ssh-lab/lab-report.md`, commit and push it along with the evidence files listed above.
