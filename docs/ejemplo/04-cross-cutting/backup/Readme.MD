# Estándar Corporativo de Backup para Entornos Industriales

> **Versión:** 1.0  
> **Fecha:** 2025-02-19  
> **Autor:** Arquitectura IT

---

## 📦 Contenido del Paquete

Este paquete contiene el **estándar corporativo completo de backup** para plantas industriales edge, incluyendo documentación técnica, runbooks operacionales y diagramas arquitectónicos.

### **Documento Principal (1,102 líneas)**
- `00-backup-standard.md` — Estándar completo con 14 secciones

### **Runbooks Operacionales (483 líneas)**
1. `02-vm-backup-runbook.md` — Backup de VMs (Hyper-V, VMware, Nutanix)
2. `03-sql-backup-runbook.md` — Backup de SQL Server (Full, Differential, Transaction Log)
3. `04-network-backup-runbook.md` — Backup de Cisco Switches y Fortinet Firewalls
4. `05-restore-procedure-runbook.md` — Procedimientos de restore (VMs, DBs, Files)
5. `06-troubleshooting-runbook.md` — Troubleshooting común (jobs failed, agents offline, dedup low)

### **Diagramas C4 Mermaid (219 líneas)**
1. `c4-level1-context.mermaid` — Vista de contexto (stakeholders, sistemas externos)
2. `c4-level2-container.mermaid` — Vista de contenedores (componentes internos)
3. `network-security.mermaid` — Seguridad de red (firewall, segregación IT/OT)

**Total:** 1,804 líneas de documentación técnica

---

## 🎯 Características Principales del Estándar

### ✅ **Agnóstico de Fabricante**
Plantas pueden seleccionar solución de backup que prefieran (Veeam, Commvault, Veritas, etc.) siempre que cumplan requisitos técnicos mínimos definidos.

### ✅ **4 Tiers de Criticidad**
| Tier | RPO | RTO | Frecuencia | Ejemplo |
|------|-----|-----|------------|---------|
| **Platino** | < 4h | < 4h | Cada 4h o continuo | SAP Production, SCADA |
| **Oro** | < 24h | < 24h | Diario | File servers, DBs Tier 2 |
| **Plata** | < 7d | < 72h | Semanal | Apps no críticas |
| **Bronce** | < 30d | < 2 sem | Mensual | Archival, históricos |

### ✅ **Principio 3-2-1 Obligatorio**
- **3 copias:** 1 primaria + 2 backups
- **2 medios:** Disco + Tape/Cloud
- **1 off-site:** >50km o Azure

### ✅ **Seguridad IT/OT (Modelo Purdue)**
- Segregación de red OT (Level 2-3) ↔ IT (Level 4)
- Firewall industrial con mínimos puertos (TCP 10000-10010)
- Agentes con permisos mínimos (least privilege)
- Encriptación AES-256 at rest, TLS 1.3 in transit

### ✅ **Deduplicación Obligatoria**
- Inline o post-process
- Ratio esperado: 10-20:1
- Ahorro típico: 90-95% de storage

### ✅ **Workloads Soportados**
- Sistemas Operativos: Windows Server, Linux (RHEL, Ubuntu, SLES)
- Hipervisores Tipo 2: Hyper-V, VMware vSphere, Nutanix AHV
- Bases de Datos: SQL Server, Oracle, MySQL, PostgreSQL
- Network Devices: Cisco Catalyst 9xxx, Fortinet FortiGate
- Aplicaciones: SAP (centralizado)

### ✅ **Storage Targets Flexibles**
- **Primary:** NAS (NFS, SMB), SAN (iSCSI, FC), discos locales (RAID)
- **Secondary:** Tape Libraries (LTO-8+), NAS remoto
- **Tertiary (opcional):** Azure Blob Storage (Cool $0.01/GB, Archive $0.001/GB)

### ✅ **Compliance**
- GDPR (data residency, right to deletion)
- ISO 27001 (encryption, access control)
- SOX (financial data 7 años retention)
- ISA-95 (Purdue model compliance)

---

## 📊 Secciones del Documento Principal

