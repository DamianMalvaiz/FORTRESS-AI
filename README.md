# FORTRESS-AI

> Financial sector cybersecurity simulation — full-cycle pentest, DFIR, and AI-powered threat analysis over a complete enterprise infrastructure.

---

## Overview

FORTRESS-AI is a year-long independent cybersecurity project that simulates the complete IT infrastructure of a fictional Mexican regional bank (**BNF — Banco Nacional Ficticio**) and executes professional-grade offensive security operations against it.

The project covers the full security cycle: **build the infrastructure → attack it → detect it → analyze it forensically → document it at consulting level** — with an AI layer integrated into the detection and reporting pipeline.

Built entirely on dedicated hardware using open-source tooling. No cloud dependency for core operations.

---

## Project Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FORTRESS-AI — BNF Infrastructure                  │
│                                                                       │
│  ┌─────────┐   ┌──────────────┐   ┌─────────────────────────────┐   │
│  │   DMZ   │   │     LAN      │   │          SERVERS            │   │
│  │         │   │              │   │                             │   │
│  │ pfSense │◄─►│ Kali Linux   │   │ Windows Server 2019 (DC)    │   │
│  │ (FW)    │   │ (attacker)   │   │ Windows Server 2019 (File)  │   │
│  └─────────┘   │ Windows 7    │   │ Ubuntu Server (web/db)      │   │
│                │ (legacy)     │   │ Metasploitable 3            │   │
│  ┌─────────┐   └──────────────┘   └─────────────────────────────┘   │
│  │   SOC   │◄──────── monitors all traffic ───────────────────────  │
│  │         │                                                         │
│  │Security │   ┌──────────────┐   ┌─────────────────────────────┐   │
│  │  Onion  │   │  MANAGEMENT  │   │        CLOUD (AWS)          │   │
│  │  Wazuh  │   │  pfSense #2  │   │  EC2 t2.micro + VPN         │   │
│  │  Splunk │   │              │   │  S3 + CloudTrail            │   │
│  └─────────┘   └──────────────┘   └─────────────────────────────┘   │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │                    AI ENGINE                                │     │
│  │   SIEM alert classifier · Anomaly detector · Report gen    │     │
│  └─────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────┘
```

**Host:** Acer Nitro V15 — Ubuntu Server 24.04 LTS — KVM/QEMU  
**VMs:** 10 virtual machines across 5 network zones  
**Cloud:** AWS free tier (hybrid extension)

---

## Target Company — BNF

| Attribute | Detail |
|-----------|--------|
| Name | Banco Nacional Ficticio S.A. |
| Sector | Mexican regional banking |
| Size | ~800 employees, 50 branches |
| Domain | fortress.bnf.local |
| Infrastructure | Hybrid — on-premise + AWS |
| Regulatory context | CNBV, SPEI integration, PCI-DSS baseline |

---

## The 4 Pillars

### 1. Enterprise Infrastructure
Complete replica of a regional bank's IT architecture: segmented network zones, Active Directory with 200+ objects, legacy and modern systems coexisting, hybrid cloud connectivity.

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
| Phase 1 | Months 1–3 | Infrastructure setup, network segmentation, AWS integration |
| Phase 2 | Months 4–6 | SOC deployment, SIEM configuration, automation, BI dashboard |
| Phase 3 | Months 7–10 | Pentest cycles, DFIR analysis, professional reports |
| Phase 4 | Months 11–12 | AI integration, documentation, portfolio finalization |

**Target certifications during project:** eJPT (Month 6) → CompTIA Security+ (Month 12)

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Hypervisor | KVM/QEMU + libvirt |
| Firewall | pfSense CE 2.7 |
| Attacker | Kali Linux 2024 |
| AD / Windows | Windows Server 2019 Evaluation |
| Linux targets | Ubuntu Server 22.04, Metasploitable 3 |
| SOC / SIEM | Security Onion 2.4, Wazuh, Splunk Free |
| Monitoring | Zeek, Suricata, Kibana, ELK Stack |
| Cloud | AWS (EC2, S3, IAM, CloudTrail, VPC) |
| Automation | Python, Bash, Ansible |
| BI | Power BI |
| AI | Anthropic API (Claude) |
| Documentation | Markdown, draw.io |

---

## Repository Structure

```
FORTRESS-AI/
├── docs/
│   ├── arquitectura/     # Network diagrams, topology, ADRs
│   ├── politicas/        # ISO 27001 security policies
│   └── adr/              # Architecture Decision Records
├── fase1/                # Infrastructure setup documentation
├── fase2/                # SOC, automation, BI dashboard
├── fase3/                # Pentest cycles and DFIR analysis
├── fase4/                # AI engine and final documentation
├── scripts/
│   ├── automatizacion/   # Infrastructure automation scripts
│   ├── ofensivo/         # Offensive tooling and PoCs
│   └── forense/          # DFIR collection scripts
├── reports/
│   ├── pentest/          # Professional pentest reports (PTES)
│   ├── forense/          # DFIR reports with chain of custody
│   └── ejecutivo/        # Executive summary reports
├── evidence/
│   ├── escenario1/       # AD compromise evidence
│   ├── escenario2/       # Data exfiltration evidence
│   └── escenario3/       # Cloud attack evidence
└── ai/                   # AI engine source code
```

---

## Current Status

| Component | Status |
|-----------|--------|
| Server (Ubuntu Server 24.04) | ✅ Deployed |
| KVM/QEMU | ✅ Installed and verified |
| /fortress storage structure | ✅ Configured |
| GitHub repository | ✅ Active |
| Network segmentation (pfSense) | 🔄 In progress |
| Windows Server 2019 (AD) | ⏳ Pending |
| SOC (Security Onion) | ⏳ Pending |
| Pentest scenarios | ⏳ Pending |
| AI engine | ⏳ Pending |

---

## Academic Context

**Institution:** Universidad Tecnológica del Valle de Toluca (UTVT)  
**Program:** Ingeniería en Redes Inteligentes y Ciberseguridad  
**Modality:** Dual education — concurrent industrial practice at Continental Tire (Santa Fe, CDMX)  
**Expected graduation:** May 2027  

**Subjects covered by this project:**
Advanced Infrastructure Administration · Big Data Technologies · Data Center Infrastructure · IT Security Management · Cloud Computing · Work Planning · Ethical Hacking · Infrastructure Automation I & II · IT Quality Systems · Business Intelligence · Digital Forensics · Enterprise Network Administration · Project Management I & II · Time Management · High Performance Team Leadership · Business Negotiation

---

## Author

**Ángel Damián Malvaiz González**  
Dual student — Ingeniería en Redes Inteligentes y Ciberseguridad  
Continental Tire IT Department — L1/L2 National Support  
Toluca, México

---

> *"The best way to defend a system is to understand exactly how it gets attacked."*

---

**⚠️ Disclaimer:** All systems attacked in this project are owned and controlled by the author. No external systems are targeted. This project is for educational and research purposes only.
