# SOC Analyst Portfolio

Focuses on alert triage, noise reduction, and operational consistency. Applies high-volume yes/no/unknown decision-making principles to security alerts under incomplete data conditions. Remote. US overnight SOC operations coverage from UTC+3.

## 1. Flagship Report
* [Phishing Triage Decision](https://github.com/em-G208/soc-portfolio/blob/main/reports/Phishing_Triage_Report.md)

## 2. Operational Execution
**Alert Triage Batch**
* [Triage Batch Operations](https://github.com/em-G208/soc-portfolio/blob/main/reports/03_Triage_Batch.md)
* Processing of 10 standard security alerts.
* Focuses on matching signals, testing basic hypotheses, and making clear close/monitor/escalate decisions.

**Decision Logic**
* [Decision Summary](https://github.com/em-G208/soc-portfolio/blob/main/reports/04_Decision_Summary.md)
* The baseline operational rules used to identify false positives and trigger escalations.
* Defines how missing data and unknown variables are handled without guessing.

**Core Operating Principles:**
* Escalate immediately when a decision cannot be made safely.
* Treat "unknown" as an active risk factor, not as an absence of evidence.
* Prioritize potential impact (blast radius) over waiting for full confirmation.

## 3. Technical Context
* [ICMP Tunneling - Detection and Escalation Decision Logic](https://github.com/em-G208/soc-portfolio/tree/main/labs/icmp-tunneling)
