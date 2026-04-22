# Architecture Reality Gap — FORTRESS-AI / BNF

**Declaración formal del alcance de la simulación lógica y su relación con la arquitectura física de referencia**

---

| Campo | Valor |
|---|---|
| **Documento** | 03 — Architecture Reality Gap |
| **Proyecto** | FORTRESS-AI |
| **Organización simulada** | BNF — Banco Nacional Ficticio S.A. |
| **Versión** | 1.0 |
| **Fecha de emisión** | 21 de abril de 2026 |
| **Autor** | Ángel Damián Malvaiz González |
| **Estado** | Aprobado — Fase 1, Semana 2 |
| **Dependencias** | [01 — Project Charter](01-project-charter.md), [02 — Network Design](02-network-design.md) |

---

## Índice

1. [Propósito de este documento](#1-propósito-de-este-documento)
2. [Declaración formal de alcance](#2-declaración-formal-de-alcance)
3. [Arquitectura física de referencia](#3-arquitectura-física-de-referencia)
4. [Arquitectura lógica implementada](#4-arquitectura-lógica-implementada)
5. [Comparación lado a lado](#5-comparación-lado-a-lado)
6. [Tabla de equivalencias](#6-tabla-de-equivalencias)
7. [Lo que SÍ se simula](#7-lo-que-sí-se-simula)
8. [Lo que NO se simula](#8-lo-que-no-se-simula)
9. [Justificación académica y profesional](#9-justificación-académica-y-profesional)
10. [Respuesta preparada para preguntas frecuentes](#10-respuesta-preparada-para-preguntas-frecuentes)
11. [Roadmap de extensión a producción](#11-roadmap-de-extensión-a-producción)
12. [Control de versiones](#12-control-de-versiones)

---

## 1. Propósito de este documento

Este documento existe para **declarar honestamente** el alcance del proyecto FORTRESS-AI y prevenir malentendidos sobre lo que se está construyendo.

FORTRESS-AI es una **simulación lógica** de la arquitectura de red, seguridad y operaciones de un banco regional mexicano. **No es** una réplica física de un datacenter bancario ni pretende serlo. Las simplificaciones y omisiones respecto a una arquitectura física real son **deliberadas, documentadas y justificadas**.

Este documento sirve tres propósitos:

1. **Transparencia académica** — declarar con precisión qué se entrega y qué no, para que los evaluadores de la UTVT puedan evaluar el proyecto contra su alcance real y no contra expectativas infladas.

2. **Transparencia profesional** — permitir que cualquier revisor (consultora, reclutador, profesor externo) entienda las decisiones de diseño y la brecha consciente respecto a producción.

3. **Soporte para defensa técnica** — proporcionar respuestas preparadas a preguntas esperables del tipo "¿por qué no tiene X?" o "¿cómo escalaría a Y?".

> **Principio que guía este documento:** un proyecto honesto sobre sus limitaciones genera más confianza técnica que uno que las oculta.

---

## 2. Declaración formal de alcance

### 2.1 Lo que FORTRESS-AI es

FORTRESS-AI es una **simulación lógica** enfocada en el **plano de control, seguridad y observabilidad** de una infraestructura bancaria. Se ejecuta sobre virtualización KVM/QEMU en hardware único (Acer Nitro V15, 32 GB RAM). Incluye:

- Arquitectura lógica de red segmentada en 5 zonas
- Firewall central (pfSense) enrutando entre zonas
- Active Directory con dominio empresarial
- Servicios bancarios emulados (web, DB, file server)
- Sistemas legacy (Windows 7) como superficie de ataque
- SOC funcional con SIEM, IDS/IPS y HIDS
- Tres ciclos completos de pentest bajo metodología PTES
- Análisis forense digital (DFIR) post-ataque
- Motor de inteligencia artificial para análisis de alertas

### 2.2 Lo que FORTRESS-AI NO es

- **No es** una réplica de la infraestructura física de un datacenter bancario real
- **No es** un lab de CCNA para practicar configuración de equipos Cisco físicos
- **No es** un entorno de alta disponibilidad (HA) con redundancia
- **No es** un sistema productivo ni un piloto de producción
- **No es** una prueba de throughput ni un benchmark de rendimiento
- **No es** un reemplazo para experiencia en operación de hardware real

### 2.3 Enfoque del proyecto

FORTRESS-AI prioriza:

- **Profundidad conceptual** sobre cobertura de hardware
- **Capas de seguridad, detección y forense** sobre topología física detallada
- **Escenarios de ataque-defensa completos** sobre demostración de throughput
- **Documentación nivel consultoría** sobre diagramas de cableado físico
- **Portabilidad** (un solo servidor, open source) sobre infraestructura empresarial

---

## 3. Arquitectura física de referencia

Esta sección describe cómo se **implementaría BNF en producción real** si fuera un banco operativo con 800 empleados y 50 sucursales. Este es el modelo mental contra el cual se compara la simulación.

### 3.1 Topología jerárquica de tres capas

BNF real seguiría el **modelo jerárquico clásico** de Cisco para redes empresariales: core / distribución / acceso.

```
                           ┌──────────────────┐
                           │   Internet (ISP) │
                           └────────┬─────────┘
                                    │
                           ┌────────▼─────────┐
                           │  Router de borde │   Cisco ASR 1000 series
                           │  (BGP con ISP)   │   (redundante en par)
                           └────────┬─────────┘
                                    │ Port-channel LACP
                           ┌────────▼─────────┐
                           │  Firewall en HA  │   Cisco Firepower / pfSense
                           │  (par activo-    │   en cluster HA
                           │   pasivo)        │
                           └────────┬─────────┘
                                    │ Port-channel
                           ┌────────▼─────────┐
                           │  SWITCH CORE L3  │   Cisco Catalyst 9500
                           │  (par redundante │   (40/100 Gbps backbone)
                           │   con VSS/StackWise)
                           └─┬──────┬──────┬──┘
                             │      │      │ (port-channels redundantes)
                 ┌───────────┘      │      └────────────┐
          ┌──────▼─────┐    ┌───────▼───────┐    ┌──────▼──────┐
          │Distribución│    │ Distribución  │    │Distribución │   Cisco Catalyst 9400
          │   DMZ      │    │   LAN         │    │   Servers   │
          └──┬─────────┘    └──┬────────────┘    └──┬──────────┘
             │                 │                    │
          [Acceso]          [Acceso]             [Acceso]         Cisco Catalyst 9300
             │                 │                    │
          [Hosts DMZ]       [PCs empleados]      [Servidores]
```

### 3.2 Componentes físicos reales

| Capa | Dispositivo típico en banco mediano | Función |
|---|---|---|
| Borde | Cisco ASR 1001-HX | Enrutamiento con ISP, BGP, filtrado básico |
| Firewall | Cisco Firepower 2100 en HA / par de pfSense con CARP | Inspección profunda, IPS, segmentación |
| Core | Cisco Catalyst 9500 en par con VSS | Routing inter-VLAN de alto throughput |
| Distribución | Cisco Catalyst 9400 | Agregación y políticas por zona |
| Acceso | Cisco Catalyst 9300 / 2960-X | Conectividad a usuarios finales |
| WLC | Cisco 9800 series | Gestión de access points corporativos |

### 3.3 Protocolos y tecnologías que estarían activos

- **STP / RSTP / MST** — prevención de loops en la malla
- **HSRP / VRRP / GLBP** — gateway redundante en core
- **OSPF / EIGRP** — enrutamiento dinámico interno
- **BGP** — peering con ISPs para tráfico Internet
- **LACP / EtherChannel** — agregación de enlaces (port-channels)
- **802.1Q / VLAN trunking** — segmentación L2
- **Private VLANs** — aislamiento intra-VLAN
- **DHCP Snooping, Dynamic ARP Inspection** — protección L2
- **802.1X / NAC** — control de acceso por puerto
- **TACACS+ / RADIUS** — AAA para administración
- **SNMP v3, NetFlow** — monitoreo y observabilidad

### 3.4 Consideraciones de disponibilidad y capacidad

Un BNF real dimensionaría:

- **Dual-homed** conexiones a Internet con 2 ISPs distintos
- **N+1 redundancia** en firewall, core, distribución
- **Backup power** con UPS y generadores
- **Climatización redundante** en salas de telecomunicaciones
- **DRP (Disaster Recovery)** con sitio secundario geográficamente separado
- **Out-of-band management** red física aislada para administración

---

## 4. Arquitectura lógica implementada

Esta sección describe **lo que realmente se construye** en FORTRESS-AI, ejecutándose sobre virtualización KVM/QEMU.

### 4.1 Topología plana simplificada

```
                      ┌──────────────┐
                      │   Internet   │  (router doméstico del autor)
                      └──────┬───────┘
                             │ NAT
                      ┌──────▼───────┐
                      │   pfSense    │  (VM única con 6 interfaces virtuales)
                      │ (1 instancia)│  rol: firewall + router + DHCP + DNS
                      └──────┬───────┘
            ┌─────┬──────────┼──────────┬─────────┐
            │     │          │          │         │
         [br-dmz][br-lan][br-servers][br-soc][br-mgmt]   bridges KVM
            │     │          │          │         │
         [VMs  ][VMs     ][VMs      ][VMs    ][VMs     ]
          DMZ  ] LAN     ] Servers  ] SOC    ] Mgmt    ]
```

### 4.2 Componentes virtualizados

| Rol | Implementación real |
|---|---|
| "Router de borde" | No existe — pfSense WAN se conecta directo al router doméstico |
| Firewall | pfSense CE 2.7 como VM única (sin HA) |
| "Core switch" | No existe como entidad independiente — pfSense enruta entre bridges |
| "Switches de distribución" | No existen |
| "Switches de acceso" | Bridges KVM (dispositivos Linux tipo `virbr-*`) |
| Hosts finales | VMs KVM con diferentes sistemas operativos |

### 4.3 Decisiones de simplificación documentadas

1. **No hay HA de firewall** — una sola instancia pfSense. Si cae, el lab entero cae.
2. **No hay redundancia de enlaces** — los bridges KVM son enlaces lógicos únicos.
3. **No hay hierarquía core/distribución/acceso** — topología plana de una capa lógica.
4. **No hay port-channels** — no aplica en virtualización KVM básica.
5. **No hay STP/RSTP** — los bridges KVM no corren spanning-tree.
6. **No hay protocolos de routing dinámico** — solo rutas estáticas / directas en pfSense.
7. **No hay VLAN tagging real** — cada zona es un bridge independiente; las "VLAN IDs" que aparecen en la documentación son identificadores descriptivos, no configuración 802.1Q real.

---

## 5. Comparación lado a lado

```
         ARQUITECTURA FÍSICA REAL                 FORTRESS-AI (simulación)
         =========================                ==========================

         ┌──────────────┐                         ┌──────────────┐
         │   Internet   │                         │   Internet   │
         │     (ISP)    │                         │ (router casa)│
         └──────┬───────┘                         └──────┬───────┘
                │                                        │ NAT
         ┌──────▼───────┐                                │
         │ Router borde │  Cisco ASR                     │ (no existe)
         └──────┬───────┘                                │
                │ port-channel                           │
         ┌──────▼───────┐                         ┌──────▼───────┐
         │ Firewall HA  │  Firepower par          │   pfSense    │  VM única
         └──────┬───────┘                         └──────┬───────┘
                │ port-channel                           │
         ┌──────▼───────┐                                │
         │  Core L3 HA  │  Catalyst 9500                 │ (integrado en
         └─┬────┬────┬──┘                                │  pfSense)
           │    │    │                                   │
        ┌──▼─┐┌─▼──┐┌▼───┐                               │
        │Dist││Dist││Dist│  Catalyst 9400                │ (no existe)
        └─┬──┘└─┬──┘└─┬──┘                               │
          │    │    │                              ┌─────┴─────┐
        ┌─▼─┐┌─▼─┐┌─▼─┐                           │  bridges  │
        │Acc││Acc││Acc│  Catalyst 9300             │    KVM    │  (L2 virtual)
        └─┬─┘└─┬─┘└─┬─┘                            └─────┬─────┘
          │    │    │                                    │
        [PCs][Srv][SOC]                                [VMs]
```

---

## 6. Tabla de equivalencias

Relación uno-a-uno entre elementos de arquitectura real y su equivalente en la simulación:

| Elemento físico real | Equivalente en FORTRESS-AI | Nivel de fidelidad |
|---|---|---|
| Router de borde (ASR 1000) | Router doméstico del host + NAT pfSense WAN | Bajo — comportamiento básico de NAT saliente |
| Firewall en HA (Firepower par) | pfSense CE 2.7 standalone | Medio — cumple función firewall/router, sin redundancia |
| Switch core L3 (Catalyst 9500) | Tabla de routing de pfSense | Medio — hace routing inter-VLAN lógico |
| Switches de distribución (9400) | No hay equivalente | N/A |
| Switches de acceso (2960/9300) | Bridges KVM (`virbr-*`) | Alto a nivel L2 — hacen forwarding de tramas |
| VLANs 802.1Q | Redes KVM aisladas sin tagging | Bajo — mismo resultado lógico (aislamiento), sin protocolo real |
| Port-channels LACP | N/A | No aplica |
| HSRP / VRRP | N/A | No aplica |
| OSPF / EIGRP | Rutas directas/estáticas pfSense | Bajo — no hay protocolo de routing dinámico |
| 802.1X NAC | No implementado | No aplica |
| TACACS+ / RADIUS | AAA nativo de pfSense + Active Directory | Medio — cumple función AAA básica |
| Out-of-band management | Zona Management dentro del mismo plano | Bajo — solo separación lógica |
| Site-to-site VPN | VPN pfSense → AWS Free Tier | Alto — es VPN real funcional |
| WAF en DMZ | ModSecurity sobre nginx en VM | Alto — es WAF real funcional |
| DMZ | Zona lógica aislada por reglas pfSense | Alto — comportamiento equivalente |
| SIEM (Splunk ES / QRadar) | Security Onion + Wazuh + Splunk Free | Alto — herramientas open source reales |

---

## 7. Lo que SÍ se simula

Elementos con **alta fidelidad** respecto a un entorno real — el comportamiento es equivalente, solo cambia la escala y el soporte físico.

### 7.1 Comportamiento de red
- Aislamiento entre zonas (funciona igual que segmentación L2 física)
- Reglas de firewall L3/L4 (idénticas a las de Firepower o Palo Alto en sintaxis)
- NAT saliente y port forwarding
- Resolución DNS interna con AD DNS
- DHCP por zona
- Filtrado por IP, puerto, protocolo

### 7.2 Comportamiento de aplicaciones
- Active Directory real con GPO, usuarios, grupos, SPNs
- Protocolos Kerberos, LDAP, SMB, RDP funcionando idéntico
- Servicios web con nginx / IIS sobre sistemas operativos reales
- Bases de datos PostgreSQL con datos simulados de clientes
- Vulnerabilidades reales en Metasploitable 3 y Windows 7

### 7.3 Comportamiento de seguridad
- Reglas Snort/Suricata en Security Onion (idénticas a producción)
- Agentes Wazuh reales con políticas de monitoreo
- Splunk con búsquedas SPL reales
- Logs Windows Event Log, syslog, Sysmon — formato idéntico a producción
- Detección de técnicas MITRE ATT&CK reales

### 7.4 Comportamiento de ataques
- Herramientas ofensivas reales (Metasploit, Impacket, BloodHound)
- Técnicas MITRE reales: Kerberoasting, Pass-the-Hash, DCSync, Golden Ticket
- CVEs reales explotados (EternalBlue CVE-2017-0144, etc.)
- Lateral movement real entre VMs comprometidas

### 7.5 Comportamiento forense
- Artefactos reales en memoria, disco, registro, logs
- Timeline reconstrucción con Volatility, Autopsy, Plaso
- Cadena de custodia con hashes criptográficos reales
- IOC extracción y documentación formato estándar

**Conclusión del punto 7:** en términos de valor educativo y demostrable, el 80% del proyecto tiene fidelidad alta. Los escenarios de ataque-detección-forense son **indistinguibles** de un ejercicio real excepto por el throughput y la escala.

---

## 8. Lo que NO se simula

Elementos deliberadamente fuera de alcance. Esta sección es la **declaración honesta de limitaciones**.

### 8.1 Infraestructura física
- No hay hardware Cisco físico (router, switch, firewall)
- No hay rack, cableado estructurado, patch panels, fibra
- No hay UPS, generadores, sistemas de climatización
- No hay sala de telecomunicaciones física

### 8.2 Redundancia y alta disponibilidad
- No hay par HA de firewall
- No hay switches redundantes en ninguna capa
- No hay enlaces duplicados con port-channels
- No hay dual-homed Internet
- No hay sitio DRP geográficamente separado

### 8.3 Protocolos avanzados de red
- No hay STP / RSTP / MST
- No hay HSRP / VRRP / GLBP
- No hay OSPF, EIGRP, IS-IS en operación
- No hay BGP con ISPs
- No hay 802.1Q trunking real (las VLAN IDs son descriptivas)
- No hay 802.1X NAC
- No hay DHCP Snooping, DAI, IP Source Guard

### 8.4 Operación física y logística
- No hay throughput real de carga empresarial (800 empleados concurrentes)
- No hay latencia de WAN real entre sitios
- No hay capacidad realista de cómputo (el lab está limitado por 32 GB RAM)
- No hay políticas de parcheo en masa en cientos de endpoints

### 8.5 Integración con sistemas bancarios específicos
- No hay integración real con SPEI (aunque el contexto regulatorio se documenta)
- No hay sistemas core bancario (Temenos, Finacle, etc.) — se simulan con apps web genéricas
- No hay conexión a redes interbancarias
- No hay cumplimiento certificado de PCI-DSS (se toma como referencia, no como auditoría)

---

## 9. Justificación académica y profesional

### 9.1 Por qué estas decisiones son válidas académicamente

El programa de Ingeniería en Redes Inteligentes y Ciberseguridad de UTVT exige cubrir 16 materias a través del proyecto integrador. De estas 16 materias:

- **3 materias** se relacionan directamente con infraestructura física (Administración de Infraestructura, Data Centers, Redes Empresariales)
- **13 materias** se enfocan en capas superiores: seguridad, operaciones, automatización, IA, gestión, forense

**FORTRESS-AI cubre las 16 materias** porque las 3 de infraestructura física se satisfacen con el diseño lógico documentado (este documento demuestra conocimiento del modelo real, que es lo que se evalúa), mientras que las 13 restantes se cubren con la ejecución práctica real del lab.

Un proyecto que hubiera priorizado infraestructura física (por ejemplo, con 3 switches Cisco prestados) habría cubierto 3 materias con más profundidad pero habría sacrificado la ejecución de las 13 restantes por falta de tiempo y recursos.

### 9.2 Por qué estas decisiones son válidas profesionalmente

En el mercado laboral mexicano de ciberseguridad (Scitum, IQsec, Totalsec, KIO, etc.):

- **Las posiciones de red engineer** requieren experiencia hands-on en Cisco IOS, CCNA/CCNP, gestión de infraestructura física. Este NO es el roadmap del autor.
- **Las posiciones de pentester / SOC analyst / consultor seguridad** requieren comprensión de arquitecturas de red (para pivotear, segmentar ataques, entender logs) pero NO requieren experiencia configurando STP en un Catalyst 9300. Este SÍ es el roadmap del autor.

El perfil objetivo del autor tras graduación es **pentester / consultor de seguridad ofensiva**, alineado con las certificaciones en ruta: eJPT → Security+ → CRTO → OSCP. Ninguna de estas certificaciones exige experiencia con hardware Cisco físico.

### 9.3 Por qué es honesto declararlo así

Un proyecto que simulara ser "infraestructura completa de un banco" sin declarar sus limitaciones estaría cometiendo dos errores:

1. **Error técnico** — el proyecto no sobreviviría al escrutinio de cualquier arquitecto senior de redes empresariales.
2. **Error ético** — vendería expectativas que no se cumplen.

Declarar la brecha abiertamente:

1. Construye confianza técnica ante evaluadores y reclutadores
2. Demuestra madurez profesional
3. Invita al diálogo técnico en vez de la defensa
4. Prepara al autor para responder preguntas difíciles con seguridad

---

## 10. Respuesta preparada para preguntas frecuentes

Esta sección provee respuestas listas a las preguntas más probables de profesores, reclutadores y revisores técnicos.

### P1: "¿Por qué no hay switches físicos? ¿No es eso una red empresarial?"

> FORTRESS-AI es una simulación lógica del plano de control de la red, no del plano físico. En un despliegue real de BNF se implementaría arquitectura jerárquica de tres capas (core-distribución-acceso) con Cisco Catalyst 9500/9400/9300, port-channels LACP en malla parcial y par de firewalls en HA. Ese diseño físico está documentado en la sección 3 del documento de reality gap (doc 03). La simulación en KVM emula el comportamiento lógico suficiente para ejecutar los escenarios de ataque, detección y forense del proyecto, que son los objetivos académicos centrales.

### P2: "¿Cómo manejarías redundancia en producción real?"

> En producción implementaría pfSense con CARP en par activo-pasivo para el firewall, VSS o StackWise en el core Catalyst 9500, port-channels LACP con enlaces duplicados en cada capa, HSRP para gateway redundante con priorización por zona, y conexiones dual-homed a dos ISPs distintos con BGP. Adicionalmente, un sitio DRP en región separada con replicación continua de servicios críticos.

### P3: "¿Por qué decís que hay VLANs si usás bridges KVM?"

> Las VLAN IDs en mi documentación son identificadores descriptivos que facilitan el mapeo mental al diseño físico real. A nivel implementación, cada zona está aislada mediante un bridge KVM independiente, lo cual logra el mismo objetivo de segmentación L2 que una VLAN 802.1Q, pero sin el protocolo de tagging. La decisión se documenta como simplificación en el documento 02. En una migración a Open vSwitch con VLAN tagging real, la configuración de pfSense sería sub-interfaces em0.10, em0.20, etc.

### P4: "¿No se ve poco profesional no tener HA del firewall?"

> Es una decisión consciente de scope. El objetivo del proyecto es demostrar capacidades de ciberseguridad (ofensiva, defensiva, forense, IA), no capacidades de alta disponibilidad de red. Implementar HA requeriría duplicar VMs críticas y configurar CARP + pfsync, lo cual consumiría aproximadamente una semana sin agregar valor a los objetivos centrales. En producción sí sería obligatorio; aquí se documenta como fuera de alcance justificado.

### P5: "¿Qué aprendiste de infraestructura real en este proyecto?"

> Aprendí a diseñar una arquitectura lógica segmentada y documentarla a nivel consultoría, lo cual incluye conocer la arquitectura física de referencia contra la cual se simplifica. Puedo explicar en detalle el modelo jerárquico Cisco, los protocolos involucrados (STP, HSRP, LACP, OSPF), y las decisiones que tomaría si BNF fuera real. Mi experiencia hands-on con hardware Cisco se construye paralelamente a través del camino formal de CCNA, no a través de FORTRESS-AI.

### P6: "¿Cómo defenderías este proyecto ante un arquitecto de red senior?"

> Le diría que FORTRESS-AI no compite con su experiencia en arquitectura física. Complementa ese conocimiento con un dominio distinto: operaciones de seguridad, detección de intrusos, forense post-incidente y automatización con IA. En una consultora real, el arquitecto de red y el pentester colaboran pero tienen perfiles distintos. Mi proyecto prepara el segundo perfil, no el primero.

---

## 11. Roadmap de extensión a producción

Esta sección describe **qué agregaría** si FORTRESS-AI evolucionara más allá del alcance académico.

### 11.1 Extensión física (hipotética — no planeada en el alcance actual)

Si se dispusiera de hardware Cisco físico, se montaría un lab físico complementario con:

- 1x router Cisco ISR (1900/2900 series) — borde simulado
- 1x switch multicapa Cisco (3560 o 3850) — core L3
- 2x switches Cisco 2960 — acceso por zona (LAN + Servers)
- Conexiones con cables consola + ethernet
- Configuración Cisco IOS documentada en `scripts/cisco-configs/`
- Diagrama físico en draw.io separado

Este lab físico se marcaría como "Extended Lab — Physical Implementation" y se usaría para practicar CCNA. **No es parte de FORTRESS-AI como tal.**

### 11.2 Extensiones virtuales futuras (posibles)

- **Migración a VLAN tagging real con Open vSwitch** — si el autor obtiene CCNA durante el proyecto
- **Implementación de pfSense HA con CARP** — si se consiguen recursos adicionales (RAM extra, segundo servidor)
- **Integración con EVE-NG o GNS3** — para simulación de equipos Cisco virtualizados dentro del mismo lab
- **Conexión VPN site-to-site a AWS real** — ya planeado en Fase 3

### 11.3 Extensiones de seguridad (dentro del alcance)

Estas SÍ están planeadas en el alcance original:

- Motor IA para clasificación de alertas SIEM (Fase 4)
- Generador automático de reportes pentest (Fase 4)
- Dashboard Power BI sobre métricas SOC (Fase 2)

---

## 12. Control de versiones

| Versión | Fecha | Autor | Cambios |
|---|---|---|---|
| 1.0 | 2026-04-21 | Ángel Damián Malvaiz González | Emisión inicial del reality gap |

---

## Anexo A — Referencias de arquitectura empresarial

- Cisco Enterprise Architecture Framework (Campus Design)
- Cisco SAFE Reference Architecture
- NIST SP 800-41 Rev. 1 — Guidelines on Firewalls
- NIST SP 800-125 — Guide to Security for Full Virtualization Technologies
- CIS Controls v8 — Network Infrastructure Management
- PCI-DSS v4.0 — Network Segmentation

## Anexo B — Glosario

| Término | Definición |
|---|---|
| **HA** | High Availability — redundancia de componentes para continuidad |
| **Port-channel** | Agregación de múltiples enlaces físicos en uno lógico (LACP) |
| **STP / RSTP / MST** | Spanning Tree Protocol — previene loops en redes L2 |
| **HSRP / VRRP** | First-hop redundancy — gateway redundante |
| **VSS / StackWise** | Tecnologías Cisco para switches en par redundante |
| **LACP** | Link Aggregation Control Protocol — agregación dinámica |
| **BGP** | Border Gateway Protocol — routing externo con ISPs |
| **NAC** | Network Access Control — verificación de compliance por puerto |
| **DRP** | Disaster Recovery Plan |
| **OOB** | Out-of-band management — red física separada para administración |

---

> *"Un proyecto que declara sus limitaciones con honestidad técnica es más confiable que uno que pretende cubrirlas con ambigüedad."*

**Documento controlado — FORTRESS-AI · Architecture Reality Gap v1.0 · 21 de abril de 2026**
