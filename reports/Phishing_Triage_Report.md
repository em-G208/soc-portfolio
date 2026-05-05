# PHISHING TRIAGE REPORT

---

## MANAGER SUMMARY

- Credential phishing attempt detected via user-reported suspicious email.

- **Decision:** Escalate. Credential submission could not be confirmed; user interaction with a credential harvesting URL introduces potential credential or session exposure.

- **Actions:** Forced credential reset and session revocation initiated; monitoring active.

- Triage completed in < 25 minutes.

---

## WHO

- Michael B. from HR-01 reported a suspicious email. The sender identity is unknown; attribution is not possible at triage level.

## WHAT

- The email contained a spoofed sender header. Upon clicking the link, the user was redirected to a fake URL mimicking the company's login page. Michael B. clicked the link but it is uncertain whether credentials were submitted. Even without confirmed submission, session token exposure or browser auto-fill interaction cannot be ignored.

## WHEN

- At 9:05 AM (US/EST) email was sent to Michael B. and at 10:27 AM, Michael reported the suspicious email. Triage started at 10:33 AM.

## WHERE

- Michael B.'s work computer (Windows) is affected. Microsoft account and VPN access may be at risk pending credential verification. If SSO is in place, this exposure may extend beyond a single account, potentially allowing access to downstream SaaS systems and internal tools. Lateral movement is not observed at this stage.

## WHY

- The purpose of this attack is harvesting credentials of the authorized personnel. HR personnel typically have broad access to sensitive systems and employee data, making them a primary target for credential theft.

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
- Should check the email gateway logs to see if any other phishing email was sent to other users.
- Should check auth logs for any successful or failed login attempts after the click timestamp; specifically look for impossible travel, new device registration, failed login burst, or MFA bypass attempts. 
- Proxy and DNS logs should also be reviewed for access to the phishing domain.

### Prevention:
- Must block all spoofed sender domains in the email gateway.
- Must add the fake URL to threat intel feed
- Must create a detection rule for similar patterns.

- - - 
  
## RISK DISCLOSURE

### Decision Boundary:
* Spoofed header email with a fake credential harvesting URL was enough to escalate. 
- Credential submission data was missing; therefore exposure was assumed and escalation was warranted.

### Failure Modes:
- This decision might be wrong if auth logs show no successful or failed login attempts after the click timestamp and email gateway confirms no other delivery. 
- In that case, contain and close would be sufficient.

### Escalation Trigger:
- If any post-click authentication attempt is detected (successful or failed), the incident transitions from potential exposure to confirmed credential compromise. Any confirmed additional delivery to other users would further validate this decision.