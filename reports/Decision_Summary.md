# Triage Decision Summary

This document outlines the decision rules used across the triage batch. It sets clear boundaries for False Positives (closing), escalations, and handling missing data.

## 1. False Positive (FP) Heuristics
*Conditions required to safely close an alert.*

### FP Heuristic #1 — Verified Benign Operation
* **Rule:** If a benign explanation exists AND evidence supports it AND signals match, then FP likely.
* **Core Idea:** The anomaly is real, but the operation is validated.
* **Requires:** Multiple sources matching (e.g., baseline, change request, VPN, EDR).
* **Failure Mode:** Closing the alert based on a weak or single assumption.

### FP Heuristic #2 — Expected System Behavior
* **Rule:** If there is an anomaly BUT the current business context explains it, then FP likely.
* **Core Idea:** The behavior is normal for that specific environment.
* **Requires:** Connecting the event to known daily operations (backups, VPN routing, admin scripts).
* **Failure Mode:** Escalating a normal system behavior because of missing environment knowledge.

### FP Heuristic #3 — Baseline Consistency
* **Rule:** If the activity matches the historical baseline AND there are no new malicious indicators, then FP likely.
* **Core Idea:** The event looks anomalous but is statistically normal for this system.
* **Requires:** Consistent historical data (same time, same host, same user).
* **Failure Mode:** Focusing on a single alert and ignoring the historical baseline.

---

## 2. Escalation Rules
*Conditions required to contain or investigate.*

### Escalation Rule #1 — Post-Compromise Activity
* **Rule:** Escalate if privilege escalation OR persistence OR lateral movement is detected.
* **Rationale:** A chain of malicious behavior is a higher risk than a single alert.

### Escalation Rule #2 — Malicious Process Chain
* **Rule:** Escalate if there is a suspicious process chain (e.g., Office -> cmd -> schtasks OR browser -> PowerShell) AND an outbound external connection exists AND no known benign explanation exists.
* **Rationale:** Suspicious parent-child processes combined with external network connections confirm malicious intent.

### Escalation Rule #3 — Identity Compromise
* **Rule:** Escalate if there is an authentication anomaly (impossible travel, MFA fatigue, brute force) AND it comes from an untrusted source (foreign IP, unknown device) AND post-login activity is suspicious.
* **Rationale:** Auth anomalies alone create noise. Correlating them with device/location and post-login behavior confirms the compromise.

---

## 3. Unknown-Handling Rules
*Handling decisions with missing data.*

### Unknown Rule #1 — Unknown Process Origin
* **Rule:** If the parent process or user is unknown AND there is no post-execution activity, action is MONITOR.
* **Core Idea:** Without knowing the source, you cannot safely close or escalate. 

### Unknown Rule #2 — Unverified Benign Hypothesis
* **Rule:** If there is an anomaly AND the benign explanation is possible but unverified, action is MONITOR.
* **Core Idea:** The hypothesis exists, but we do not have enough logs to prove or disprove it. Wait for post-execution signals.