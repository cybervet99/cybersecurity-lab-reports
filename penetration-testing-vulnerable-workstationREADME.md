# Penetration Testing Lab: Exploiting Vulnerable Workstation (vsftpd RCE)

## Methodology & Execution Steps

### Phase 1: Network Reconnaissance & Host Discovery
* **Step 1.1:** Initiated the reconnaissance phase using Zenmap (the GUI frontend for Nmap) on the primary examiner workstation[cite: 3].
* **Step 1.2:** Executed a ping sweep across the target local network subnet `172.30.0.0/24` to discover live systems[cite: 3].
* **Step 1.3:** Identified target IP `172.30.0.55` as an active host on the network[cite: 3].

### Phase 2: Service Enumeration & Port Scanning
* **Step 2.1:** Configured and launched an "Intense Scan" in Zenmap targeting `172.30.0.55`[cite: 3].
* **Step 2.2:** Filtered through the 1,000 total scanned ports; 977 ports returned as closed[cite: 3].
* **Step 2.3:** Enumerated 23 open TCP ports and logged their running service versions (as captured in **Figure 1**)[cite: 3]:
  * **Port 21/tcp:** `vsftpd 2.3.4` (FTP)[cite: 3]
  * **Port 22/tcp:** `OpenSSH 4.7p1`[cite: 3]
  * **Port 23/tcp:** `Linux telnetd`[cite: 3]
  * **Port 25/tcp:** `Postfix smtpd`[cite: 3]
  * **Port 53/tcp:** `ISC BIND 9.4.2`[cite: 3]
  * **Port 80/tcp:** `Apache httpd 2.2.8`[cite: 3]
  * **Port 111/tcp:** `rpcbind 2`[cite: 3]
  * **Port 139/tcp & 445/tcp:** `Samba smbd 3.X-4.X`[cite: 3]
  * **Port 512/tcp, 513/tcp, 514/tcp:** Legacy execution/login/shell services (`netkit-rsh`)[cite: 3]
  * **Port 1099/tcp:** `Java RMI Registry`[cite: 3]
  * **Port 1524/tcp:** `Metasploitable root shell`[cite: 3]
  * **Port 2049/tcp:** `NFS 2-4`[cite: 3]
  * **Port 2121/tcp:** `ProFTPD 1.3.1`[cite: 3]
  * **Port 3306/tcp:** `MySQL 5.0.51a-3ubuntu5`[cite: 3]
  * **Port 5432/tcp:** `PostgreSQL DB 8.3.0-8.3.7`[cite: 3]
  * **Port 5900/tcp:** `VNC protocol 3.3`[cite: 3]
  * **Port 6000/tcp:** `X11` (Access Denied)[cite: 3]
  * **Port 6667/tcp:** `UnrealIRCd`[cite: 3]
  * **Port 8009/tcp:** `Apache Jserv Protocol v1.3`[cite: 3]
  * **Port 8180/tcp:** `Apache Tomcat/Coyote JSP engine 1.1`[cite: 3]

### Phase 3: Vulnerability Assessment & Analysis
* **Step 3.1:** Automated an enterprise vulnerability scan against target `172.30.0.55` using Tenable Nessus[cite: 3].
* **Step 3.2:** Evaluated the severity distribution dashboard (as captured in **Figure 2**): **9 Critical**, **7 High**, **18 Medium**, **5 Low**, and **67 Info** findings[cite: 3].
* **Step 3.3:** Analyzed top critical vulnerabilities flagged by Nessus[cite: 3]:
  * **Plugin 134862 (CVSS 9.8):** Apache Tomcat AJP Connector Request Injection (Ghostcat)[cite: 3]
  * **Plugin 1088 (CVSS 9.8):** Bind Shell Backdoor Detection[cite: 3]
  * **Plugin 33850 (CVSS 10.0):** Unix Operating System Unsupported Version Detection[cite: 3]
  * **Plugin 34460 (CVSS 10.0):** Unsupported Web Server Detection[cite: 3]
  * **Plugin 32314 & 32321 (CVSS 10.0):** Debian OpenSSH/OpenSSL PRNG Weakness[cite: 3]
  * **Plugin 11356 (CVSS 10.0):** NFS Exported Share Information Disclosure[cite: 3]
  * **Plugin 61708 (CVSS 10.0):** VNC Server 'password' Password[cite: 3]
