# Project Charter — FORTRESS-AI

**Framework Ofensivo de Red, Testing, Respuesta y Seguridad con Inteligencia Artificial**

---

| Campo | Valor |
|---|---|
| **Código de proyecto** | FORTRESS-AI-2026 |
| **Versión del documento** | 1.0 |
| **Fecha de emisión** | 21 de abril de 2026 |
| **Autor / Project Lead** | Ángel Damián Malvaiz González |
| **Institución académica** | Universidad Tecnológica del Valle de Toluca (UTVT) |
| **Programa** | Ingeniería en Redes Inteligentes y Ciberseguridad |
| **Duración prevista** | 12 meses (Abril 2026 — Abril 2027) |
| **Estado** | En ejecución — Fase 1, Semana 2 |
| **Repositorio** | https://github.com/DamianMalvaiz/FORTRESS-AI |

---

## 1. Resumen ejecutivo

FORTRESS-AI es un proyecto académico-profesional de 12 meses que construye desde cero la infraestructura IT completa de un banco regional mexicano ficticio (**BNF — Banco Nacional Ficticio S.A.**) y ejecuta sobre ella operaciones de seguridad ofensiva de nivel consultoría profesional.

El proyecto cubre el ciclo completo de una operación de seguridad: **diseño y despliegue de infraestructura empresarial → ejecución de pentests con metodología PTES → detección desde un SOC con SIEM → análisis forense post-incidente (DFIR) → generación de reportes ejecutivos y técnicos con apoyo de inteligencia artificial**.

El resultado final es un portafolio técnico de nivel consultoría que demuestra capacidades integrales de infraestructura, red team, blue team, forense digital y automatización con IA — diferenciador real frente a perfiles que solo cubren un dominio.

---

## 2. Justificación y caso de negocio

### 2.1 Contexto académico
El programa de Ingeniería en Redes Inteligentes y Ciberseguridad de la UTVT requiere un proyecto integrador que cubra las 16 materias principales del plan de estudios. FORTRESS-AI fue diseñado para mapear directamente a estas materias, convirtiéndolas en un único entregable cohesivo en lugar de proyectos fragmentados.

### 2.2 Contexto profesional
El sector financiero mexicano es el que más invierte en ciberseguridad en el país y es el mercado principal de consultoras como **Scitum, IQsec y Totalsec** — empresas objetivo del autor tras graduación. Un proyecto que simule una infraestructura bancaria real con contexto regulatorio mexicano (**CNBV, SPEI, PCI-DSS**) genera diferenciación directa en procesos de selección.

### 2.3 Problema que resuelve
La mayoría de perfiles junior en ciberseguridad presentan en su portafolio:
- Máquinas de HackTheBox/TryHackMe aisladas sin contexto empresarial
- Scripts sueltos sin integración en una operación real
- Reportes genéricos sin formato profesional ni cadena de custodia

FORTRESS-AI resuelve esto mediante un proyecto único que demuestra capacidades end-to-end en un escenario empresarial documentado a nivel consultoría.

---

## 3. Objetivos

### 3.1 Objetivo general
Diseñar, desplegar, atacar, defender, analizar forense y documentar profesionalmente la infraestructura IT completa de una organización bancaria simulada, integrando inteligencia artificial como capa de análisis operativo.

### 3.2 Objetivos específicos

1. Desplegar una infraestructura virtualizada segmentada que replique fielmente una red empresarial bancaria, incluyendo Active Directory, servicios web, bases de datos, sistemas legacy y extensión a nube híbrida AWS.
2. Configurar un Centro de Operaciones de Seguridad (SOC) funcional con SIEM, IDS/IPS y HIDS capaz de detectar y correlacionar eventos en tiempo real.
3. Ejecutar tres ciclos completos de pentest bajo metodología PTES, cubriendo compromiso de Active Directory, exfiltración de datos y ataques desde nube.
4. Realizar análisis forense digital (DFIR) de cada escenario atacado, con reconstrucción de timeline, cadena de custodia y evidencia admisible.
5. Integrar un motor de inteligencia artificial basado en la API de Anthropic para clasificación automática de alertas SIEM, detección de anomalías y generación asistida de reportes.
6. Producir documentación técnica y ejecutiva a nivel consultoría (reportes PTES con CVSS, políticas ISO 27001, procedimientos DFIR).
7. Obtener la certificación **eJPT** al cierre de Fase 2 y **CompTIA Security+** al cierre de Fase 4.

