# SOC Incident Report - Introduction to Phishing Scenario

## Incident Overview
* Activity Date: November 20, 2025 (20:02:00 UTC - 20:05:55 UTC)
* Environment: TryHackMe SOC Simulator (Splunk, Email & Network Logs)
* Objective: Triage inbound phishing alerts, analyze email metadata, correlate firewall telemetry, and establish true/false positive classifications.
* Outcome: Analyzed 5 alerts. Identified 3 True Positive phishing attempts and 2 False Positive internal business communications.
* Impact: No compromised hosts. Perimeter firewall rules successfully blocked all outbound network connections resulting from link clicks.

## Technical Details and Artifacts

### Incident 1: Amazon Delivery Phish (Alerts #8815 & #8816)
* Verdict: True Positive (Contained)
* Type: Brand Impersonation / Malicious Shortener
* Sender: urgents@amazon.biz
* Recipient: h.harris@thetrydaily.thm
* Source Host: 10.20.2.17 (Port 34257)
* Destination IP: 67.199.248.11 (Port 80)
* URL: http://bit.ly/3sHkX3da12340
* Action Taken: Blocked by firewall rule "Blocked Websites"

### Incident 2: Microsoft Account Impersonation (Alert #8817)
* Verdict: True Positive
* Type: Typo-Squatted Domain / Credential Harvesting
* Sender: no-reply@m1crosoftsupport.co
* Recipient: c.allen@thetrydaily.thm
* URL: https://m1crosoftsupport.co/login
* Reported Attacker IP: 102.89.222.143 (Lagos, Nigeria)

### Incident 3: HR Onboarding Email (Alerts #8814 & #8818)
* Verdict: False Positive
* Sender: onboarding@hrconnex.thm
* Recipient: j.garcia@thetrydaily.thm
* Target URL: https://hrconnex.thm/onboarding/15400654060/j.garcia

## Investigation Steps

### 1. Verification of HR Emails (#8814 & #8818)
Initial triage flagged an inbound onboarding link from hrconnex.thm sent to j.garcia. Cross-referencing the domain in SIEM logs revealed an IT support ticket from internal HR confirming that hrconnex.thm is an authorized third-party platform for new hire paperwork. This activity was marked as a False Positive.

### 2. Analysis of Amazon Delivery Phish (#8815 & #8816)
User h.harris received a package delivery warning from amazon.biz containing a shortened Bitly link. SIEM logs showed the user clicked the link at 20:04:23 UTC, initiating an outbound connection from 10.20.2.17 to 67.199.248.11 on port 80. Egress firewall logs confirmed the connection attempt was blocked by policy before data transfer occurred.

### 3. Analysis of Microsoft Alert (#8817)
User c.allen received a fraudulent notice regarding unauthorized account activity in Nigeria, directing them to log in at m1crosoftsupport.co. The domain uses a typo-squatted string (m1crosoft) to harvest credentials. Marked as True Positive.

## Classification and Escalation

* True Positive Justification: Incidents #8815, #8816, and #8817 used unauthorized lookalike domains, obfuscated URLs, and urgency-based social engineering to direct users to unverified infrastructure.
* Escalation Decision:
  * Alert #8816: Closed. Firewall logs verify 0 bytes were transferred and the outbound TCP session was dropped. No host containment required.
  * Alert #8817: Escalated to L2. Requested global blocklisting for m1crosoftsupport.co across mail and web gateways.

## Recommended Actions

1. Egress Block Verification: Keep 67.199.248.11 blocked on edge firewalls.
2. Gateway Filtering: Add m1crosoftsupport.co and amazon.biz to mail gateway blocklists.
3. Mailbox Cleanup: Purge remaining instances of these messages from employee inboxes.
4. User Training: Follow up with user h.harris regarding URL shorteners in external emails.
5. Rule Tuning: Whitelist domain hrconnex.thm for legitimate HR workflows to reduce false positive alerts.
