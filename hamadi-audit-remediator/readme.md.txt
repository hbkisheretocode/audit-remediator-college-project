## CIS Guardian: Autonomous Linux Hardening Engine
A Python & Ansible-driven framework for real-time CIS compliance and self-healing resilience.

CIS Guardian isn't just a scanner; it’s a proactive security layer for Ubuntu. It maps system states against 30+ critical CIS (Center for Internet Security) benchmarks, remediates vulnerabilities with one click, and runs a background "Drift Monitor" that automatically reverts unauthorized configuration changes.

### Project Core
audit.remediator.py – The main Python orchestrator. Handles the UI, PDF generation, and background threading.

rules.json – The intelligence layer. Maps CIS IDs to shell-level audit checks and risk severities.

remediate.yml / unharden.yml – The "Hybrid" Ansible playbook. Hardens the OS whether the system is Online or Air-gapped. unharden helps to rollback for testing purposes

cis-drift.service – The systemd unit file that turns the script into a persistent Linux daemon.

###  Quick Start
1. Dependencies
The engine requires Ansible for remediation and a few Python libraries for the interface and reporting.

Bash

sudo apt update && sudo apt install ansible -y
pip install rich fpdf2
2. Manual Execution
Launch the interactive dashboard to run a manual audit or apply the hardening baseline:

Bash

sudo python3 audit-remediator.py
3. Enabling the Background "Drift"
To ensure the system stays hardened even when you aren't watching, deploy the service:

Bash

sudo cp cis-drift.service /etc/systemd/system/
# Edit the WorkingDirectory in the .service file to your path
sudo systemctl daemon-reload
sudo systemctl enable --now cis-drift

### Controlled Domains
Kernel/FS: Disables legacy modules (cramfs, jffs2, hfs) and locks bootloader permissions.

Networking: Hardens the sysctl stack (IP Forwarding, ICMP Redirects, TCP SYN Cookies).

Auth/SSH: Enforces SSH Protocol 2, disables Root Login, and sets 90-day Password Aging.

Logging: Ensures auditd and rsyslog are not just installed, but actively running.

Permissions: Strict 0600/0644 lockdowns on /etc/shadow, /etc/passwd, and /etc/group.

###  Demonstration: The "Drift" Test
Start the Drift Monitor (Option 4 in the menu).

Open a second terminal and intentionally "break" the security:
sudo chmod 777 /etc/shadow

Watch the monitor terminal. It will detect the FAIL status within 15 seconds, trigger the Ansible remediation, and restore the permissions to PASS automatically.

### Dev Notes & Troubleshooting
Apt Locks: If you get a "No space" or "Apt lock" error while online, run sudo apt-get clean.

Offline Mode: The engine is built with ignore_errors logic. If it can't reach the internet, it will skip package installs but still harden all local files and kernel settings, reaching close to 80% compliance(because of no packages it will fail)

PDF Reports: Every manual audit generates a professional PDF in the project folder with a timestamped compliance score.