---

## 4. Alcance

### 4.1 Incluido en el alcance

**Infraestructura:**
- 10 máquinas virtuales sobre KVM/QEMU distribuidas en 5 zonas de red segmentadas
- Firewall central pfSense CE 2.7 con reglas entre zonas
- Active Directory con dominio `fortress.bnf.local`, 200+ objetos (usuarios, grupos, equipos)
- Servicios web, bases de datos PostgreSQL, servidores de archivos, sistemas legacy (Windows 7)
- Extensión híbrida a AWS (EC2, S3, IAM, CloudTrail, VPC) en free tier

**Operaciones ofensivas (3 escenarios PTES):**
- Escenario 1: Compromiso de AD (Kerberoasting → Lateral Movement → DCSync)
- Escenario 2: Exfiltración de datos de clientes (Web App → DB → Extracción)
- Escenario 3: Ataque desde nube (AWS misconfiguration → Pivot a red interna)

**Operaciones defensivas:**
- SIEM Security Onion 2.4 con Zeek, Suricata, Kibana
- HIDS Wazuh con políticas de monitoreo
- Splunk Free con licencia educativa
- Dashboards de Power BI sobre datos del SIEM

**Análisis forense:**
- Procedimientos DFIR por escenario
- Cadena de custodia documentada
- Análisis de artefactos (memoria, disco, red, logs)

**Inteligencia artificial:**
- Clasificador de alertas SIEM (triage automatizado)
- Detector de anomalías en logs Zeek/Wazuh
- Generador asistido de borradores de reportes

**Documentación:**
- Reportes pentest formato PTES con scoring CVSS
- Reportes DFIR con cadena de custodia y timeline
- Resúmenes ejecutivos
- Política de seguridad ISO 27001 aplicada a BNF
- Architecture Decision Records (ADR)
- Manuales de operación

### 4.2 Fuera del alcance

- Sistemas reales en producción de terceros (el proyecto es 100% en infraestructura propia aislada)
- Desarrollo de malware original o herramientas ofensivas desde cero (se usan herramientas estándar: Metasploit, Impacket, etc.)
- Ataques a proveedores reales de nube fuera del free tier propio de AWS del autor
- Certificaciones CRTO y OSCP (quedan en roadmap post-proyecto como evolución natural)
- Auditoría formal externa de los entregables
- Operación continua del SOC más allá de las ventanas de pentest programadas

---

## 5. Entregables principales

### Fase 1 — Infraestructura (Meses 1–3)
- Infraestructura virtualizada operativa con 10 VMs en 5 zonas segmentadas
- Dominio Active Directory `fortress.bnf.local` con 200+ objetos
- Extensión híbrida AWS desplegada
- Diagramas de topología en draw.io
- Documentación completa del despliegue
- Video demo de la infraestructura

### Fase 2 — SOC y automatización (Meses 4–6)
- SIEM Security Onion funcional con reglas personalizadas
- Scripts de automatización en Python, Bash y Ansible
- Dashboard Power BI sobre datos del SIEM
- Política de seguridad ISO 27001 aplicada a BNF
- Certificación eJPT obtenida

### Fase 3 — Pentest y forense (Meses 7–10)
- 3 reportes pentest PTES completos con scoring CVSS y remediación
- 3 reportes DFIR con cadena de custodia y timeline de ataque
- Pruebas de concepto (PoCs) publicadas en GitHub
- Evidencia digital organizada por escenario

### Fase 4 — IA y cierre (Meses 11–12)
- Motor de IA operativo (clasificador + anomaly detector + report generator)
- Repositorio GitHub final con documentación completa
- Artículo académico describiendo la metodología
- Video demo del proyecto completo
- Presentación ejecutiva para defensa académica
- Certificación CompTIA Security+ obtenida

