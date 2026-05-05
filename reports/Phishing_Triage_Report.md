# PHISHING TRIAGE REPORT

---

## MANAGER SUMMARY

- **Alert:** Credential phishing attempt detected via user-reported suspicious email.

- **Decision:** Escalate.
  
- **Reason:** User clicked the link; credential exposure unknown. Unknown + high-impact account (HR + possible SSO) = unacceptable false negative risk

- **Actions:** Containment applied. Compromise not confirmed. Monitoring active.

- Triage completed in < 25 minutes.

---

## WHO

- Michael B. from HR-01 reported a suspicious email. 
- The sender identity is unknown; attribution is not possible at triage level.

## WHAT

- The email contained a spoofed sender header. 
- Upon clicking the link, the user was redirected to a fake URL mimicking the company's login page. 
- Michael B. clicked the link but it is uncertain whether credentials were submitted.
- Even without confirmed submission, potential session or credential exposure cannot be ignored. 
- This creates an unknown exposure state, not a confirmed compromise.

## WHEN

- At 9:05 AM (US/EST) email was sent to Michael B. and at 10:27 AM, Michael reported the suspicious email. Triage started at 10:33 AM.

## WHERE

- Michael B.'s work computer (Windows) is affected. 
- Microsoft account and VPN access are potentially exposed pending credential verification.  
- Lateral movement is not observed at this stage.

## WHY

- Escalation is based on potential impact, not confirmed compromise. 
- HR accounts typically have access to internal systems, employee data, and multiple SaaS platforms.
- If SSO is enabled, a single credential compromise may spread across systems.
- This increases the blast radius beyond a single account, justifying escalation even without confirmed credential submission.
- Decision prioritizes false negative avoidance over false positive cost.

---

## KILL CHAIN MAPPING

- **Reconnaissance:** unknown, not observed at triage level.

- **Weaponization:** spoofed header and fake URL page.  
  **MITRE:** No direct MITRE mapping (pre-attack phase)

- **Delivery:** phishing email.  
  **MITRE:** T1566.002 - Spearphishing Link

- **Exploitation:** redirect to fake login page mimicking company portal (social engineering); credential submission unconfirmed - input capture suspected, not observed. 
  **MITRE:** T1056.003 - Input Capture: Web Portal Capture (suspected)

- **Installation:** not applicable, no malware execution observed.

- **Command & Control:** not applicable, no C2 execution observed.

- **Actions on Objective:** credential harvest for unauthorized access.  
  **MITRE:** T1078 - Valid Accounts (intended - not yet observed)

---

## REMEDIATION

### Containment:
1. Immediate password reset. (user + enforced)  
2. Revoke all active sessions and authentication tokens.
3. Force MFA re-authentication.  
4. Temporarily disable the affected account until verification is complete.

### Investigation:
- Review email gateway logs for additional recipients.
- Review authentication logs post-click (failed/success, new device, impossible travel, MFA anomalies)
- Review proxy/DNS logs for phishing domain access
  
### Prevention:
- Block all spoofed sender domains in the email gateway.
- Add the fake URL to threat intel feed
- Create a detection rule for similar patterns.

- - - 
  
## RISK DISCLOSURE

### Decision Boundary:
-  Unknown credential exposure combined with a credential harvesting URL was sufficient to escalate.

### Failure Modes:
- If no authentication activity is observed post-click and no additional recipients are identified, the incident may be downgraded to contained exposure.

### Escalation Trigger:
- If any post-click authentication attempt is detected (successful or failed), the incident transitions from potential exposure to confirmed credential compromise. Any confirmed additional delivery to other users would further validate this decision.
