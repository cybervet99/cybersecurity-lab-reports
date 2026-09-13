# Penetration Testing Lab: Exploiting Vulnerable Workstation (vsftpd RCE)

## Methodology & Execution Steps

### Phase 1: Network Reconnaissance & Host Discovery
* **Step 1.1:** Initiated the reconnaissance phase using Zenmap (the GUI frontend for Nmap) on the primary examiner workstation.
* **Step 1.2:** Executed a ping sweep across the target local network subnet `172.30.0.0/24` to discover live systems.
* **Step 1.3:** Identified target IP `172.30.0.55` as an active host on the network.

### Phase 2: Service Enumeration & Port Scanning
* **Step 2.1:** Configured and launched an "Intense Scan" in Zenmap targeting `172.30.0.55`.
* **Step 2.2:** Filtered through the 1,000 total scanned ports; 977 ports returned as closed.
* **Step 2.3:** Enumerated 23 open TCP ports and logged their running service versions:
  * **Port 21/tcp:** `vsftpd 2.3.4` (FTP)
  * **Port 22/tcp:** `OpenSSH 4.7p1`
  * **Port 23/tcp:** `Linux telnetd`
  * **Port 25/tcp:** `Postfix smtpd`
  * **Port 53/tcp:** `ISC BIND 9.4.2`
  * **Port 80/tcp:** `Apache httpd 2.2.8`
  * **Port 111/tcp:** `rpcbind 2`
  * **Port 139/tcp & 445/tcp:** `Samba smbd 3.X-4.X`
  * **Port 512/tcp, 513/tcp, 514/tcp:** Legacy execution/login/shell services (`netkit-rsh`)
  * **Port 1099/tcp:** `Java RMI Registry`
  * **Port 1524/tcp:** `Metasploitable root shell`
  * **Port 2049/tcp:** `NFS 2-4`
  * **Port 2121/tcp:** `ProFTPD 1.3.1`
  * **Port 3306/tcp:** `MySQL 5.0.51a-3ubuntu5`
  * **Port 5432/tcp:** `PostgreSQL DB 8.3.0-8.3.7`
  * **Port 5900/tcp:** `VNC protocol 3.3`
  * **Port 6000/tcp:** `X11` (Access Denied)
  * **Port 6667/tcp:** `UnrealIRCd`
  * **Port 8009/tcp:** `Apache Jserv Protocol v1.3`
  * **Port 8180/tcp:** `Apache Tomcat/Coyote JSP engine 1.1`

### Phase 3: Vulnerability Assessment & Analysis
* **Step 3.1:** Automated an enterprise vulnerability scan against target `172.30.0.55` using Tenable Nessus.
* **Step 3.2:** Evaluated the severity distribution dashboard: **9 Critical**, **7 High**, **18 Medium**, **5 Low**, and **67 Info** findings.
* **Step 3.3:** Analyzed top critical vulnerabilities flagged by Nessus:
  * **CVE-2011-2523 / Plugin 1088 (CVSS 9.8 / 10.0):** vsftpd 2.3.4 Backdoor Execution / Bind Shell Detection
  * **Plugin 134862 (CVSS 9.8):** Apache Tomcat AJP Connector Request Injection (Ghostcat)
  * **Plugin 33850 (CVSS 10.0):** Unix Operating System Unsupported Version Detection
  * **Plugin 34460 (CVSS 10.0):** Unsupported Web Server Detection
  * **Plugin 32314 & 32321 (CVSS 10.0):** Debian OpenSSH/OpenSSL PRNG Weakness
  * **Plugin 11356 (CVSS 10.0):** NFS Exported Share Information Disclosure
  * **Plugin 61708 (CVSS 10.0):** VNC Server 'password' Password
* **Step 3.4:** Inspected **Nessus Plugin ID 52703**, confirming service identification for `vsftpd v2.3.4` listening on Port 21.
* **Step 3.5:** Cross-referenced threat intelligence regarding `vsftpd v2.3.4` (CVE-2011-2523, CVSS v2 score 10.0 / CVSS v3 score 9.8), which contains an unauthenticated backdoor triggered by sending a smiley face (`:)`) in the username parameter, opening a bound root shell on port 6200.

### Phase 4: Exploitation & Privilege Verification
* **Step 4.1:** Launched the Metasploit Framework console on Kali Linux.
* **Step 4.2:** Loaded the exploit module: `exploit/unix/ftp/vsftpd_234_backdoor`.
* **Step 4.3:** Configured target settings (`RHOSTS 172.30.0.55`, `RPORT 21`) and executed the module.
* **Step 4.4:** Successfully established Command Shell Session 1 (`172.30.0.7:43098 -> 172.30.0.55:6200`) on April 9, 2025, at 10:27:39 -0700.
* **Step 4.5:** Issued the identity command `whoami` to verify session permissions.
* **Step 4.6:** Confirmed return value `root`, establishing unauthenticated root privileges.

### Phase 5: Post-Exploitation & Target Auditing
* **Step 5.1:** Executed `ifconfig` within the interactive shell to inspect target network interfaces:
  * **`eth0`:** MAC Address `00:50:56:bd:19:a9` | IP `172.30.0.55` | Netmask `255.255.255.0` | Broadcast `172.30.0.255`
  * **`lo`:** IP `127.0.0.1` | Netmask `255.0.0.0`
* **Step 5.2:** Audited active network filtering rules by issuing `iptables --list`.
* **Step 5.3:** Verified that `INPUT`, `FORWARD`, and `OUTPUT` chains all default to `policy ACCEPT` without active restrictive filtering rules, explaining why the backdoor port (6200) was exposed.

---

## Technical Summary of Findings

| Metric / Parameter | Identified Value |
| :--- | :--- |
| **Target IP Address** | `172.30.0.55` |
| **Attacker IP Address** | `172.30.0.7` |
| **Primary Exploited Service** | `vsftpd 2.3.4` (Port 21/tcp) |
| **Vulnerability Identification** | CVE-2011-2523 (CVSS v3: 9.8 / CVSS v2: 10.0) |
| **Backdoor Access Port** | TCP Port `6200` |
| **Access Level Obtained** | `root` (UID 0) |
| **Nessus Critical Findings** | 9 Vulnerabilities (CVSS 9.8 - 10.0) |
| **Host Firewall State** | Completely open (`iptables` policy ACCEPT) |

---

## Lessons Learned & Remediation Strategy

* **Patch & Lifecycle Management:** Operating end-of-life or compromised software builds (like vsftpd 2.3.4) leaves systems vulnerable to instant remote code execution without needing credential attacks. Systems must be regularly updated or isolated.
* **Vulnerability Scanning Integration:** Utilizing automated scanning tools like Nessus helps detect high-risk software versions and rogue listening backdoors prior to adversary exploitation.
* **Egress & Ingress Firewall Hardening:** Unrestricted `iptables` configurations allow arbitrary ports (e.g., port 6200) to bind and serve remote sessions. Enforcing strict ingress/egress firewall rules limits lateral movement and blocks unauthorized inbound shell bindings.

---

## Appendix

### Appendix A: Examiner Workstation Specifications
* **Computer Name:** Ryan's Macbook Pro
* **OS Name & Version:** macOS Ventura 13.7.1
* **Hardware Model:** 2017 MacBook Pro 13-inch

### Appendix B: Tools Used
* Kali Linux
* Zenmap / Nmap
* Tenable Nessus
* Metasploit Framework
* PuTTY
* Remote Lab Workstation