---

## 6. Hitos principales

| # | Hito | Fecha objetivo |
|---|---|---|
| M1 | Infraestructura base desplegada (pfSense + AD + 5 zonas) | Mes 2 |
| M2 | Fase 1 completada — lab operativo y documentado | Mes 3 |
| M3 | SOC funcional con SIEM detectando eventos simulados | Mes 5 |
| M4 | Certificación eJPT aprobada | Mes 6 |
| M5 | Escenario 1 de pentest ejecutado y reportado (AD) | Mes 7 |
| M6 | Escenario 2 de pentest ejecutado y reportado (Datos) | Mes 8 |
| M7 | Escenario 3 de pentest ejecutado y reportado (Cloud) | Mes 9 |
| M8 | Motor IA integrado y operativo sobre datos reales del lab | Mes 11 |
| M9 | Presentación ejecutiva final y certificación Security+ | Mes 12 |

---

## 7. Cronograma de alto nivel

```
Mes  1  2  3  4  5  6  7  8  9  10 11 12
     ├──Fase 1──┤
              ├──Fase 2──┤
                       ├────Fase 3────┤
                                    ├─Fase 4─┤
```

| Fase | Meses | Foco |
|---|---|---|
| Fase 1 | 1–3 | Infraestructura base, red segmentada, AD, nube híbrida |
| Fase 2 | 4–6 | SOC, SIEM, automatización, BI, ISO 27001, eJPT |
| Fase 3 | 7–10 | 3 pentests completos + 3 análisis DFIR |
| Fase 4 | 11–12 | Motor IA, documentación final, portafolio, Security+ |

---

## 8. Stakeholders

| Stakeholder | Rol | Interés |
|---|---|---|
| Ángel Damián Malvaiz González | Project Lead y ejecutor único | Desarrollo técnico, portafolio profesional, certificaciones |
| UTVT (asesor académico) | Supervisor académico | Cumplimiento de las 16 materias del programa |
| Continental Tire (empleador actual) | Soporte contextual (educación dual) | Aprendizaje aplicable al rol L1/L2 |
| Scitum / IQsec / Totalsec (potenciales) | Empleadores objetivo | Evaluación de capacidades post-graduación |

---

## 9. Recursos

### 9.1 Recursos humanos
- **Ejecutor único:** Ángel Damián Malvaiz González
- **Dedicación estimada:** 15–25 horas/semana combinando estudio, práctica y documentación

### 9.2 Hardware
| Componente | Especificación |
|---|---|
| Servidor dedicado | Acer Nitro V15 — Ryzen (24 vCPU), 32 GB RAM |
| Disco OS servidor | KIOXIA 512 GB (Ubuntu Server 24.04 LTS) |
| Disco secundario | WD 512 GB (Windows 11 uso personal) |
| Estación de trabajo | Windows 11 / Mac (planeado a partir de Mes 3) |

### 9.3 Software y licencias (todo gratuito o educativo)
| Categoría | Herramienta | Licencia |
|---|---|---|
| SO servidor | Ubuntu Server 24.04 LTS | Open source |
| Hipervisor | KVM/QEMU + libvirt | Open source |
| Firewall | pfSense CE 2.7 | Open source |
| Ofensivo | Kali Linux, Metasploit, Impacket, Burp | Open source / Community |
| SOC | Security Onion, Wazuh, ELK | Open source |
| SIEM comercial | Splunk Free | Educativa UTVT |
| Windows Server 2019 | Microsoft Evaluation | 180 días evaluación |
| Cloud | AWS Free Tier | Gratuito 12 meses |
| BI | Power BI Desktop | Gratuito |
| IA | Anthropic API (Claude) | Pago por uso (bajo costo estimado) |

### 9.4 Presupuesto estimado
| Concepto | Monto (MXN) |
|---|---|
| API Anthropic (Claude) | ~$500 |
| Dominios / hosting adicional (opcional) | ~$300 |
| Contingencia | ~$500 |
| **Total estimado** | **~$1,300 MXN** |

---

## 10. Supuestos

