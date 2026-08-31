# Zero-Trust Network Segmentation: Risk Assessment, Risk Register, and Access Control Policy

Version: 1.0 (2026-07-01), public reference edition
Frameworks: ISO/IEC 27001:2022 Annex A · NIST CSF 2.0 · Zero Trust (NIST SP 800-207 principles)

> Reference architecture based on a real implementation. This is a representative Zero-Trust segmentation case study for a small self-managed environment. It is deliberately generic: all addresses, identifiers, device and product names, provider names, and topology specifics have been removed or abstracted. It documents a design pattern and its risk decisions, not any live production network.

---

## 1. Scope and system description

The environment is a self-managed network and virtualization lab used for security research and self-hosted services. It began as a single flat subnet behind one edge firewall: no segmentation, an active IP-address collision, and IoT devices, personal laptops, a hypervisor, and lab virtual machines all sharing one broadcast domain. In that starting state, one compromised IoT device had lateral reach to everything.

The objective of the project was to re-architect the network into a Zero-Trust, default-deny topology with tagged VLAN segmentation and a provably air-gapped lab segment, then verify each control empirically before and after cutover.

Components in scope (sanitized):

| Layer | Component | Role |
|---|---|---|
| Edge | Firewall with built-in IDS/IPS | Inter-segment policy enforcement, default-deny, intrusion alarms |
| Distribution | 802.1Q managed switch | VLAN trunking and per-port assignment |
| Compute | Type-1 hypervisor | Hosts lab VMs; management plane |
| Lab VMs | Offensive-research VM, sandbox firewall VM, development VM, experimental AI-agent VM | Isolated lab workloads |
| Cloud | VPS running an agentic AI environment | Off-prem workload, separate trust boundary |
| Endpoints | Personal, work, and IoT devices | Segmented by trust level |

Network segments (VLANs):

| Segment | Trust level | Purpose |
|---|---|---|
| Home | Trusted | Personal and work endpoints |
| Lab + hypervisor management | Restricted | Hypervisor and lab VMs; air-gapped from all other segments |
| IoT | Untrusted | Untrusted devices; no inbound, restricted egress |
| Isolated | Untrusted / isolated | A sensitive single-purpose host; isolated with narrow, port-restricted egress |
| Guest | Untrusted | Visitor devices; VPN-forced egress, no LAN reach |

Within the trusted segment, further micro-segmentation groups (identity-based) separate personal, single-purpose, and work devices from one another.

---

## 2. Asset inventory

| ID | Asset | Segment | Confidentiality | Integrity | Availability |
|---|---|---|---|---|---|
| A-01 | Hypervisor host and management interface | Lab/Mgmt | High | High | High |
| A-02 | Offensive-research VM (holds tools, findings) | Lab | Medium | High | Medium |
| A-03 | Sandbox firewall VM (malware-analysis air-gap) | Lab | Medium | High | Medium |
| A-04 | Development VM (source, credentials, AI coding agents) | Lab | High | High | Medium |
| A-05 | Experimental AI-agent VM | Lab | Medium | Medium | Low |
| A-06 | Cloud VPS agentic AI environment | Cloud | High | High | Medium |
| A-07 | Personal endpoints | Home | High | Medium | Medium |
| A-08 | Work endpoints | Home | High | High | Medium |
| A-09 | IoT devices | IoT | Low | Medium | Medium |
| A-10 | Isolated single-purpose host | Isolated | Medium | High | Medium |
| A-11 | Edge firewall configuration and logs | Edge | High | High | High |

---

## 3. Threat catalogue

Primary threat scenarios considered against the flat starting state and the target architecture:

- T1: Compromised IoT device (vendor CVE, weak firmware) pivots laterally to personal or lab assets.
- T2: Malware in the offensive-research/sandbox lab escapes to the personal network.
- T3: Unauthorized access to the hypervisor management plane grants control of all VMs.
- T4: Guest or untrusted device eavesdrops on or reaches internal services.
- T5: IoT device exfiltrates data or phones home to untrusted endpoints.
- T6: Isolated single-purpose host is exploited through an exposed service.
- T7: Address collision or rogue/unknown device causes reachability loss or an unmonitored foothold.
- T8: Weak or downgraded wireless authentication enables network access.
- T9: Loss of edge configuration (device failure) causes prolonged outage with no recovery path.
- T10: Credential or key compromise on the cloud VPS exposes the agentic AI workload.

---

## 4. Risk register

Scoring: Likelihood and Impact each rated Low / Medium / High. Risk = Likelihood x Impact (L+L=Low; one High=Medium; H+H=High). Inherent = before the controls below. Residual = after implemented controls. Owner is the environment administrator (single-operator environment).

