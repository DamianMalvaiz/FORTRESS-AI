# FORTRESS-AI

> Financial sector cybersecurity simulation — full-cycle pentest, DFIR, and AI-powered threat analysis over a virtualized enterprise infrastructure.

![Status](https://img.shields.io/badge/status-Phase%201%20Week%203-yellow)
![License](https://img.shields.io/badge/license-MIT-blue)
![Stack](https://img.shields.io/badge/stack-KVM%20%7C%20pfSense%20%7C%20AD%20%7C%20Security%20Onion-green)

---

## Overview

FORTRESS-AI is a year-long independent cybersecurity project that simulates the logical IT architecture of a fictional Mexican regional bank (**BNF — Banco Nacional Ficticio**) and executes professional-grade offensive security operations against it.

The project covers the full security cycle: **build the infrastructure → attack it → detect it → analyze it forensically → document it at consulting level** — with an AI layer integrated into the detection and reporting pipeline.

Built entirely on dedicated hardware using open-source tooling. Virtualized simulation focused on the control, security, and observability plane of a banking infrastructure.

> ⚠️ **Scope clarification:** FORTRESS-AI is a **logical simulation**, not a physical datacenter replica. See [Architecture Reality Gap](docs/03-architecture-reality-gap.md) for the formal scope declaration and the gap between this simulation and a production banking environment.

---

## Network Topology

![FORTRESS-AI Network Topology](docs/diagrams/fortress-ai-topology-v1.svg)

*Logical network architecture — virtualized on KVM/QEMU. 5 segmented zones with differentiated trust levels, interconnected through a central pfSense firewall.*

---

## Documentation

Three foundational documents describe the project architecture and scope:

| # | Document | Description |
|---|---|---|
| 01 | [Project Charter](docs/01-project-charter.md) · [DOCX](docs/01-project-charter.docx) | Scope, objectives, timeline, risks, stakeholders |
| 02 | [Logical Network Design](docs/02-network-design.md) | IP addressing (VLSM), segmentation, zone matrix, firewall rules |
| 03 | [Architecture Reality Gap](docs/03-architecture-reality-gap.md) | Simulation scope vs physical production reference |

---

## Target Company — BNF

| Attribute | Detail |
|-----------|--------|
| Name | Banco Nacional Ficticio S.A. |
| Sector | Mexican regional banking |
| Size | ~800 employees, 50 branches |
| Domain | `fortress.bnf.local` |
| Infrastructure | Hybrid — on-premise (virtualized) + AWS |
| Regulatory context | CNBV, SPEI integration, PCI-DSS baseline |

---

## The 4 Pillars

### 1. Enterprise Infrastructure (Logical Simulation)
Segmented 5-zone network on KVM/QEMU virtualization, with Active Directory (`fortress.bnf.local`), legacy and modern systems coexisting, hybrid cloud extension to AWS.

### 2. Professional Pentest Cycles (PTES)
Three full attack scenarios executed against BNF infrastructure:
- **Scenario 1:** Active Directory compromise (Kerberoasting → lateral movement → DCSync)
- **Scenario 2:** Customer data exfiltration (web app → DB server → data extraction)
- **Scenario 3:** Cloud attack surface (misconfigured AWS → pivot to internal network)

Each scenario produces a professional pentest report with CVSS scores, PoCs, and remediation guidance.

### 3. Digital Forensics & Incident Response (DFIR)
Post-attack forensic analysis of each scenario: attack timeline reconstruction, IoC identification, artifact collection with chain of custody, and evidence-based reporting.

### 4. AI-Powered Analysis
Integrated AI engine built on the Anthropic API:
- SIEM alert classifier (reduces analyst noise by triaging Security Onion alerts)
- Log anomaly detector (identifies attack patterns in Zeek/Wazuh logs)
- Automated report draft generator (converts raw findings into structured reports)

---

## Project Timeline

| Phase | Duration | Focus |
|-------|----------|-------|
| **Phase 1** | Months 1–3 | Infrastructure setup, network segmentation, AWS integration |
| **Phase 2** | Months 4–6 | SOC deployment, SIEM configuration, automation, BI dashboard |
| **Phase 3** | Months 7–10 | Pentest cycles, DFIR analysis, professional reports |
| **Phase 4** | Months 11–12 | AI integration, documentation, portfolio finalization |

**Target certifications during project:** eJPT (Month 6) → CompTIA Security+ (Month 12)

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Hypervisor | KVM/QEMU + libvirt |
| Firewall / Router | pfSense CE 2.7 |
| Offensive platform | Kali Linux 2024 |
| AD / Windows | Windows Server 2019 Evaluation |
| Linux targets | Ubuntu Server 22.04, Metasploitable 3 |
| SOC / SIEM | Security Onion 2.4, Wazuh, Splunk Free |
| Monitoring | Zeek, Suricata, Kibana, ELK Stack |
| Cloud | AWS Free Tier (EC2, S3, IAM, CloudTrail, VPC) |
| Automation | Python, Bash, Ansible |
| BI | Power BI Desktop |
| AI | Anthropic API (Claude) |
| Documentation | Markdown, draw.io |

---

## Repository Structure

```
FORTRESS-AI/
├── docs/
│   ├── diagrams/                       # Architecture diagrams
│   │   ├── fortress-ai-topology-v1.drawio
│   │   ├── fortress-ai-topology-v1.svg
│   │   └── fortress-ai-topology-v1.png
│   ├── 01-project-charter.md           # Charter (markdown)
│   ├── 01-project-charter.docx         # Charter (Word)
│   ├── 02-network-design.md            # Logical network design v2.0
│   └── 03-architecture-reality-gap.md  # Scope declaration & gap analysis
├── LICENSE
└── README.md
```

**Planned future additions (Phase 2+):**

```
├── scripts/
│   ├── automation/         # Infrastructure automation (Ansible, Bash, Python)
│   ├── offensive/          # Offensive tooling and PoCs
│   └── forensics/          # DFIR collection scripts
├── reports/
│   ├── pentest/            # PTES-format pentest reports
│   ├── forensics/          # DFIR reports with chain of custody
│   └── executive/          # Executive summaries
├── evidence/
│   ├── scenario-1/         # AD compromise evidence
│   ├── scenario-2/         # Data exfiltration evidence
│   └── scenario-3/         # Cloud attack evidence
└── ai/                     # AI engine source code
```

---

## Current Status

### Documentation (Phase 1 — Week 2) ✅

| Deliverable | Status | Version |
|---|---|---|
| Project Charter | ✅ Complete | v1.0 |
| Logical Network Design | ✅ Complete | v2.0 |
| Architecture Reality Gap | ✅ Complete | v1.0 |
| Topology Diagram (SVG/PNG/drawio) | ✅ Complete | v1.0 |

### Infrastructure (Phase 1 — Week 3 in progress) 🔄

| Component | Status |
|---|---|
| Server OS (Ubuntu Server 24.04 LTS) | ✅ Deployed |
| KVM/QEMU + libvirt | ✅ Installed and verified |
| `/fortress` storage structure | ✅ Configured |
| KVM storage pool | ✅ Active |
| SSH access (`damian@192.168.101.104`) | ✅ Functional |
| 5 virtual network bridges (KVM) | ⏳ Pending — Week 3 |
| pfSense CE 2.7 VM | ⏳ Pending — Week 3 |
| Active Directory (Windows Server 2019) | ⏳ Pending — Week 4 |
| Target VMs (Kali, Win7, Metasploitable) | ⏳ Pending — Week 5–6 |
| SOC deployment (Security Onion, Wazuh) | ⏳ Pending — Week 7–8 |

---

## Roadmap

### Phase 1 — Infrastructure (Months 1–3)
- [x] Server setup (Ubuntu Server 24.04 + KVM/QEMU)
- [x] Project Charter
- [x] Logical Network Design
- [x] Architecture Reality Gap
- [x] Topology diagram
- [ ] pfSense VM deployment
- [ ] 5 virtual network bridges (KVM)
- [ ] Active Directory with 200+ objects
- [ ] Target VMs (Kali, Win7, Metasploitable 3)
- [ ] AWS Free Tier integration

### Phase 2 — SOC & Automation (Months 4–6)
- [ ] Security Onion deployment
- [ ] Wazuh HIDS with agent rollout
- [ ] Splunk Free with educational license
- [ ] Ansible automation playbooks
- [ ] Power BI dashboards
- [ ] ISO 27001 policy documentation
- [ ] **eJPT certification** 🎯

### Phase 3 — Pentest & Forensics (Months 7–10)
- [ ] Scenario 1: AD Compromise (Kerberoasting → DCSync)
- [ ] Scenario 2: Data Exfiltration (Web → DB extraction)
- [ ] Scenario 3: Cloud Attack (AWS misconfig → internal pivot)
- [ ] DFIR analysis per scenario
- [ ] PTES-format reports with CVSS

### Phase 4 — AI & Delivery (Months 11–12)
- [ ] SIEM alert classifier (Anthropic API)
- [ ] Log anomaly detector
- [ ] Automated report generator
- [ ] Academic paper on methodology
- [ ] Final demo video
- [ ] Executive presentation
- [ ] **CompTIA Security+ certification** 🎯

---

## Academic Context

**Institution:** Universidad Tecnológica del Valle de Toluca (UTVT)  
**Program:** Ingeniería en Redes Inteligentes y Ciberseguridad  
**Modality:** Dual education — concurrent industrial practice at Continental Tire (Santa Fe, CDMX)  
**Expected graduation:** May 2027  

**Subjects covered by this project:**

Advanced Infrastructure Administration · Big Data Technologies · Data Center Infrastructure · IT Security Management · Cloud Computing · Work Planning · Ethical Hacking · Infrastructure Automation I & II · IT Quality Systems · Business Intelligence · Digital Forensics · Enterprise Network Administration · Project Management I & II · Time Management · High Performance Team Leadership · Business Negotiation

---

## Hardware Specifications

| Component | Specification |
|---|---|
| Server | Acer Nitro V15 (dedicated) |
| CPU | AMD Ryzen (24 virtual cores available for KVM) |
| RAM | 32 GB |
| Storage OS | KIOXIA 512 GB SSD |
| OS | Ubuntu Server 24.04 LTS (headless) |
| Hostname | `fortress-server` |
| Host IP | `192.168.101.104` |

---

## Author

**Ángel Damián Malvaiz González**  
Dual student — Ingeniería en Redes Inteligentes y Ciberseguridad  
Continental Tire IT Department — L1/L2 National Support  
Toluca, México

📧 GitHub: [@DamianMalvaiz](https://github.com/DamianMalvaiz)

**Career roadmap:** eJPT → CompTIA Security+ → CRTO → OSCP  
**Target companies post-graduation:** Scitum · IQsec · Totalsec

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

> *"The best way to defend a system is to understand exactly how it gets attacked."*

---

**⚠️ Ethical Disclaimer:** All systems attacked in this project are owned and controlled by the author within an isolated virtualized lab environment. No external systems are targeted. This project is for educational and research purposes only.
