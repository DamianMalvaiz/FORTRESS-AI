# Logical Network Design — FORTRESS-AI / BNF

**Esquema de direccionamiento IP, segmentación lógica y arquitectura de red virtualizada**

> ⚠️ **Importante:** Este documento describe la **arquitectura lógica** implementada sobre virtualización KVM/QEMU. Para la arquitectura física de referencia (cómo se implementaría BNF en un banco real con hardware Cisco), consultar [Documento 03 — Architecture Reality Gap](03-architecture-reality-gap.md). Este diseño lógico es intencionalmente una simplificación del diseño físico de producción, con alcance académico y de portafolio de ciberseguridad.

---

| Campo | Valor |
|---|---|
| **Documento** | 02 — Logical Network Design |
| **Proyecto** | FORTRESS-AI |
| **Organización simulada** | BNF — Banco Nacional Ficticio S.A. |
| **Versión** | 2.0 |
| **Fecha de emisión** | 21 de abril de 2026 |
| **Autor** | Angel Damian Malvaiz González |
| **Estado** | Aprobado — Fase 1, Semana 2 |
| **Dependencias** | [01 — Project Charter](01-project-charter.md), [03 — Reality Gap](03-architecture-reality-gap.md) |

---

## Índice

1. [Resumen ejecutivo](#1-resumen-ejecutivo)
2. [Principios de diseño](#2-principios-de-diseño)
3. [Espacio de direcciones base](#3-espacio-de-direcciones-base)
4. [Zonas de red](#4-zonas-de-red)
5. [Direccionamiento por zona](#5-direccionamiento-por-zona)
6. [Inventario maestro de IPs](#6-inventario-maestro-de-ips)
7. [pfSense — gateway multi-zona](#7-pfsense--gateway-multi-zona)
8. [Matriz de comunicación entre zonas](#8-matriz-de-comunicación-entre-zonas)
9. [DHCP y resolución DNS](#9-dhcp-y-resolución-dns)
10. [Convenciones operativas](#10-convenciones-operativas)
11. [Consideraciones de seguridad](#11-consideraciones-de-seguridad)
12. [Plan de crecimiento](#12-plan-de-crecimiento)
13. [Control de versiones](#13-control-de-versiones)

---

## 1. Resumen ejecutivo

Este documento define el **direccionamiento IP y la segmentación lógica** de BNF para el proyecto FORTRESS-AI, implementado sobre virtualización KVM/QEMU. Establece cinco zonas de red con niveles de confianza diferenciados, interconectadas por un firewall central pfSense que actúa como gateway de cada zona.

### 1.1 Alcance del diseño

Este documento cubre exclusivamente la **capa lógica**:

- Asignación de subredes y rangos IP
- Dimensionamiento de zonas con VLSM
- Direccionamiento de hosts
- Matriz de comunicación entre zonas
- Convenciones de hostnames
- DHCP y DNS

**Está fuera del alcance de este documento:**

- Arquitectura física (switches, routers, cableado) — ver doc 03
- Implementación de alta disponibilidad (HA) — ver doc 03
- Configuración específica de pfSense (reglas de firewall detalladas) — documento separado en Semana 3
- Hardening de sistemas operativos — documento separado en Fase 2

### 1.2 Base tecnológica

El diseño se implementa sobre:

- **Hipervisor:** KVM/QEMU con libvirt
- **Bridges virtuales:** uno por zona (implementación L2 de KVM)
- **Firewall/Router:** pfSense CE 2.7 como VM única con múltiples interfaces virtuales
- **Rango base:** `10.0.0.0/16` reservado dentro del espacio privado `10.0.0.0/8` (RFC 1918)

### 1.3 Decisiones clave del diseño lógico

- **5 zonas de red** con niveles de confianza diferenciados
- **Segmentación mediante bridges KVM independientes** (equivalente lógico a VLANs físicas)
- **Gateway universal en `.1`** de cada zona (convención predecible)
- **Espaciamiento funcional de IPs** — rangos que cuentan la historia de la red sin consultar documentación
- **Pentesters y targets con IP estática** — la sigilosidad se logra con técnica, no con direccionamiento

### 1.4 Relación con arquitectura física de referencia

Cada zona lógica de este diseño tiene una **correspondencia conceptual** con una arquitectura física real de banco. La tabla completa de equivalencias está en el doc 03, sección 6. Resumen rápido:

| Elemento lógico (este doc) | Equivalente físico (doc 03) |
|---|---|
| Bridge KVM por zona | Switch de acceso Cisco Catalyst 2960/9300 |
| pfSense routing inter-zona | Core switch L3 Cisco Catalyst 9500 |
| pfSense firewall + router | Firewall Cisco Firepower en HA + Router ASR 1000 |
| Red doméstica como WAN | Conexión a ISP con BGP |

---

## 2. Principios de diseño

El esquema de direccionamiento lógico de BNF se rige por siete principios aplicados de forma consistente en cada zona. Estos principios están alineados con prácticas estándar de arquitectura de red bancaria y con guías de referencia como NIST SP 800-41 y CIS Controls v8.

### 2.1 Convención sobre capricho

El gateway siempre ocupa la IP `.1` de cada zona. No por razones de seguridad — un atacante dentro de la red descubre el gateway en segundos con `ip route` — sino por **mantenibilidad, auditabilidad y respuesta a incidentes**. En un escenario de crisis a las 3 AM, ningún ingeniero debería perder tiempo buscando dónde está el gateway.

### 2.2 Dimensionamiento funcional

Cada zona recibe el tamaño de subred que necesita, aplicando VLSM para evitar desperdicio o restricción:

- Zonas de alta exposición (DMZ) se mantienen pequeñas — crecer aquí significa aumentar superficie de ataque.
- Zonas de uso masivo (LAN) se dimensionan con holgura para clientes DHCP.
- Zonas críticas con crecimiento proyectado (Servers) reciben espacio para expansión.

### 2.3 Espaciamiento semántico

Las IPs no se asignan consecutivamente sin criterio. Se introducen saltos deliberados entre grupos funcionales:

```
10.0.3.10-.19  →  Servidores Windows
10.0.3.20-.29  →  Servidores Linux
10.0.3.30-.39  →  Targets vulnerables
```

Estos saltos actúan como **documentación implícita**. Cuando un analista SOC vea una alerta con IP `10.0.3.30`, sabrá inmediatamente que proviene de un sistema vulnerable sin consultar inventario.

### 2.4 Separación estático/dinámico

Hosts controlados por el equipo (servidores, appliances, estaciones de pentest) siempre con IP estática. Clientes dinámicos (PCs de empleados simulados) en rangos DHCP explícitamente reservados. Nunca se mezclan.

### 2.5 Reserva para crecimiento

Ninguna zona se dimensiona al 100% de su uso actual. Se mantiene al menos 30% de IPs libres para expansión, y se documentan las IPs reservadas para uso futuro previsto.

### 2.6 Zona de administración más restringida

La zona Management es la más pequeña y la más protegida. Toda administración de infraestructura (pfSense admin, SSH a servidores, RDP a DCs) fluye exclusivamente desde esta zona. Compromiso aquí = compromiso total.

### 2.7 El defensor aislado

El SOC vive en su propia zona con reglas estrictas: recibe tráfico de observación desde todas las zonas, pero ninguna zona puede iniciar conexiones hacia él. Esto previene que un atacante que comprometa una zona productiva pueda cegar la detección.

---

## 3. Espacio de direcciones base

### 3.1 Rango elegido

BNF utiliza el espacio privado **`10.0.0.0/8`** (RFC 1918). De este espacio masivo, el proyecto consume únicamente el bloque `10.0.0.0/16` para permitir expansión futura sin rediseño.

### 3.2 Justificación

| Rango RFC 1918 | Considerado | Decisión |
|---|---|---|
| `10.0.0.0/8` | ✅ | **Seleccionado** — estándar de industria bancaria, espacio amplio, fácil de segmentar |
| `172.16.0.0/12` | ❌ | Descartado — menos común en entornos empresariales, menos intuitivo |
| `192.168.0.0/16` | ❌ | Descartado — colisiona con red doméstica del host (`192.168.101.0/24`) |

### 3.3 Separación del entorno real

El host de virtualización (`fortress-server`, `192.168.101.104`) permanece en el rango doméstico del autor. El tráfico de BNF está completamente aislado en redes virtuales KVM y **no se enruta hacia Internet excepto mediante NAT controlado a través de pfSense** para actualizaciones de VMs específicas.

---

## 4. Zonas de red

BNF se segmenta en **cinco zonas** con niveles de confianza decrecientes conforme aumenta la exposición y crecientes conforme aumenta la criticidad interna.

### 4.1 Vista general

| # | Zona | Nivel de confianza | Función principal |
|---|---|---|---|
| 1 | **DMZ** | Baja (se asume comprometible) | Servicios expuestos a Internet |
| 2 | **LAN** | Media (usuarios humanos) | Estaciones de trabajo, targets de pentest |
| 3 | **Servers** | Alta | Corazón productivo del banco — AD, DB, web |
| 4 | **SOC** | Alta (aislada) | Monitoreo, SIEM, HIDS — observador incorruptible |
| 5 | **Management** | Máxima (MFA obligatorio) | Administración de infraestructura |

### 4.2 Diagrama conceptual (arquitectura lógica)

> **Nota:** Este diagrama muestra la topología lógica implementada en KVM. No representa la arquitectura física jerárquica (core/distribución/acceso) de un banco real. Ver doc 03, sección 3 para la arquitectura física de referencia.

```
                        INTERNET
                           │
                      ┌────▼────┐
                      │ pfSense │  (firewall + router + gateway de todas las zonas)
                      │  6 ifcs │  1 WAN + 5 LAN virtuales
                      └────┬────┘
          ┌───────────┬────┼────┬───────────┐
          │           │    │    │           │
      ┌───▼───┐  ┌───▼───┐ │ ┌──▼────┐  ┌───▼────────┐
      │  DMZ  │  │  LAN  │ │ │ SOC   │  │ Management │
      │bridge │  │bridge │ │ │bridge │  │  bridge    │
      │       │  │       │ │ │       │  │            │
      │10.0.1.│  │10.0.2.│ │ │10.0.4.│  │  10.0.5.0  │
      │ 0/28  │  │  0/24 │ │ │ 0 /28 │  │    /28     │
      └───────┘  └───────┘ │ └───────┘  └────────────┘
                       ┌───▼────┐
                       │Servers │
                       │ bridge │
                       │10.0.3.0│
                       │  /25   │
                       └────────┘

    Flujo de observación:  todas las zonas → SOC (unidireccional)
    Flujo administrativo:  Management → todas las zonas (con MFA)
```

### 4.3 Justificación de cada zona

**DMZ** — Zona de exposición controlada. Contiene servicios que **deben** alcanzarse desde Internet (portal web del banco, API móvil). Se asume comprometible por diseño. Reglas de firewall impiden que DMZ inicie conexiones hacia LAN o Servers.

**LAN** — Zona de usuarios humanos. Principal vector de phishing y compromiso inicial en ataques reales. Incluye la estación de pentest (Kali) y el target legacy (Windows 7) para los escenarios ofensivos.

**Servers** — Corazón productivo. Active Directory, bases de datos, servidores web internos. Acceso desde LAN solo por puertos específicos autenticados. Compromiso aquí = impacto crítico.

**SOC** — Observador de la red. Recibe logs, espejos de tráfico y alertas de todas las zonas. Aislado para que un atacante que comprometa otra zona no pueda apagar la detección ni borrar evidencia.

**Management** — Zona de administración. Única puerta legítima para modificar infraestructura. Pequeña, restrictiva, con MFA obligatorio en todos los accesos.

---

## 5. Direccionamiento por zona

### 5.1 Tabla resumen

| Zona | Subred CIDR | Máscara | Gateway | Broadcast | IPs usables |
|---|---|---|---|---|---|
| DMZ | `10.0.1.0/28` | 255.255.255.240 | 10.0.1.1 | 10.0.1.15 | 14 |
| LAN | `10.0.2.0/24` | 255.255.255.0 | 10.0.2.1 | 10.0.2.255 | 254 |
| Servers | `10.0.3.0/25` | 255.255.255.128 | 10.0.3.1 | 10.0.3.127 | 126 |
| SOC | `10.0.4.0/28` | 255.255.255.240 | 10.0.4.1 | 10.0.4.15 | 14 |
| Management | `10.0.5.0/28` | 255.255.255.240 | 10.0.5.1 | 10.0.5.15 | 14 |

### 5.2 Zona DMZ — `10.0.1.0/28`

```
Red:         10.0.1.0
Gateway:     10.0.1.1    (pfSense interface virtual em0 → bridge br-dmz)
Broadcast:   10.0.1.15
Rango útil:  10.0.1.1 – 10.0.1.14
```

| IP | Dispositivo | Estado | Notas |
|---|---|---|---|
| 10.0.1.1 | Gateway pfSense DMZ | Semana 3 | Interface virtual em0 |
| 10.0.1.2 – 10.0.1.9 | Reserva infraestructura | — | DNS perimetral futuro, NTP |
| 10.0.1.10 | Web público BNF (futuro) | Fase 3 | Portal banca en línea expuesto |
| 10.0.1.11 | Reverse proxy / WAF (futuro) | Fase 3 | Frontend de protección |
| 10.0.1.12 – 10.0.1.14 | Reserva expansión | — | — |

**Notas de diseño:** la DMZ arranca vacía en Fase 1. Se poblará durante el Escenario 3 de pentest (ataque desde nube con pivote hacia DMZ). Mantener solo 3 slots libres es intencional — exponer más servicios aumenta superficie sin agregar valor simulado.

### 5.3 Zona LAN — `10.0.2.0/24`

```
Red:         10.0.2.0
Gateway:     10.0.2.1    (pfSense interface virtual em1 → bridge br-lan)
Broadcast:   10.0.2.255
Rango útil:  10.0.2.1 – 10.0.2.254
```

| Rango | Uso | Asignaciones |
|---|---|---|
| 10.0.2.1 | Gateway pfSense LAN | Activo Semana 3 |
| 10.0.2.2 – 10.0.2.9 | Reserva infraestructura | DNS secundario, DHCP relay |
| 10.0.2.10 – 10.0.2.49 | Hosts estáticos usuario | Windows 7 en `.10` |
| 10.0.2.50 – 10.0.2.99 | Workstations especiales | Kali Linux en `.50` |
| 10.0.2.100 – 10.0.2.199 | Pool DHCP — PCs simuladas | Activación Fase 2 |
| 10.0.2.200 – 10.0.2.250 | Impresoras, IoT, casos especiales | Reserva |
| 10.0.2.251 – 10.0.2.254 | Reserva administrativa | — |

**Asignaciones específicas:**

| IP | VM | Razón |
|---|---|---|
| **10.0.2.10** | Windows 7 SP1 | Primer host estático — target legacy (EternalBlue, CVE-2017-0144). IP fija facilita trazabilidad en reportes PTES |
| **10.0.2.50** | Kali Linux | Apartado del rango de usuarios para distinción visual en logs. IP fija = pentest profesional con trazabilidad |

**Nota sobre Kali con IP estática:** en engagements profesionales (Scitum, IQsec), la máquina del pentester siempre tiene IP fija conocida — es lo que va en el contrato y en los reportes. La "invisibilidad" del atacante viene de técnicas de evasión (proxying, ofuscación, timing) y no del direccionamiento. Kali dinámico rompería la correlación SIEM y los reportes forenses.

### 5.4 Zona Servers — `10.0.3.0/25`

```
Red:         10.0.3.0
Gateway:     10.0.3.1    (pfSense interface virtual em2 → bridge br-servers)
Broadcast:   10.0.3.127
Rango útil:  10.0.3.1 – 10.0.3.126
```

| Rango | Función | Asignaciones |
|---|---|---|
| 10.0.3.1 | Gateway pfSense Servers | Activo Semana 3 |
| 10.0.3.2 – 10.0.3.9 | Reserva infraestructura | DNS secundario, NTP |
| 10.0.3.10 – 10.0.3.19 | Servidores Windows | DC `.10`, File Server `.11` |
| 10.0.3.20 – 10.0.3.29 | Servidores Linux productivos | Web `.20`, DB `.21` |
| 10.0.3.30 – 10.0.3.39 | **Targets vulnerables** | Metasploitable `.30` |
| 10.0.3.40 – 10.0.3.126 | Reserva expansión | DC secundario, Exchange, más targets |

**Asignaciones específicas:**

| IP | VM | Razón |
|---|---|---|
| **10.0.3.10** | Windows Server 2019 #1 — DC + DNS primario | Máquina más crítica. Primera IP "servidor" = aparece primero en logs, es la que más se consulta |
| **10.0.3.11** | Windows Server 2019 #2 — File Server + IIS | Segunda máquina Windows, consecutiva al DC |
| **10.0.3.20** | Ubuntu Server — Portal banca en línea | Salto a `.20` para separar visualmente stack Windows de Linux |
| **10.0.3.21** | Ubuntu Server — PostgreSQL (datos clientes) | Consecutiva al Web porque son stack conjunto |
| **10.0.3.30** | Metasploitable 3 | Salto a `.30` — identificación visual inmediata como target vulnerable |

**El poder del espaciamiento funcional:** cuando un analista SOC vea en una alerta `10.0.3.30 → 10.0.3.10 puerto 445`, entenderá al instante la historia: un sistema vulnerable intenta alcanzar al DC. Este es el patrón exacto de lateral movement post-compromiso inicial que se simulará en el Escenario 1 del pentest. **El diseño de IPs literalmente narra el ataque.**

### 5.5 Zona SOC — `10.0.4.0/28`

```
Red:         10.0.4.0
Gateway:     10.0.4.1    (pfSense interface virtual em3 → bridge br-soc)
Broadcast:   10.0.4.15
Rango útil:  10.0.4.1 – 10.0.4.14
```

| IP | Dispositivo | Estado | RAM |
|---|---|---|---|
| 10.0.4.1 | Gateway pfSense SOC | Semana 3 | — |
| 10.0.4.2 – 10.0.4.9 | Reserva infraestructura | — | — |
| **10.0.4.10** | Security Onion 2.4 (SIEM + IDS/IPS) | Fase 2 | 14 GB |
| **10.0.4.11** | Wazuh Manager (HIDS) | Fase 2 | 3 GB |
| 10.0.4.12 | Splunk Free (futuro) | Fase 2 | — |
| 10.0.4.13 – 10.0.4.14 | Reserva — sensor IDS adicional, honeypot | — | — |

**Notas de diseño:** el SOC no crece como los servidores productivos — tiene un número fijo de herramientas y no se expande proporcionalmente al negocio. `/28` es suficiente y permanente. Security Onion recibe la IP más baja del bloque de herramientas (`.10`) porque es la pieza central del SOC.

### 5.6 Zona Management — `10.0.5.0/28`

```
Red:         10.0.5.0
Gateway:     10.0.5.1    (pfSense interface virtual em4 → bridge br-mgmt)
Broadcast:   10.0.5.15
Rango útil:  10.0.5.1 – 10.0.5.14
```

| IP | Dispositivo | Estado |
|---|---|---|
| 10.0.5.1 | Gateway pfSense Management — acceso al panel admin pfSense | Activo Semana 3 |
| 10.0.5.2 – 10.0.5.9 | Reserva infraestructura | — |
| **10.0.5.10** | Bastion host — jump server (futuro) | Fase 2 |
| **10.0.5.11** | Ansible control node (futuro) | Fase 2 |
| 10.0.5.12 – 10.0.5.14 | Reserva consolas admin | — |

**Notas sobre el bastion host:** en Fase 2, toda administración de BNF (SSH a Linux, RDP a Windows, acceso a paneles web) pasará exclusivamente por el bastion host. Esto implementa el patrón **"jump server"** de las arquitecturas de consultoría — única puerta de entrada admin, con MFA, logs de sesión completos, grabación de comandos. Un atacante que quiera administrar infraestructura debe primero comprometer el bastion. Es una capa de defensa adicional y una fuente rica de evidencia forense.

---

## 6. Inventario maestro de IPs

### 6.1 Vista consolidada — todas las VMs actuales y planificadas

| IP | Hostname | Zona | Rol | RAM | Fase |
|---|---|---|---|---|---|
| 10.0.1.1 | fw-dmz | DMZ | pfSense interface virtual em0 | — | 1 |
| 10.0.1.10 | bnf-web-public | DMZ | Portal público BNF | 2 GB | 3 |
| 10.0.1.11 | bnf-waf | DMZ | Reverse proxy / WAF | 1 GB | 3 |
| 10.0.2.1 | fw-lan | LAN | pfSense interface virtual em1 | — | 1 |
| 10.0.2.10 | lab-win7-01 | LAN | Windows 7 SP1 (target legacy) | 2 GB | 1 |
| 10.0.2.50 | attacker-kali | LAN | Kali Linux (pentest) | 3 GB | 1 |
| 10.0.3.1 | fw-srv | Servers | pfSense interface virtual em2 | — | 1 |
| 10.0.3.10 | dc01-bnf | Servers | Windows Server 2019 — DC + DNS | 4 GB | 1 |
| 10.0.3.11 | fs01-bnf | Servers | Windows Server 2019 — File Server + IIS | 3 GB | 1 |
| 10.0.3.20 | web01-bnf | Servers | Ubuntu — Portal banca interno | 2 GB | 1 |
| 10.0.3.21 | db01-bnf | Servers | Ubuntu — PostgreSQL datos clientes | 2 GB | 1 |
| 10.0.3.30 | vuln-meta3 | Servers | Metasploitable 3 (target) | 2 GB | 1 |
| 10.0.4.1 | fw-soc | SOC | pfSense interface virtual em3 | — | 1 |
| 10.0.4.10 | soc-secuonion | SOC | Security Onion 2.4 | 14 GB | 2 |
| 10.0.4.11 | soc-wazuh | SOC | Wazuh Manager | 3 GB | 2 |
| 10.0.4.12 | soc-splunk | SOC | Splunk Free | 2 GB | 2 |
| 10.0.5.1 | fw-mgmt | Management | pfSense interface virtual em4 + panel admin | — | 1 |
| 10.0.5.10 | mgmt-bastion | Management | Bastion host (jump server) | 1 GB | 2 |
| 10.0.5.11 | mgmt-ansible | Management | Ansible control node | 1 GB | 2 |

### 6.2 Convención de hostnames

```
[función]-[identificador]-[entorno]
```

Ejemplos:
- `dc01-bnf` → Domain Controller #1 de BNF
- `web01-bnf` → Web server #1 de BNF
- `soc-secuonion` → SOC, Security Onion
- `fw-lan` → Firewall en interface LAN
- `attacker-kali` → Atacante autorizado, plataforma Kali

Esta convención permite filtrar rápidamente en logs y en el SIEM: `hostname contains "dc"` → todos los controladores de dominio.

---

## 7. pfSense — gateway multi-zona

pfSense es la **única VM con presencia en las cinco zonas**. Tiene una interface de red virtual por cada zona, y cada interface actúa simultáneamente como gateway de esa zona y como punto de control de reglas de firewall.

### 7.1 Mapa de interfaces virtuales

| Interface virtual | Zona | IP | Red conectada | Bridge KVM asociado |
|---|---|---|---|---|
| em0 | DMZ | 10.0.1.1/28 | 10.0.1.0/28 | br-dmz |
| em1 | LAN | 10.0.2.1/24 | 10.0.2.0/24 | br-lan |
| em2 | Servers | 10.0.3.1/25 | 10.0.3.0/25 | br-servers |
| em3 | SOC | 10.0.4.1/28 | 10.0.4.0/28 | br-soc |
| em4 | Management | 10.0.5.1/28 | 10.0.5.0/28 | br-mgmt |

**Nota sobre identificadores de zona:** las etiquetas como "VLAN 10, 20, 30..." que aparecen en diagramas y documentación son **identificadores descriptivos de zona**, no configuración 802.1Q real. Cada zona está implementada como un bridge KVM independiente (equivalencia lógica a una VLAN física, sin tagging). Ver doc 03, sección 4.3, decisión 7, para contexto completo.

### 7.2 Interface WAN

Además de las 5 interfaces internas, pfSense tiene una interface WAN adicional que recibe IP del host (`192.168.101.x`, asignada por el router doméstico vía DHCP). Esta interface permite NAT saliente para actualizaciones de VMs pero **nunca acepta conexiones entrantes desde Internet** en el lab local.

### 7.3 Acceso al panel admin

El panel web de pfSense escucha **exclusivamente en la interface Management (`10.0.5.1:443`)**. Ninguna otra zona puede alcanzar el panel admin. Esto se refuerza con reglas explícitas en la capa de firewall.

### 7.4 Rol de pfSense en la arquitectura lógica

En FORTRESS-AI, pfSense cumple simultáneamente cinco roles que en un banco real estarían separados en dispositivos físicos distintos (ver doc 03, sección 6, tabla de equivalencias):

1. **Firewall** — inspección y filtrado de tráfico entre zonas
2. **Router** — enrutamiento L3 entre subredes
3. **Gateway** — primera dirección IP en cada zona
4. **DHCP server** — asignación dinámica en zona LAN
5. **VPN gateway** — conexión a AWS en Fase 3

Esta consolidación de funciones es una **decisión deliberada de simplificación** consistente con el alcance académico del proyecto.

---

## 8. Matriz de comunicación entre zonas

Esta tabla define qué zonas pueden iniciar conexiones hacia otras zonas. Es la base que se traducirá a reglas concretas de pfSense en Semana 3.

### 8.1 Leyenda

- ✅ = Permitido por puertos específicos
- ❌ = Bloqueado por default
- 👁 = Solo observación (logs, espejo de tráfico, unidireccional)
- 🔐 = Permitido solo con MFA / bastion host

### 8.2 Matriz

| Origen ↓ / Destino → | DMZ | LAN | Servers | SOC | Management | Internet |
|---|---|---|---|---|---|---|
| **DMZ** | — | ❌ | ❌ | 👁 | ❌ | ❌ |
| **LAN** | ✅ HTTP/S | — | ✅ AD, SMB, HTTP/S | 👁 | ❌ | ✅ NAT |
| **Servers** | ❌ | ❌ | — | 👁 | ❌ | ✅ NAT controlado |
| **SOC** | ❌ | ❌ | ❌ | — | ❌ | ❌ |
| **Management** | 🔐 | 🔐 | 🔐 | 🔐 | — | ✅ NAT |
| **Internet** | ✅ HTTP/S puerto 80/443 | ❌ | ❌ | ❌ | ❌ | — |

### 8.3 Reglas clave derivadas

1. **DMZ no puede iniciar conexiones internas.** Si un atacante compromete un servidor DMZ, queda contenido.
2. **El SOC solo observa, nunca inicia.** Recibe syslog (UDP 514), agentes Wazuh (TCP 1514/1515), espejo de tráfico. No responde ni origina.
3. **Management es el único origen admin.** Todo SSH, RDP y acceso a paneles admin parte de esta zona.
4. **Internet solo alcanza DMZ.** Ninguna otra zona recibe tráfico entrante externo.
5. **NAT saliente restringido.** Servers solo puede iniciar a Internet para actualizaciones específicas (apt, Windows Update) en puertos controlados.

### 8.4 Puertos específicos autorizados (extracto — detalle completo en doc de firewall)

| Origen | Destino | Puerto | Protocolo | Uso |
|---|---|---|---|---|
| LAN | Servers `.10` (DC) | 53 | UDP/TCP | DNS |
| LAN | Servers `.10` (DC) | 88 | TCP | Kerberos |
| LAN | Servers `.10` (DC) | 389 | TCP | LDAP |
| LAN | Servers `.11` (File) | 445 | TCP | SMB |
| LAN | Servers `.20` (Web) | 443 | TCP | HTTPS banca interna |
| Todas | SOC `.10` | 514 | UDP | Syslog |
| Todas (agentes) | SOC `.11` | 1514/1515 | TCP | Wazuh |
| Management `.10` | Todas | 22 | TCP | SSH desde bastion |
| Management `.10` | Servers (Win) | 3389 | TCP | RDP desde bastion |
| Management `.10` | pfSense `.1` de cada zona | 443 | TCP | Admin pfSense |

---

## 9. DHCP y resolución DNS

### 9.1 Servidores DHCP

| Zona | Servicio DHCP | Rango | Notas |
|---|---|---|---|
| DMZ | Deshabilitado | — | Solo IPs estáticas (superficie mínima) |
| LAN | pfSense DHCP | 10.0.2.100 – 10.0.2.199 | Activación Fase 2 para simular PCs empleados |
| Servers | Deshabilitado | — | Solo estáticas |
| SOC | Deshabilitado | — | Solo estáticas |
| Management | Deshabilitado | — | Solo estáticas |

### 9.2 Resolución DNS

**DNS primario de BNF:** `10.0.3.10` (Domain Controller Windows Server 2019)

- Todas las VMs Windows unidas al dominio `fortress.bnf.local` consultan al DC
- Servidores Linux consultan al DC como DNS primario, con fallback a `1.1.1.1` para resolución externa
- El DC hace forwarding a `1.1.1.1` y `8.8.8.8` para resolución externa

**Dominio:** `fortress.bnf.local`

Ejemplos de FQDN esperados:
- `dc01.fortress.bnf.local` → 10.0.3.10
- `fs01.fortress.bnf.local` → 10.0.3.11
- `web01.fortress.bnf.local` → 10.0.3.20

---

## 10. Convenciones operativas

### 10.1 Convención de IPs por rango dentro de cada zona

Aplicable a zonas `/24` y mayores. En zonas `/28` el espíritu se mantiene aunque los rangos se compacten:

| Rango | Uso |
|---|---|
| `.1` | Gateway (pfSense) |
| `.2 – .9` | Infraestructura de red (DNS, DHCP, NTP) |
| `.10 – .49` | Servidores con IP estática |
| `.50 – .99` | Appliances, workstations especiales |
| `.100 – .199` | Rango DHCP |
| `.200 – .250` | Reserva casos especiales |
| `.251 – .254` | Reserva administrativa |

### 10.2 Documentación de cambios

Cualquier cambio en el esquema de IPs (asignación, movimiento, retiro) se registra como un **Architecture Decision Record (ADR)** en `docs/adr/` y referencia este documento.

### 10.3 Snapshots antes de cambios

Antes de cualquier cambio de red en pfSense o en VMs críticas, se toma snapshot KVM. Permite rollback en minutos si una regla mal diseñada rompe conectividad.

### 10.4 Nombres de bridges KVM

Los bridges KVM se nombran con el prefijo `br-` seguido del nombre corto de la zona:

| Bridge | Zona |
|---|---|
| `br-dmz` | DMZ |
| `br-lan` | LAN |
| `br-servers` | Servers |
| `br-soc` | SOC |
| `br-mgmt` | Management |

Esto facilita identificación rápida en comandos `brctl show` y en archivos XML de libvirt.

---

## 11. Consideraciones de seguridad

### 11.1 Defense in Depth aplicado a la arquitectura lógica

El direccionamiento IP y la segmentación son **la capa 2 (segmentación lógica)** de una estrategia de defensa en profundidad que incluye:

| Capa | Implementación en BNF simulado |
|---|---|
| 1. Perímetro | Reglas pfSense entre zonas |
| 2. Segmentación | Bridges KVM + subredes definidas en este documento |
| 3. Acceso | Active Directory + AAA en pfSense + MFA en bastion |
| 4. Detección | Security Onion + Wazuh observando todas las zonas |
| 5. Host | Hardening por VM, firewall local, actualizaciones |
| 6. Datos | Cifrado en reposo (BitLocker), TLS en tránsito |

> **Nota importante:** en un banco real, la capa 1 (perímetro) incluiría también protecciones físicas (seguridad en datacenter, control de acceso biométrico, CCTV) y la capa 2 incluiría VLANs 802.1Q reales con protocolos L2 de protección (DHCP Snooping, Dynamic ARP Inspection, 802.1X). Estos elementos están fuera del alcance de la simulación lógica. Ver doc 03 para contexto completo.

### 11.2 Riesgos identificados y mitigados en el diseño lógico

| # | Riesgo | Mitigación aplicada en este diseño |
|---|---|---|
| R1 | Atacante pivota de DMZ a LAN | Matriz de comunicación bloquea inicio de conexiones desde DMZ |
| R2 | Atacante borra logs al comprometer Servers | SOC aislado en zona dedicada; Servers no puede iniciar hacia SOC |
| R3 | Atacante accede a panel pfSense | Panel solo accesible desde Management, con MFA |
| R4 | Colisión entre lab y red doméstica | Rangos `10.0.0.0/16` separados de `192.168.101.0/24` doméstica |
| R5 | Servicios DMZ comprometidos sirven de pivote a Internet | NAT saliente desde DMZ restringido a respuestas de conexiones establecidas |

### 11.3 Lo que NO protege este diseño

Para ser honestos sobre alcance, este diseño **no** protege contra:

- Phishing que comprometa credenciales de usuarios LAN legítimos
- Malware ejecutado por un usuario con privilegios
- Vulnerabilidades de día cero en servicios expuestos
- Ataques físicos al hardware del servidor
- Escenarios de alta disponibilidad (caída de pfSense = caída del lab)
- Ataques a la capa física real (no aplica porque no hay capa física — ver doc 03)

Para esos vectores se depende de las otras capas de defensa (EDR, GPO, patching, seguridad física) y, en una implementación real, de la arquitectura física redundante.

---

## 12. Plan de crecimiento

### 12.1 Capacidad disponible

Espacio reservado para futuras zonas dentro del `10.0.0.0/16` base:

| Rango | Uso futuro previsto |
|---|---|
| `10.0.3.128/25` | Segunda mitad de Servers — futura zona DB dedicada si el proyecto escala |
| `10.0.6.0/24` – `10.0.9.0/24` | 4 zonas adicionales disponibles |
| `10.0.10.0/24` | Zona Cloud-interconnect (VPN a AWS) |
| `10.0.20.0/24` | Zona IoT / dispositivos bancarios especiales |
| `10.0.100.0/24` | Zona backup / replicación |

### 12.2 Hitos de expansión

| Fase | Expansión prevista |
|---|---|
| Fase 1 (actual) | 10 VMs en las 5 zonas base |
| Fase 2 | Activación DHCP LAN, bastion host, ansible, segundo SIEM |
| Fase 3 | Población DMZ con servicios expuestos (web público + WAF) |
| Fase 4 | IP para VM del motor IA (probable en SOC como analizador) |

### 12.3 Posible evolución técnica futura

Cambios que se podrían implementar más adelante sin romper este diseño base:

- **Migración a VLAN tagging 802.1Q real** con Open vSwitch, si se decide profundizar en protocolos L2 reales
- **Implementación de pfSense HA con CARP**, si se consiguen recursos adicionales (más RAM o segundo servidor)
- **Integración con equipos Cisco físicos**, si se consigue hardware prestado (ver doc 03, sección 11.1)

Cualquiera de estas evoluciones se documentaría como un **ADR (Architecture Decision Record)** antes de implementar.

---

## 13. Control de versiones

| Versión | Fecha | Autor | Cambios |
|---|---|---|---|
| 1.0 | 2026-04-21 | Ángel Damián Malvaiz González | Emisión inicial del diseño de red |
| 2.0 | 2026-04-21 | Ángel Damián Malvaiz González | Alineación con doc 03 (Reality Gap). Título cambiado a "Logical Network Design". Aclaración sobre VLAN IDs descriptivas vs 802.1Q real. Referencias cruzadas a doc 03. Nuevas secciones: 7.4 (rol de pfSense), 10.4 (nombres bridges KVM), 12.3 (evolución futura). Ampliadas secciones 1, 4.2 y 11 con contexto de simulación lógica. |

---

## Anexo A — Referencias técnicas

### Direccionamiento y protocolos
- RFC 1918 — Address Allocation for Private Internets
- RFC 4632 — Classless Inter-domain Routing (CIDR)

### Seguridad de red
- NIST SP 800-41 Rev. 1 — Guidelines on Firewalls and Firewall Policy
- NIST SP 800-125 — Guide to Security for Full Virtualization Technologies
- CIS Controls v8 — Control 12 (Network Infrastructure Management)
- PCI-DSS v4.0 — Requirement 1 (Network Security Controls)

### Arquitectura de referencia (usada en doc 03)
- Cisco Enterprise Campus Architecture
- Cisco SAFE Reference Architecture

## Anexo B — Glosario rápido

| Término | Definición |
|---|---|
| **CIDR** | Classless Inter-Domain Routing — notación `/N` para máscara de subred |
| **VLSM** | Variable Length Subnet Masking — usar máscaras distintas por subred |
| **DMZ** | Demilitarized Zone — zona de exposición controlada a Internet |
| **SOC** | Security Operations Center — zona/equipo de monitoreo y respuesta |
| **SIEM** | Security Information and Event Management — centralización y correlación de logs |
| **HIDS** | Host-based Intrusion Detection System — detección desde el endpoint |
| **IDS/IPS** | Intrusion Detection/Prevention System — detección/prevención basada en red |
| **Bastion host** | Servidor intermediario único para administración, con MFA y auditoría |
| **RFC 1918** | Documento que define los rangos de IP privadas |
| **DHCP** | Dynamic Host Configuration Protocol — asignación dinámica de IPs |
| **Bridge KVM** | Switch virtual L2 implementado en el kernel de Linux, usado por libvirt para interconectar VMs |
| **libvirt** | API y conjunto de herramientas para gestionar virtualización en Linux (incluye KVM, QEMU, Xen) |

---

> *"Un buen direccionamiento de red no solo conecta máquinas — cuenta la historia de la arquitectura sin necesidad de diagrama."*

**Documento controlado — FORTRESS-AI · Logical Network Design v2.0 · 21 de abril de 2026**
