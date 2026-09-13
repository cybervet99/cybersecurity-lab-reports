# Cybersecurity Incident Response & SOC Portfolio

Hands-on Security Operations Center (SOC) investigation write-ups, incident response reports, and threat detection analysis covering SIEM triage, phishing, network forensics, and MITRE ATT&CK techniques.

## About Me
I started my career as a Database Administrator in the U.S. Army at Ft. Carson, managing access controls, system patching, backup integrity, and database infrastructure for ~150 users in a classified environment. During a battalion-wide system migration, I handled security auditing, role-based access control, and log verification across every endpoint. While I may not have used formal SOC terminology at the time, the fundamental work, hardened infrastructure, system hygiene, and proactive monitoring—was identical to core security operations.

Following my military service, I provided IT support at SideArm Sports, consistently resolving 30–40 tickets per week in Jira while supporting an active user base of 75+ employees. I hold a cybersecurity degree from Syracuse University. When evaluating SOC Analyst roles, the core responsibilities, log triage, endpoint defense, threat detection, and incident response, align directly with my hands-on background. I’ve performed the operational foundation for years; now I combine that military and IT experience with formal SOC methodologies.

---

## Technical Competencies
* **SIEM & Log Triage:** Log aggregation, alert triage, query writing (Splunk, Elastic).
* **Incident Response:** Email header analysis, payload verification, firewall log correlation, EDR triage.
* **Framework Alignment:** MITRE ATT&CK mapping (Persistence, Credential Access, Defense Evasion).

---

## Lab Reports & Incident Investigations

| Incident / Lab Name | Threat Vectors | Key Telemetry / Tools | Report Link |
| :--- | :--- | :--- | :--- |
| **Introduction to Phishing** | Brand Impersonation, Bitly Redirects, Typo-Squatting | Egress Firewall Logs, Sandbox, SIEM | [View Incident Report](/thm-introduction-to-phishing/README.md)|
| **Exploiting Vulnerable Workstation** | Unauthenticated Backdoor (vsftpd v2.3.4), EOL Service Exploitation | Zenmap, Tenable Nessus, Metasploit Framework, iptables | [View Lab Report](./penetration-testing-vulnerable-workstation/README.md)