| Sección | Contenido |
|---------|-----------|
| **1. Executive Summary** | Propósito, alcance, principios 3-2-1 |
| **2. Clasificación de Datos** | Tiers Platino/Oro/Plata/Bronce, RPO/RTO, RACI |
| **3. Arquitectura** | Modelo de referencia, componentes, flujos, deduplicación |
| **4. Requisitos por Workload** | Windows, Linux, Hyper-V, VMware, Nutanix, SQL, Oracle, MySQL, Cisco, Fortinet, SAP |
| **5. Requisitos Técnicos** | 10 requisitos funcionales obligatorios, NFRs, seguridad, Azure opcional |
| **6. Seguridad y Compliance** | Modelo Purdue, puertos mínimos, encriptación, compliance |
| **7. Monitorización** | KPIs, alerting (P1-P4), dashboards, integración Prometheus/Zabbix/Splunk |
| **8. Procedimientos Operacionales** | Workflow backup, restore, DR testing |
| **9. RACI** | Responsabilidades por equipo (Virtualización, BBDD, SAP, Comunicaciones, etc.) |
| **10. Costes** | CAPEX/OPEX por tamaño planta (Small/Medium/Large), TCO 5 años, optimización |
| **11. Roadmap** | Fases de rollout (Piloto → Regional → Global), criterios de éxito |
| **12. Apéndices** | Glosario, referencias, documentos relacionados |

---

## 🔧 Runbooks Operacionales

### **Runbook 02: VMs**
- Configuración jobs Hyper-V y VMware
- CBT (Changed Block Tracking) setup
- Application-aware processing
- Restore opciones (original, new location, instant recovery)

### **Runbook 03: SQL Server**
- Configurar recovery model (FULL vs SIMPLE)
- Backup Full + Differential + Transaction Log
- Point-in-Time Recovery (PITR)
- Individual table restore
- Monitoring queries (último backup por DB)

### **Runbook 04: Network Devices**
- Backup Cisco Switches (SSH, SCP, TFTP)
- Backup Fortinet FortiGate (CLI, API)
- Automatización con Oxidized
- Versionado en Git
- Retention: 30d daily, 90d pre/post change

### **Runbook 05: Restore**
- Pre-restore validation (ticket, approvals)
- Restore VM (3 opciones: original, new location, instant)
- Restore SQL (full DB, point-in-time, individual table)
- Restore files (file-level browse)
- Post-restore validation y documentación

### **Runbook 06: Troubleshooting**
- Diagnóstico backup job failed (snapshot, disk full, auth, timeout)
- Agent offline (Windows service, Linux systemd)
- Deduplication ratio bajo (causas, optimización)
- Backup muy lento (network/disk bottleneck, concurrency)
- Restore falla (chain broken, media offline, space, locked files)
- Escalación L1→L2→L3→Vendor

---

## 📐 Diagramas C4 Mermaid

### **Nivel 1 - Context Diagram** (68 líneas)
**Vista de alto nivel:**
- Red OT (SCADA, PLCs) con agentes
- Firewall industrial (segregación IT/OT)
- Red IT (VMs, DBs, File Servers, Network Devices)
- Backup Server central
- Storage targets (Primary disk, Secondary tape, Tertiary Azure)
- Monitoring (Prometheus/Zabbix/Grafana)
- Stakeholders (IT Ops, Data Owners, Compliance)

### **Nivel 2 - Container Diagram** (79 líneas)
**Vista de componentes internos del Backup Server:**
- **Agents:** Windows, Linux, VMware (agentless), SQL
- **Core Services:** Orchestrator, Catalog DB
- **Processing Engine:** Deduplication (10-20:1), Compression, Encryption (AES-256)
- **API Layer:** REST API (port 9398), Monitoring Exporter (Prometheus port 9100)
- **Storage Targets:** NAS (NFS/SMB), SAN (iSCSI/FC), Tape Library, NAS Remoto, Azure Blob

### **Nivel 3 - Network Security** (72 líneas)
**Vista de seguridad y flujos de red:**
- **OT Zone:** SCADA, PLCs (10.100.x.x)
- **Firewall Rules:** 
  - ✅ Allow TCP 10000-10010 (Backup data OT→IT)
  - ✅ Allow TCP 22 (SSH config backup, unidirectional)
  - ❌ Deny all other OT→IT
- **IT Zone:** VMs, DBs, File Servers (192.168.x.x)
- **Storage Network:** NAS (NFS 2049, SMB 445), SAN (iSCSI 3260)
- **Azure Private:** ExpressRoute → Blob Storage Private Endpoint (no Internet)
- **Management Network:** Monitoring, Jump Server

---

## 🚀 Implementación

### **Fase 1: Piloto (Mes 1-3)**
- 2 plantas (1 small, 1 medium)
- Validar todos workloads
- Lessons learned

