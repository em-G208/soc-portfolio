# ICMP Tunneling - Detection and Escalation Decision Logic

- - -
  
## Manager Summary

- **Alert:** ICMP payload used for potential data exfiltration via encoded echo request  
- **Decision:** No escalation on single-signal anomaly; escalate only if correlation confirms exfiltration risk
- **Risk:** Single-signal anomalies produce high false positives due to allowlisted protocol behavior  
- **Scope:** Lab-contained activity (loopback), no external traffic generated

---

## Threat Context

ICMP is a diagnostic protocol operating at Layer 3. It uses no ports and is allowlisted by default in most network environments. Standard use: network health checks via `ping`.

Threat actors embed encoded data inside ICMP echo request payloads. No custom malware is required. The attack uses tools native to the operating system, classified as Living off the Land (LotL). It blends into normal administrative traffic.

**Why standard defenses fail:**

- ICMP has no port number, port-based firewall rules are irrelevant
- DPI on ICMP is rarely enforced due to CPU overhead
- The protocol is considered low-risk by default; payloads are not inspected

**MITRE ATT&CK:**

- Tactic: Exfiltration
- Technique: T1048.003 - Exfiltration Over Alternative Protocol: Non-Application Layer Protocol
- Supporting: T1095 - Non-Application Layer Protocol

---

## Lab

### Environment

|Component| Details                         |
| --------------- | ------------------------------- |
|Hypervisor| VirtualBox                      |
|OS| Ubuntu Linux                    |
|Network| Loopback (127.0.0.1) - isolated |
|Packet Analyzer| TShark                          |
|Attack Vector| Native ping utility (LotL)      |

Loopback interface was chosen intentionally. Using an external target would generate real network traffic with a real payload. The technique is demonstrated; the boundary is contained.

### Execution

Target string: `EXFILTRATED`  
Hex encoding: `4558464c4c545241544544`

The string is encoded in hexadecimal before injection. ICMP accepts raw hex as payload input.

```bash
ping -c 1 -p 4558464c4c545241544544 127.0.0.1
```

- `-c 1` - send one packet
- `-p 4558464c4c545241544544` - replace standard ICMP padding with this hex pattern
- `127.0.0.1` - loopback target

The Linux kernel fills the 56-byte ICMP payload with the custom hex string, repeating it cyclically. The receiving host echoes the payload back without inspection or modification.

### Evidence

Capture command:

```bash
sudo tshark -i lo -f "icmp" -w /tmp/icmp_tunnel_demo.pcap
```

Analysis command:

```bash
sudo tshark -r /tmp/icmp_tunnel_demo.pcap -Y "icmp" -T fields -e ip.src -e ip.dst -e data
```

Output:

```
127.0.0.1   127.0.0.1   5452415445444558464c4c5452415445444558464c4c5452415445444558464c4c54524154454400
127.0.0.1   127.0.0.1   5452415445444558464c4c5452415445444558464c4c5452415445444558464c4c54524154454400
```

Two packets captured: echo request and echo reply. Payload is identical in both, the destination echoed the data back without modification. Source and destination both `127.0.0.1`; traffic never left the lab.

Decoded payload: cyclic repetition of the 11-byte string filling the 56-byte payload field. The payload travels in plaintext. The only protection the attacker relies on is the absence of DPI.

---

## Detection Logic

Standard ICMP echo request payload is 56 bytes of predictable ASCII padding: `!"#$%&'()*+,-./0123456789...`

Any deviation from this pattern is anomalous.

**TShark - live detection filter:**

```bash
sudo tshark -i eth0 -Y "icmp and data.len > 0" -T fields -e ip.src -e ip.dst -e data
```

**SIEM correlation rule logic:**

|Signal|Threshold|Action|
|---|---|---|
|ICMP payload size > 64 bytes|Single occurrence|Investigate|
|ICMP payload entropy anomaly|Single occurrence|Investigate|
|ICMP request count from single source|> 10 per minute|Alert|
|ICMP traffic to external IP with non-standard payload|Single occurrence|Escalate|

**Limitation - size-based detection:**

Payload fragmentation bypasses size thresholds. An attacker can split data across multiple packets, each within normal size range. Size anomaly alone is a weak rule.

Reliable detection requires correlation: payload size + packet frequency + entropy analysis. A SIEM rule triggering on any single signal will generate noise. The value is in the intersection.

---

## Decision Logic

**Investigate:**

- Single-signal anomaly detected (payload, size, or entropy deviation)

These signals are anomalous but not sufficient for escalation alone. Baseline ICMP behavior must be established first. A single anomalous packet against a noisy baseline is low confidence.

**Escalate:**

- External destination + correlated anomalies (frequency + entropy)

Single-signal triggers investigate. Multi-signal correlation triggers escalate.

**Decision boundary:**

Payload anomaly alone does not justify escalation without external context.

**Unknown handling:**

If payload is anomalous but destination is internal and frequency is within baseline: log, monitor, do not escalate. Flag for correlation with subsequent traffic.

### Triage Context Interpretation

ICMP anomalies are evaluated based on decision risk, not anomaly presence.

Single-signal anomalies (payload, size, or entropy) are treated as low-confidence due to high false positive potential.

Decision shifts only when two conditions are met simultaneously:

- Destination context introduces external exposure risk  
- Signal correlation confirms non-random behavior (frequency + entropy)

Without both conditions, escalation introduces unnecessary noise.

Principle:

Escalation is not triggered by anomaly detection.  
It is triggered when anomaly + context + correlation indicate actionable risk.

---

## Risk Disclosure

**Decision Boundary:**

Cyclic payload offset behavior - payload starting mid-pattern rather than at string origin - was sufficient to invalidate exact string matching as a detection method. Entropy-based analysis was derived from this observation.

**Failure Modes:**

Fragmented payloads bypass size thresholds. Encrypted tunneling over ICMP bypasses entropy analysis. Detection logic in this lab assumes plaintext payload, encrypted variants require a different rule set.

**Lab Limitations:**

C2 receive-side behavior not demonstrated. Real exfiltration requires a listening server logging echo replies, loopback environment confirms send-side mechanics only. External destination behavior is inferred, not observed.

---

## References

**MITRE ATT&CK:**

- [T1048.003 - Exfiltration Over Alternative Protocol: Non-Application Layer Protocol](https://attack.mitre.org/techniques/T1048/003/)
- [T1095 - Non-Application Layer Protocol](https://attack.mitre.org/techniques/T1095/)

**Tools:**

- [TShark](https://www.wireshark.org/docs/man-pages/tshark.html)
- [ping - Linux man page](https://linux.die.net/man/8/ping)

**Lab files:**

- pcap: `icmp_tunneling_demo.pcap` - included in repository