| ID | Asset | Risk scenario | Likelihood (inh.) | Impact | Inherent | Existing controls (implemented) | ISO 27001:2022 / NIST CSF 2.0 | Residual | Treatment |
|---|---|---|---|---|---|---|---|---|---|
| R-01 | A-09 → all | IoT device CVE gives lateral movement across a flat LAN (T1) | High | High | **High** | VLAN segmentation; default-deny inter-VLAN (baseline rule); AP-level client isolation on IoT SSID | A.8.22, A.8.20 / PR.IR-01, PR.AA-05 | **Low** | Accept; re-verify each new IoT onboarding |
| R-02 | A-02/03 → A-07 | Lab malware escapes to personal network (T2) | Medium | High | **High** | Dedicated lab VLAN air-gapped from all segments; empirical escape test (external ping succeeds, internal ping must fail) run pre and post cutover; sandbox firewall VM retained as cold-standby | A.8.22, A.8.7, A.8.31 / PR.IR-01, DE.CM-09 | **Low** | Accept; mandatory escape-test re-run after any lab re-IP |
| R-03 | A-01 | Unauthorized access to hypervisor management plane (T3) | Medium | High | **High** | Hypervisor on the trusted segment at a reserved address, administered from a single designated laptop over the management UI and SSH; management access limited to trusted-segment devices. Cross-segment single-device pinhole (two-port) is designed but deferred pending the management-plane re-IP | A.8.2, A.5.15, A.8.20 / PR.AA-01, PR.AA-05 | **Medium** | Reduce: complete host re-IP to the management VLAN to enforce the single-device, two-port pinhole; then per-user RADIUS in Phase 2 |
| R-04 | A-07/08 | Guest device reaches internal services or real WAN (T4) | Medium | Medium | **Medium** | Dedicated guest VLAN; block-all-LANs rule; egress forced through VPN tunnel; no service discovery relay to guest | A.8.22, A.8.23, A.5.14 / PR.IR-01, PR.AA-05 | **Low** | Accept; rotate guest secret quarterly and after each visit |
| R-05 | A-09 | IoT device data exfiltration / call-home (T5) | High | Medium | **High** | IoT-to-Internet egress kill-switch with a controlled firmware-update window; per-device egress allow-lists; region/endpoint block lists | A.8.23, A.8.20, A.5.7 / DE.CM-01, PR.IR-01 | **Low** | Accept; review egress baselines periodically |
| R-06 | A-10 | Isolated host exploited via an exposed service (T6) | Medium | Medium | **Medium** | Host on its own isolated VLAN; blocked from all other segments; Internet egress restricted to the specific protocol ports required | A.8.22, A.8.20 / PR.IR-01, PR.PS-01 | **Low** | Accept |
| R-07 | all | IP-address collision or rogue/unknown device (T7) | Medium | Medium | **Medium** | Collision resolved via reservations; unknown-device alarms; reactive quarantine posture with monitoring | A.8.16, A.5.9 / DE.CM-01, ID.AM-01 | **Low** | Monitor; weekly review |
| R-08 | A-07/08 | Weak or downgraded wireless authentication (T8) | Medium | High | **High** | WPA3-SAE with PMF required on trusted SSIDs; legacy IoT SSID locked to WPA2-AES with compensating isolation (documented exception, no mixed mode to avoid downgrade/KRACK vectors) | A.8.5, A.8.24 / PR.AA-01, PR.AA-03 | **Low** | Accept; re-evaluate IoT exception as device firmware matures |
| R-09 | A-11 | Loss of edge configuration causes prolonged outage (T9) | Low | High | **Medium** | Configuration backup retained; previous edge device kept as labeled cold-standby with a documented ~10-minute rollback path | A.8.13, A.5.29 / RC.RP-01, PR.IR-04 | **Low** | Accept |
| R-10 | A-06 | Credential/key compromise on cloud VPS (T10) | Medium | High | **High** | Separate trust boundary from on-prem; distinct credentials; workload isolated from the home segments | A.8.2, A.8.24, A.5.23 / PR.AA-01, PR.DS-01 | **Medium** | Reduce: add key rotation + MFA review (roadmap) |
| R-11 | all | No continuous monitoring of inter-segment activity (T1-T7) | Medium | Medium | **Medium** | Edge IDS/Active Protect per VLAN; alarm triage playbook (P0-P3) with a documented first-hour response and rollback decision tree; firewall set as DNS resolver for query visibility | A.8.16, A.8.15, A.5.7 / DE.CM-01, DE.AE-02, RS.MA-01 | **Low** | Accept; formalize into a future SIEM project |

Residual risk summary: 9 of 11 risks reduced to Low. Two remain Medium with defined treatments: R-03 (management plane, pending the host re-IP that enforces the single-device pinhole) and R-10 (cloud VPS, pending key rotation and MFA review).

---

## 5. Control narrative (by function)

**Segment and isolate (ISO A.8.22 / CSF PR.IR).** The network moved from one flat subnet to five VLANs with a default-deny posture between them. Every inter-segment flow is explicitly denied unless a specific pinhole permits it. The lab segment is air-gapped and the isolation is verified by test, not assumed.