* **Step 3.4:** Inspected **Nessus Plugin ID 52703** (**Figure 6**), confirming service identification for `vsftpd v2.3.4` listening on Port 21[cite: 3].
* **Step 3.5:** Cross-referenced threat intelligence regarding `vsftpd v2.3.4`, which contains an unauthenticated backdoor triggered by sending a smiley face (`:)`) in the username parameter, opening a bound root shell on port 6200[cite: 3].

### Phase 4: Exploitation & Privilege Verification
* **Step 4.1:** Launched the Metasploit Framework console on Kali Linux[cite: 3].
* **Step 4.2:** Loaded the exploit module: `exploit/unix/ftp/vsftpd_234_backdoor`[cite: 3].
* **Step 4.3:** Configured target settings (`RHOSTS 172.30.0.55`, `RPORT 21`) and executed the module[cite: 3].
* **Step 4.4:** Successfully established Command Shell Session 1 (`172.30.0.7:43098 -> 172.30.0.55:6200`) on April 9, 2025, at 10:27:39 -0700 (**Figure 3**)[cite: 3].
* **Step 4.5:** Issued the identity command `whoami` to verify session permissions[cite: 3].
* **Step 4.6:** Confirmed return value `root`, establishing unauthenticated root privileges[cite: 3].

### Phase 5: Post-Exploitation & Target Auditing
* **Step 5.1:** Executed `ifconfig` within the interactive shell to inspect target network interfaces (**Figure 4**)[cite: 3]:
  * **`eth0`:** MAC Address `00:50:56:bd:19:a9` | IP `172.30.0.55` | Netmask `255.255.255.0` | Broadcast `172.30.0.255`[cite: 3]
  * **`lo`:** IP `127.0.0.1` | Netmask `255.0.0.0`[cite: 3]
* **Step 5.2:** Audited active network filtering rules by issuing `iptables --list` (**Figure 5**)[cite: 3].
* **Step 5.3:** Verified that `INPUT`, `FORWARD`, and `OUTPUT` chains all default to `policy ACCEPT` without active restrictive filtering rules, explaining why the backdoor port (6200) was exposed[cite: 3].

---

## Technical Summary of Findings

| Metric / Parameter | Identified Value |
| :--- | :--- |
| **Target IP Address** | `172.30.0.55`[cite: 3] |
| **Attacker IP Address** | `172.30.0.7`[cite: 3] |
| **Primary Exploited Service** | `vsftpd 2.3.4` (Port 21/tcp)[cite: 3] |
| **Backdoor Access Port** | TCP Port `6200`[cite: 3] |
| **Access Level Obtained** | `root` (UID 0)[cite: 3] |
| **Nessus Critical Findings** | 9 Vulnerabilities (CVSS 9.8 - 10.0)[cite: 3] |
| **Host Firewall State** | Completely open (`iptables` policy ACCEPT)[cite: 3] |

---

## Lessons Learned & Remediation Strategy

* **Patch & Lifecycle Management:** Operating end-of-life or compromised software builds (like vsftpd 2.3.4) leaves systems vulnerable to instant remote code execution without needing credential attacks[cite: 3]. Systems must be regularly updated or isolated.
* **Vulnerability Scanning Integration:** Utilizing automated scanning tools like Nessus helps detect high-risk software versions and rogue listening backdoors prior to adversary exploitation[cite: 3].
* **Egress & Ingress Firewall Hardening:** Unrestricted `iptables` configurations allow arbitrary ports (e.g., port 6200) to bind and serve remote sessions[cite: 3]. Enforcing strict ingress/egress firewall rules limits lateral movement and blocks unauthorized inbound shell bindings.

---

## Appendix

### Appendix A: Examiner Workstation Specifications
* **Computer Name:** Ryan's Macbook Pro[cite: 3]
* **OS Name & Version:** macOS Ventura 13.7.1[cite: 3]
* **Hardware Model:** 2017 MacBook Pro 13-inch[cite: 3]

### Appendix B: Tools Used
* Kali Linux[cite: 3]
* Zenmap / Nmap[cite: 3]
* Tenable Nessus[cite: 3]
* Metasploit Framework[cite: 3]
* PuTTY[cite: 3]
* Remote Lab Workstation[cite: 3]