1. La máquina Acer Nitro V15 mantendrá su capacidad operativa durante los 12 meses del proyecto.
2. El acceso al programa AWS Free Tier estará disponible durante al menos 12 meses desde la activación.
3. Las licencias de evaluación de Microsoft y educativa de Splunk permanecerán accesibles.
4. El autor mantendrá disponibilidad mínima de 15 horas/semana para el proyecto en paralelo con trabajo en Continental Tire y estudios UTVT.
5. La API de Anthropic mantendrá acceso razonable y costos dentro del presupuesto.
6. No se requiere autorización legal externa ya que todos los sistemas atacados son propios y aislados.

---

## 11. Restricciones

1. **Presupuesto:** proyecto debe ejecutarse con costo cercano a cero (solo consumo bajo de API).
2. **Hardware único:** toda la infraestructura debe correr en una sola máquina (32 GB RAM como límite superior de VMs concurrentes).
3. **Tiempo:** plazo fijo de 12 meses alineado a graduación Mayo 2027.
4. **Aislamiento:** el lab nunca puede exponer servicios vulnerables a Internet público.
5. **Ejecutor único:** toda la carga operativa recae en el autor; no hay equipo de soporte.
6. **Compatibilidad académica:** entregables deben mapear a materias específicas del programa UTVT.

---

## 12. Riesgos principales

| # | Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|---|
| R1 | Fallo de hardware del servidor principal | Media | Alto | Backups automáticos a disco externo; documentación de reinstalación rápida |
| R2 | Insuficiencia de RAM al correr múltiples VMs pesadas (Security Onion + AD + targets) | Alta | Medio | Encender solo VMs necesarias por escenario; ajustar RAM dinámicamente |
| R3 | Consumo excesivo de API Anthropic fuera de presupuesto | Media | Bajo | Rate limiting en código; logging de tokens; fallback a modelos locales |
| R4 | Carga laboral en Continental Tire reduce tiempo disponible | Media | Medio | Planificación semanal fija; sprints de fin de semana para hitos grandes |
| R5 | Cambios en licencias de software (Splunk, AWS, MS evaluation) | Baja | Medio | Alternativas open source identificadas para cada componente |
| R6 | Scope creep — intentar agregar demasiado | Alta | Alto | Charter como referencia fija; cambios registrados como ADRs |
| R7 | Pérdida de evidencia forense por manejo incorrecto | Baja | Alto | Snapshots KVM antes de cada pentest; procedimientos DFIR documentados |
| R8 | Configuración incorrecta de aislamiento de red expone lab | Baja | Crítico | Revisión de reglas pfSense por fase; pruebas de aislamiento documentadas |

---

## 13. Criterios de éxito

El proyecto será considerado exitoso si al cumplirse los 12 meses:

1. La infraestructura BNF está desplegada, documentada y operativa.
2. Los 3 escenarios de pentest fueron ejecutados y reportados con metodología PTES.
3. Cada pentest tiene un análisis DFIR correspondiente con cadena de custodia.
4. El SIEM detectó y registró eventos de los pentests en logs disponibles.
5. El motor de IA genera salidas útiles sobre alertas reales del SIEM.
6. El repositorio GitHub está público, ordenado y con documentación completa.
7. El autor obtuvo al menos la certificación eJPT (meta mínima) — idealmente también Security+.
8. El proyecto fue presentado y defendido académicamente ante la UTVT.
9. El portafolio resultante es utilizable en procesos de selección de Scitum, IQsec o Totalsec.

---

## 14. Control de versiones del documento

| Versión | Fecha | Autor | Cambios |
|---|---|---|---|
| 1.0 | 2026-04-21 | Ángel Damián Malvaiz González | Emisión inicial del Project Charter |

---

## 15. Aprobación

| Rol | Nombre | Firma | Fecha |
|---|---|---|---|
| Project Lead | Ángel Damián Malvaiz González | _________________ | _____________ |
| Asesor académico UTVT | _________________ | _________________ | _____________ |

---

> *"The best way to defend a system is to understand exactly how it gets attacked."*

**Documento controlado — FORTRESS-AI v1.0 — 21 de abril de 2026**