### **Fase 2: Rollout Regional (Mes 4-9)**
- Europa: 8 plantas
- América: 5 plantas
- Asia-Pacífico: 3 plantas

### **Fase 3: Rollout Global (Mes 10-18)**
- Plantas restantes (30+)

### **Fase 4: Optimización (Mes 19-24)**
- Azure backup (opcional)
- Consolidación backup servers
- Self-service restore

---

## 💰 Costes Estimados

### **CAPEX por Planta:**
- **Small:** ~$56,000
- **Medium:** ~$135,000
- **Large:** ~$295,000

### **OPEX Anual:**
- **Small:** ~$40,000/año
- **Medium:** ~$56,000/año
- **Large:** ~$125,000/año

### **TCO 5 años:**
- **Small:** $256,000 ($5/TB/mes)
- **Medium:** $415,000 ($1.4/TB/mes)
- **Large:** $920,000 ($0.77/TB/mes)

**Optimización:** Deduplicación 15:1 ahorra 93% de storage (ROI inmediato).

---

## 👥 RACI

| Actividad | Arquitectura IT | Virtualización | BBDD | SAP | Comunicaciones | Industria | Security | IT Local |
|-----------|----------------|----------------|------|-----|----------------|-----------|----------|----------|
| **Definir estándar** | **R** A | C | C | C | C | C | C | I |
| **Selección solución** | C | **R** A | C | I | I | I | C | C |
| **Instalar backup server** | C | **R** A | C | I | I | I | C | C |
| **Configurar firewall** | C | I | I | I | **R** A | I | A | C |
| **Backup VMs** | C | **R** A | I | I | I | I | I | C |
| **Backup DBs** | I | C | **R** A | I | I | I | I | C |
| **Backup SAP** | I | I | C | **R** A | I | I | I | I |
| **Backup network** | I | I | I | I | **R** A | I | I | C |
| **Monitorización diaria** | I | C | C | C | C | I | I | **R** A |
| **Restores** | I | C | C | C | C | I | A | **R** A |

---

## 📚 Referencias

- **ISA-95 (Purdue Model):** https://www.isa.org/standards-and-publications/isa-standards/isa-standards-committees/isa95
- **NIST Cybersecurity Framework:** https://www.nist.gov/cyberframework
- **ISO 27001:2013:** Clauses A.12.3 (Backup), A.17.1 (DR)
- **3-2-1 Backup Rule:** https://www.backblaze.com/blog/the-3-2-1-backup-strategy/
- **Azure Blob Pricing:** https://azure.microsoft.com/pricing/details/storage/blobs/

---

## ✅ Checklist de Adopción

**Para que una planta sea compliant con este estándar:**

- [ ] Backup server instalado y configurado
- [ ] Clasificación de datos realizada (Platino/Oro/Plata/Bronce)
- [ ] Jobs de backup configurados para todos los tiers
- [ ] Primary target configurado (disk)
- [ ] Secondary target configurado (tape o remoto off-site)
- [ ] Tertiary target opcional (Azure) si aplica
- [ ] Principio 3-2-1 implementado
- [ ] Firewall rules aplicadas (mínimos puertos)
- [ ] Agentes instalados con permisos mínimos
- [ ] Deduplicación habilitada y validada (ratio >10:1)
- [ ] Encriptación AES-256 habilitada
- [ ] Monitoring integrado (Prometheus/Zabbix)
- [ ] Alerting configurado (PagerDuty para P1/P2)
- [ ] DR test ejecutado y documentado (trimestral para Platino)
- [ ] Runbooks operacionales entregados a IT Ops
- [ ] RACI validada con todos los equipos
- [ ] Documentación as-built completada

---

## 📞 Contactos

- **Estándar Owner:** CTO Office — cto@company.com
- **Technical Lead:** Arquitectura IT — architecture-team@company.com
- **Implementation Support:** Virtualización Team — virtualization@company.com
- **Security Approval:** CISO — security-team@company.com

---

**Notas Finales:**
- Este estándar debe revisarse **anualmente** o cuando cambien tecnologías/regulaciones
- Excepciones requieren aprobación formal de CTO + CISO
- Cada planta debe crear un "Backup Implementation Plan" específico que demuestre compliance con este estándar

---

**Aprobado por:**
- ✅ CTO
- ✅ CISO
- ✅ Infrastructure Director
- ✅ Compliance Officer

**Fecha de aprobación:** 2025-02-19