**Least privilege (ISO A.8.2, A.5.15 / CSF PR.AA).** Administrative reach to the hypervisor is restricted to a single designated laptop; narrowing that to a two-port, cross-segment pinhole is designed and pending the management-plane re-IP. Guests and IoT devices get the minimum egress they need and nothing lateral. Work devices are forced through a VPN tunnel and separated from personal devices by identity group.

**Authenticate strongly (ISO A.8.5, A.8.24 / CSF PR.AA).** Trusted wireless uses WPA3-SAE with protected management frames. The one segment that cannot support it (legacy IoT) is a documented exception with compensating isolation, deliberately avoiding mixed mode to prevent downgrade attacks.

**Monitor and respond (ISO A.8.16 / CSF DE, RS).** The edge runs IDS with per-segment tuning, a written alarm-triage playbook classifies events P0 to P3, and the firewall is the DNS resolver so every lab VM's queries are visible.

**Recover (ISO A.8.13 / CSF RC).** Configuration is backed up and the prior edge device stays cabled as a cold standby with a rehearsed rollback.

---

## 6. Access Control Policy

**Purpose.** Define how access between network segments, to administrative interfaces, and onto the network is granted, restricted, and reviewed, on Zero-Trust principles.

**Scope.** All network segments, wireless networks, virtualization hosts, virtual machines, and connected devices in the environment described in Section 1.

**Policy statements.**

1. **Default deny.** All traffic between segments is denied unless an explicit, documented rule permits it. New access requires a defined source, destination, port, and business reason. (ISO A.5.15, A.8.20)
2. **Segmentation by trust.** Devices are placed on the segment matching their trust level: trusted endpoints, restricted lab/management, untrusted IoT, isolated, and untrusted guest. Cross-segment reach is the exception, never the default. (ISO A.8.22)
3. **Least-privilege administration.** The hypervisor management plane is reachable only from a single designated administrative device, restricted to the management UI and SSH ports. General endpoints cannot reach it. (ISO A.8.2)
4. **Authentication standards.** Trusted wireless requires WPA3-SAE with protected management frames. Any exception (for example legacy IoT that cannot support WPA3) must be documented, justified, and paired with compensating controls such as device isolation and egress restriction. (ISO A.8.5)
5. **Guest and untrusted access.** Guest devices are confined to the guest segment, blocked from all internal segments, and egress only through a VPN tunnel. Guest credentials are rotated quarterly and after any event where they were shared. (ISO A.8.22, A.5.14)
6. **IoT and untrusted devices.** Untrusted devices are denied inbound access, restricted to explicit egress allow-lists, and isolated from one another where supported. (ISO A.8.20, A.8.23)
7. **Monitoring and review.** Inter-segment activity is monitored by the edge IDS; alarms are triaged by the P0-P3 playbook. Access rules and connected-device inventory are reviewed weekly; the rule set is reviewed after any material change. (ISO A.8.16, A.5.18)
8. **Change and exception management.** Rule changes are documented; exceptions are time-bound where possible and recorded with their compensating controls. (ISO A.8.32)
9. **Roadmap to identity-based access.** PSK-based access will migrate to per-user credentials via WPA3-Enterprise and RADIUS, upgrading administrative and network access from device-IP identity to authenticated user identity. (ISO A.5.16 / CSF PR.AA-01)

**Roles.** Single operator acting as network owner, administrator, and reviewer. In an enterprise mapping these separate into network engineering, security operations, and an access-review approver.

**Review cadence.** Weekly device and alarm review; policy reviewed on material change or at least annually.

---

## 7. Residual risk and roadmap

Nine of eleven risks are reduced to Low. The open items:

- **R-03 (management plane):** complete the hypervisor re-IP to the management VLAN to enforce the single-device, two-port pinhole, moving residual from Medium to Low.
- **R-10 (cloud VPS):** implement credential/key rotation and review MFA on the agentic AI workload. Target: next 30 days.
- **Identity upgrade (Phase 2):** WPA3-Enterprise + RADIUS, per-user credentials, and access rules keyed to user identity rather than device IP.
- **Monitoring maturity:** promote the current IDS-plus-playbook approach into a dedicated SIEM project for correlated, retained logging.

---

## 8. Summary

A flat, single-subnet network, where any compromised device could reach everything, was re-architected into a Zero-Trust, default-deny design: five segmented VLANs, an air-gapped and escape-tested lab, a single-admin management plane (pinhole hardening in progress), VPN-forced guest egress, and IDS-based monitoring with a written alarm-triage playbook. It is documented as a risk assessment and register mapped to ISO 27001:2022 Annex A and NIST CSF 2.0, with a governing access control policy. The work took the two highest-rated risks (IoT lateral movement, lab-to-personal malware escape) from High to Low and reduced management-plane exposure, with two residual items openly tracked for treatment.
