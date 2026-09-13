# Penetration Testing Lab: Exploiting Vulnerable Workstation (vsftpd RCE)

## Overview
* Target IP: 172.30.0.55[cite: 3]
* Objective: Conduct network reconnaissance, service enumeration, vulnerability scanning, and exploit execution against a vulnerable Linux target[cite: 3].
* Key Result: Successfully exploited an unauthenticated backdoor in `vsftpd v2.3.4` (CVE-2011-2523), gaining interactive root-level command shell access[cite: 3].

## Technical Details and Artifacts

### Reconnaissance & Port Scanning
* Target IP: 172.30.0.55[cite: 3]
* Total Closed Ports: 977[cite: 3]
* Identified Open Ports and Services:[cite: 3]
  * Port 21/tcp: ftp (vsftpd 2.3.4)[cite: 3]
  * Port 22/tcp: ssh (OpenSSH 4.7p1 Debian 1ubuntu1 protocol 2.0)[cite: 3]
  * Port 23/tcp: telnet (Linux telnetd)[cite: 3]
  * Port 25/tcp: smtp (Postfix smtpd)[cite: 3]
  * Port 53/tcp: domain (ISC BIND 9.4.2)[cite: 3]
  * Port 80/tcp: http (Apache httpd 2.2.8 (Ubuntu) DAV/2)[cite: 3]
  * Port 111/tcp: rpcbind (2 RPC #100000)[cite: 3]
  * Port 139/tcp: netbios-ssn (Samba smbd 3.X-4.X workgroup: WORKGROUP)[cite: 3]
  * Port 445/tcp: netbios-ssn (Samba smbd 3.X-4.X workgroup: WORKGROUP)[cite: 3]
  * Port 512/tcp: exec (netkit-rsh rexecd)[cite: 3]
  * Port 513/tcp: login[cite: 3]
  * Port 514/tcp: shell (Netkit rshd)[cite: 3]
  * Port 1099/tcp: java-rmi (Java RMI Registry)[cite: 3]
  * Port 1524/tcp: shell (Metasploitable root shell)[cite: 3]
  * Port 2049/tcp: nfs (2-4 RPC #100003)[cite: 3]
  * Port 2121/tcp: ftp (ProFTPD 1.3.1)[cite: 3]
  * Port 3306/tcp: mysql (MySQL 5.0.51a-3ubuntu5)[cite: 3]
  * Port 5432/tcp: postgresql (PostgreSQL DB 8.3.0-8.3.7)[cite: 3]
  * Port 5900/tcp: vnc (VNC protocol 3.3)[cite: 3]
  * Port 6000/tcp: X11 (access denied)[cite: 3]
  * Port 6667/tcp: irc (UnrealIRCd)[cite: 3]
  * Port 8009/tcp: ajp13 (Apache Jserv Protocol v1.3)[cite: 3]
  * Port 8180/tcp: http (Apache Tomcat/Coyote JSP engine 1.1)[cite: 3]

### Vulnerability Scan Metrics
* Scan Severity Summary: 9 Critical, 7 High, 18 Medium, 5 Low, 67 Info[cite: 3]
* Key Critical Findings:
  * Plugin 134862 (CVSS 9.8): Apache Tomcat AJP Connector Request Injection (Ghostcat)[cite: 3]
  * Plugin 1088 (CVSS 9.8): Bind Shell Backdoor Detection[cite: 3]
  * Plugin 33850 (CVSS 10.0): Unix Operating System Unsupported Version Detection[cite: 3]
  * Plugin 34460 (CVSS 10.0): Unsupported Web Server Detection[cite: 3]
  * Plugin 32314 (CVSS 10.0): Debian OpenSSH/OpenSSL Package Random Number Generator Weakness[cite: 3]
  * Plugin 32321 (CVSS 10.0): Debian OpenSSH/OpenSSL Package Random Number Generator Weakness (SSL check)[cite: 3]
  * Plugin 11356 (CVSS 10.0): NFS Exported Share Information Disclosure[cite: 3]
  * Plugin 61708 (CVSS 10.0): VNC Server 'password' Password[cite: 3]

### Exploitation Output
* Session Triggered: `[*] Command shell session 1 opened (172.30.0.7:43098 -> 172.30.0.55:6200)`[cite: 3]
* Active Identity: `root`[cite: 3]

### Host Configuration Verification
* Interface: `eth0` | IP Address: `172.30.0.55` | Netmask: `255.255.255.0` | HWaddr: `00:50:56:bd:19:a9`[cite: 3]
* Interface: `lo` | IP Address: `127.0.0.1` | Netmask: `255.0.0.0`[cite: 3]

### Firewall Audit Output
* Chain INPUT: Policy ACCEPT (No restrictive rules)[cite: 3]
* Chain FORWARD: Policy ACCEPT (No restrictive rules)[cite: 3]
* Chain OUTPUT: Policy ACCEPT (No restrictive rules)[cite: 3]

### Vulnerability Plugin Metadata
* Plugin ID: 52703[cite: 3]
* Severity: Info[cite: 3]
* Family: FTP[cite: 3]
* Details: vsftpd Detection (Published 3/17/2011, Updated 11/22/2019)[cite: 3]
* Synopsis: An FTP server is listening on the remote port[cite: 3].
* Description: The remote host is running vsftpd, an FTP server for UNIX-like systems written in C[cite: 3].

## Investigation & Attack Methodology

### 1. Network Reconnaissance & Port Scanning
* Executed a ping sweep using Zenmap across subnet `172.30.0.0/24` to map active network hosts[cite: 3].
* Performed an "Intense Scan" against target `172.30.0.55`[cite: 3]. Identified open services, including vsftpd version 2.3.4 running on port 21/tcp[cite: 3].

### 2. Vulnerability Assessment
* Scanned target `172.30.0.55` using Tenable Nessus[cite: 3].
* Flagged legacy software vulnerabilities, specifically vsftpd 2.3.4 (Nessus Plugin 52703)[cite: 3]. 
* Identified that vsftpd 2.3.4 contains a malicious backdoor triggered by sending a username containing `:)`, forcing the daemon to spawn a listening root shell on TCP port 6200[cite: 3].

### 3. Exploitation & Access Verification
* Loaded Metasploit module `exploit/unix/ftp/vsftpd_234_backdoor`[cite: 3].
* Executed payload against target `172.30.0.55:21`[cite: 3].
* Established an active command shell session (`172.30.0.7:43098 -> 172.30.0.55:6200`)[cite: 3].
* Issued `whoami` and `ifconfig` commands to confirm unauthenticated root access on target `172.30.0.55`[cite: 3].

### 4. Post-Exploitation Inspection
* Issued `iptables --list` to inspect host-based packet filtering controls[cite: 3].
* Confirmed default `ACCEPT` policies across all chains without restrictive firewall rules, enabling unhindered inbound/outbound attacker traffic[cite: 3].

## Key Security Takeaways
* Exploit Impact: Operating legacy software such as `vsftpd 2.3.4` introduces critical network risks, allowing attackers to achieve instant root-level compromise without authentication[cite: 3].
* Defensive Controls: Network segmentation and strict host-based firewalls are mandatory; default open `iptables` configurations fail to contain lateral movement once initial access is gained[cite: 3].
* Lifecycle Management: Automated vulnerability scanning paired with proactive patch management policies are essential to remediate known CVEs prior to exploitation[cite: 3].

## Tools Used
* Kali Linux[cite: 3]
* Metasploit Framework[cite: 3]
* Nmap / Zenmap[cite: 3]
* Tenable Nessus[cite: 3]
