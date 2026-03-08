# Estándar Corporativo de Backup para Entornos Industriales

> **Versión:** 1.0  
> **Fecha:** 2025-02-19  
> **Estado:** Approved  
> **Autor:** Arquitectura IT  
> **Propietario:** CTO Office  
> **Última Revisión:** 2025-02-19  
> **Ámbito:** Global — Todas las plantas industriales

---

## 1. Executive Summary

### 1.1 Propósito

Este documento establece el **estándar corporativo de backup y recuperación** para entornos industriales edge, garantizando protección consistente de datos críticos en infraestructuras heterogéneas distribuidas globalmente, cumpliendo con requisitos de compliance, seguridad (IT/OT segregation) y optimización de costes.

### 1.2 Alcance

**Aplica a:**
- Todas las plantas industriales (Europa, América, Asia-Pacífico)
- Entornos IT y OT (respetando modelo Purdue de segmentación)
- Infraestructura on-premise (edge) y cloud híbrido (Azure opcional)

**Sistemas protegidos:**
- Sistemas Operativos: Windows Server, Linux (RHEL, Ubuntu, SLES)
- Hipervisores Tipo 2: Hyper-V, VMware vSphere, Nutanix AHV
- Bases de Datos: Microsoft SQL Server, Oracle, MySQL/MariaDB, PostgreSQL
- Dispositivos de Red: Switches Cisco Catalyst 9xxx, Fortinet FortiGate 40F+
- Aplicaciones Empresariales: SAP ERP (centralizado), aplicaciones custom

**Exclusiones:**
- Endpoints de usuario (desktops, laptops) → Gestionados por política separada
- Datos transitorios (logs de aplicación < 7 días) → No requieren backup
- Entornos de desarrollo/testing → Backup opcional, fuera de SLA

### 1.3 Principios Fundamentales

#### Principio 3-2-1 (Regla de Oro)
**Obligatorio para datos Platino y Oro:**
- **3 copias** de datos: 1 primaria (producción) + 2 backups
- **2 tipos de medios** diferentes: Disco local/NAS + Tape/Cloud
- **1 copia off-site**: Datacenter alternativo o Azure Blob Storage

#### Agnosticismo de Fabricante
Las plantas pueden seleccionar solución de backup según presupuesto y requisitos técnicos, siempre que cumpla requisitos mínimos definidos en Sección 5.

#### Seguridad IT/OT
Respeto absoluto del modelo Purdue (ISA-95):
- Agentes de backup NO cruzan boundary IT/OT sin aprobación Security
- Comunicación a través de firewall con mínimos puertos (ver Sección 6)
- Least privilege: permisos mínimos para agentes

### 1.4 Objetivos de Negocio

| Objetivo | Métrica | Target |
|----------|---------|--------|
| **Disponibilidad de datos** | % de restores exitosos | > 99% |
| **Cumplimiento de RPO** | % backups dentro de ventana | > 99.5% |
| **Cumplimiento de RTO** | % restores dentro de SLA | > 95% |
| **Reducción de storage** | Ratio de deduplicación | > 15:1 (típico) |
| **Coste por TB protegido** | USD/TB/mes | < $50 (target) |

---

## 2. Clasificación de Datos y SLAs

### 2.1 Tiers de Criticidad

| Tier | Descripción | Ejemplos | % de Datos |
|------|-------------|----------|------------|
| **Platino** | Crítico — Impacto inmediato en producción | SAP Production, Databases Tier 1, SCADA/MES | 10% |
| **Oro** | Importante — Impacto en operaciones 24-48h | File Servers, Databases Tier 2, VMs críticas | 30% |
| **Plata** | Estándar — Operaciones pueden continuar temporalmente | Aplicaciones no críticas, Development DBs | 40% |
| **Bronce** | Archival — Datos históricos, compliance | Logs históricos, documentación antigua | 20% |

### 2.2 RPO (Recovery Point Objective)

| Tier | RPO | Frecuencia Backup Mínima | Retención Mínima |
|------|-----|-------------------------|------------------|
| **Platino** | < 4 horas | Continuous (CDP) o cada 4h | 30 días full + 90 días incremental |
| **Oro** | < 24 horas | Diario | 14 días full + 30 días incremental |
| **Plata** | < 7 días | Semanal | 7 días full |
| **Bronce** | < 30 días | Mensual | 12 meses |

**Nota:** RPO es el máximo de datos que podemos permitirnos perder. Platino < 4h significa que en caso de desastre, perdemos máximo las últimas 4 horas de cambios.

### 2.3 RTO (Recovery Time Objective)

| Tier | RTO | Ventana de Restore | Prioridad en DR |
|------|-----|-------------------|----------------|
| **Platino** | < 4 horas | 24×7 | P1 — Inmediato |
| **Oro** | < 24 horas | Business hours (extendible 24×7 si necesario) | P2 — Urgente |
| **Plata** | < 72 horas | Business hours | P3 — Normal |
| **Bronce** | < 2 semanas | Business hours | P4 — Best effort |

**Nota:** RTO es el tiempo máximo tolerable de inactividad. Platino < 4h significa que debemos poder restaurar en menos de 4 horas.

### 2.4 Matriz de Responsabilidades (Clasificación)

| Workload | Owner Clasificación | Aprueba Tier | Valida RPO/RTO |
|----------|-------------------|--------------|----------------|
| **SAP Production** | SAP Team | CTO + CFO | SAP Team + Industria |
| **Databases SQL** | BBDD Team | Database Manager | BBDD Team + Application Owner |
| **VMs (Hyper-V/VMware)** | Virtualización Team | Infrastructure Manager | VM Owner (app team) |
| **File Servers** | Sistemas Operativos | IT Manager | Department heads |
| **Network Devices** | Comunicaciones Team | Network Manager | Comunicaciones Team |

---

## 3. Arquitectura de Backup

### 3.1 Modelo de Referencia

```
┌──────────────────────────────────────────────────────────────────┐
│                    PLANTA INDUSTRIAL (EDGE)                      │
│                                                                  │
│  ┌─────────────────────────┐    ┌───────────────────────────┐  │
│  │    RED OT (Level 2-3)   │    │   RED IT (Level 4-5)      │  │
│  │                         │    │                           │  │
│  │  • SCADA/MES            │    │  • VMs (Hyper-V/VMware)   │  │
│  │  • PLCs                 │    │  • File Servers           │  │
│  │  • HMIs                 │    │  • Databases              │  │
│  │                         │    │  • Switches/Firewalls     │  │
│  │  [Backup Agent OT]      │    │  [Backup Agents IT]       │  │
│  └──────────┬──────────────┘    └─────────┬─────────────────┘  │
│             │                              │                    │
│             │    Firewall (minimal ports)  │                    │
│             └──────────────┬───────────────┘                    │
│                            │                                    │
│                   ┌────────▼────────┐                          │
│                   │ Backup Server   │                          │
│                   │ (IT Zone)       │                          │
│                   │                 │                          │
│                   │ • Orchestration │                          │
│                   │ • Deduplication │                          │
│                   │ • Encryption    │                          │
│                   │ • Monitoring    │                          │
│                   └────────┬────────┘                          │
│                            │                                    │
│            ┌───────────────┼───────────────┐                  │
│            │               │               │                  │
│    ┌───────▼──────┐ ┌──────▼─────┐ ┌──────▼──────┐          │
│    │ Primary      │ │ Secondary  │ │ Tertiary    │          │
│    │ Target       │ │ Target     │ │ Target      │          │
│    │              │ │            │ │             │          │
│    │ • Disk (NAS) │ │ • Tape     │ │ • Off-site  │          │
│    │ • iSCSI/FC   │ │ • Disk     │ │ • Cloud     │          │
│    │ • Local      │ │ (remote)   │ │ (Azure)     │          │
│    │              │ │            │ │             │          │
│    │ Retention:   │ │ Retention: │ │ Retention:  │          │
│    │ 7-30 días    │ │ 30-90 días │ │ 1-7 años    │          │
│    └──────────────┘ └────────────┘ └─────────────┘          │
└──────────────────────────────────────────────────────────────────┘
```

### 3.2 Componentes Arquitectónicos

#### 3.2.1 Backup Agents (por workload)

| Workload | Tipo de Agent | Instalación | Permisos Mínimos Requeridos |
|----------|--------------|-------------|----------------------------|
| **Windows VMs** | Guest-level agent | Dentro del VM | Local Administrator |
| **Linux VMs** | Guest-level agent | Dentro del VM | root o sudo backup-user |
| **Hyper-V** | Hypervisor-level | En host Hyper-V | Hyper-V Administrator |
| **VMware** | Agentless (via vCenter) | N/A (usa API) | Read-only + backup role en vCenter |
| **Nutanix** | Agentless (via Prism) | N/A (usa API) | Read-only + backup role en Prism |
| **SQL Server** | DB-specific agent | En SQL server | sysadmin role (o db_backupoperator + db_datareader) |
| **Cisco Switches** | Network device backup | En backup server | EXEC level 15 (enable) via SSH |
| **Fortinet FW** | Network device backup | En backup server | admin (read-write) via HTTPS API |
| **SAP** | SAP-specific agent | En SAP app servers | SAP_BASIS user con permisos BR*TOOLS |

**Principio:** Usar siempre el método con **menos privilegios** posible. Ejemplo: para VMware, preferir agentless (vCenter API) antes que agent dentro de cada VM.

#### 3.2.2 Backup Server (Central)

**Ubicación:** Red IT de cada planta (1 servidor por planta típicamente)

**Requisitos mínimos:**
- CPU: 8 cores (16 cores recomendado para plantas >50 VMs)
- RAM: 32 GB (64 GB recomendado)
- OS: Windows Server 2022 o Linux (RHEL 8+, Ubuntu 22.04 LTS)
- Network: 2× 10 GbE NICs (bonding para redundancia)
- Storage: 
  - OS disk: 500 GB SSD
  - Deduplication cache: 1-2 TB NVMe SSD (ratio: 1 TB cache por 50 TB de datos protegidos)

**Funciones:**
- Orquestación de backup jobs
- Deduplicación en línea (before write to target)
- Encriptación AES-256 (at rest en targets)
- Catálogo de backups (metadata database)
- Reporting y alerting
- API REST para integración con monitoring

#### 3.2.3 Backup Targets (Storage)

**Primary Target (Disco — Fast recovery):**
- **Tecnologías soportadas:** NAS (NFS, SMB), SAN (iSCSI, FC), DAS (discos locales RAID)
- **Capacidad típica:** 20-100 TB por planta (depende de tamaño)
- **Retención:** 7-30 días (suficiente para recovery rápido)
- **Performance:** > 500 MB/s lectura/escritura
- **Deduplicación:** Obligatoria (target o inline en backup server)

**Secondary Target (Disco o Tape — Mid-term retention):**
- **Tecnologías soportadas:** NAS remoto (otro site), Tape Library (LTO-8+), Object Storage
- **Ubicación:** Off-site (mínimo 50 km de primary) o datacenter alternativo
- **Retención:** 30-90 días
- **Performance:** Best effort (no crítico para RTO)

**Tertiary Target (Cloud — Long-term archival - OPCIONAL):**
- **Tecnología:** Azure Blob Storage (Cool o Archive tier)
- **Casos de uso:**
  - Plantas sin capacidad para secondary target on-premise
  - Compliance (retención >1 año)
  - DR cross-región (failover a cloud)
- **Costes:** ~$0.01/GB/mes (Cool) o $0.001/GB/mes (Archive)
- **Requisitos:** ExpressRoute o VPN site-to-site (NO Internet público)

### 3.3 Flujos de Backup

#### Backup Full (Baseline)
```
Source Data → Backup Agent → [Change Block Tracking] → Backup Server 
    → [Deduplication + Compression] → [Encryption AES-256] 
    → Primary Target (Disk)
    → [Copy job] → Secondary Target (Tape/Remote Disk)
    → [Optional] → Tertiary Target (Azure)
```

**Frecuencia:** 
- Platino/Oro: 1× semanal (típicamente sábado noche)
- Plata: 1× mensual
- Bronce: 1× trimestral

#### Backup Incremental (Cambios desde último full/incremental)
```
Source Data → Backup Agent → [Changed blocks only] → Backup Server 
    → [Deduplication] → Primary Target (append to chain)
```

**Frecuencia:**
- Platino: Cada 4 horas o continuous (CDP)
- Oro: Diario (noche)
- Plata: Semanal
- Bronce: No aplica (solo full mensual)

#### Backup Diferencial (Cambios desde último full)
```
Source Data → [Changed blocks since last full] → Primary Target
```

**Uso:** Alternativa a incremental cuando se prefiere restore más simple (solo full + último diferencial, sin cadena)

### 3.4 Deduplicación

**Obligatoria para:**
- Todos los tiers (Platino, Oro, Plata, Bronce)
- Primary y secondary targets

**Métodos soportados:**
- **Inline (source-side):** Deduplicación en backup server antes de escribir a target (preferido — reduce tráfico red)
- **Post-process (target-side):** Deduplicación en storage target (NAS/SAN con dedup nativa)

**Ratios esperados:**
- VMs Windows: 15:1 a 25:1 (típico: 20:1)
- VMs Linux: 10:1 a 20:1
- File Servers: 5:1 a 15:1 (depende de tipo de archivos)
- Databases: 3:1 a 8:1 (menos dedup por naturaleza binaria)

**Verificación:** Ratio de deduplicación debe monitorizarse semanalmente. Si < 5:1, investigar causas (datos encriptados pre-backup, archivos únicos).

---

## 4. Requisitos por Tipo de Workload

### 4.1 Sistemas Operativos (Windows)

**Métodos de backup soportados:**
1. **Image-level (preferido):** Backup completo del disco/volumen
   - Permite BMR (Bare Metal Recovery)
   - Incluye OS, aplicaciones, configuración
   - Uso: Windows Server VMs en Hyper-V/VMware

2. **File-level:** Backup selectivo de carpetas
   - Uso: File servers, cuando solo necesitamos datos (no OS)

**Exclusiones por defecto:**
- `C:\Windows\Temp\`
- `C:\$Recycle.Bin\`
- `C:\pagefile.sys`, `C:\hiberfil.sys`
- Logs de aplicación > 30 días (si no son críticos)

**VSS (Volume Shadow Copy Service):**
- Obligatorio para consistency de backups en caliente
- Solución de backup debe integrar con VSS
- Verificar VSS writers antes de cada backup

**Permisos agent:**
- Cuenta de servicio: `DOMAIN\BackupAgent` (mínimo)
- Permisos: Local Administrator en servers a proteger
- Alternativa (más seguro): Usar Backup Operators group + permisos específicos VSS

### 4.2 Sistemas Operativos (Linux)

**Métodos de backup soportados:**
1. **Image-level:** Backup de particiones LVM o discos completos
2. **File-level:** rsync, tar, o agent propietario

**Directorios críticos a incluir:**
- `/etc/` (configuración)
- `/var/` (datos variables, logs críticos)
- `/home/` (usuarios)
- `/opt/` (aplicaciones custom)
- `/root/` (configuración root)

**Exclusiones por defecto:**
- `/tmp/`, `/var/tmp/`
- `/proc/`, `/sys/`, `/dev/` (pseudo-filesystems)
- `/mnt/`, `/media/` (montajes temporales)

**Permisos agent:**
- Usuario: `backup` (crear usuario dedicado)
- Sudo: `backup ALL=(ALL) NOPASSWD: /usr/bin/rsync, /usr/bin/tar, /sbin/lvm`
- Acceso SSH: Key-based authentication (NO passwords)

**Consistency:**
- Para VMs: snapshot a nivel hypervisor (VMware/Hyper-V)
- Para físicos: LVM snapshots o filesystem freeze (xfs_freeze, fsfreeze)

### 4.3 Hipervisores (Hyper-V)

**Método:** Backup a nivel hypervisor (agentless preferido)

**Integración VSS:**
- Usar Hyper-V VSS writer para quiescing
- Backups application-consistent (no crash-consistent)

**Configuración:**
```powershell
# Verificar Integration Services en VMs
Get-VM | Get-VMIntegrationService -Name "VSS"

# Habilitar VSS en todas las VMs
Get-VM | Get-VMIntegrationService -Name "VSS" | Enable-VMIntegrationService
```

**Best practices:**
- Backup en ventana de bajo uso (noche/fines de semana)
- No más de 4 VMs simultáneas por host (evitar contención I/O)
- Verificar checkpoints no quedan huérfanos post-backup

**Permisos:**
- Cuenta de servicio: Miembro de "Hyper-V Administrators" en host
- Permisos WMI: Read sobre namespace `root\virtualization\v2`

### 4.4 Hipervisores (VMware vSphere)

**Método:** Backup agentless via vCenter API (VADP - vStorage APIs for Data Protection)

**Changed Block Tracking (CBT):**
- Obligatorio habilitar en todas las VMs para incrementales eficientes
- Verificar CBT no se rompe post-backup

```bash
# Verificar CBT habilitado
vim-cmd vmsvc/get.config <vmid> | grep ctkEnabled
# Debe mostrar: ctkEnabled = true
```

**VMware Tools:**
- Obligatorio instalado y actualizado en guest OS
- Permite quiescing (consistency de filesystem y aplicaciones)

**Permisos vCenter:**
- Role: "Backup Operator" (custom role)
- Privilegios mínimos:
  - Datastore > Browse datastore
  - Global > Licenses
  - Virtual Machine > Snapshot management > Create snapshot / Remove snapshot
  - Virtual Machine > Provisioning > Allow disk access

**Limitaciones:**
- Snapshots no deben mantenerse > 24h (afectan performance)
- Solución de backup debe consolidar snapshots automáticamente post-job

### 4.5 Hipervisores (Nutanix AHV)

**Método:** Backup via Nutanix Prism API + Protection Domains

**Protection Domains:**
- Agrupar VMs por tier (Platino, Oro, Plata)
- Schedule de snapshots configurable en Prism

**Integración con solución de backup:**
- API REST de Nutanix v3
- Usar snapshots de Prism como source para backup externo

**Permisos:**
- User role: "Backup Admin" (custom en Prism)
- Permisos: VM viewer + snapshot management

### 4.6 Bases de Datos (Microsoft SQL Server)

**Método:** Backup nativo SQL + integración con solución de backup

**Tipos de backup:**
1. **Full database backup:** 1× semanal (Platino/Oro)
2. **Differential backup:** 1× diario (Platino/Oro)
3. **Transaction log backup:** Cada 15-60 min (Platino), cada 4-6h (Oro)

**Configuración SQL Server:**
```sql
-- Habilitar backup compression (ahorra espacio y tiempo)
EXEC sp_configure 'backup compression default', 1;
RECONFIGURE;

-- Verificar recovery model (debe ser FULL para Platino/Oro)
SELECT name, recovery_model_desc FROM sys.databases;
-- Platino/Oro: FULL
-- Plata: SIMPLE (no requiere log backups)
```

**Integración con backup solution:**
- Opción 1: Native SQL backup to disk → backup solution copia archivos .bak
- Opción 2: Backup solution usa VDI (Virtual Device Interface) para backup directo

**Permisos:**
- Cuenta de servicio: `DOMAIN\BackupSQL`
- Role: `db_backupoperator` en cada database + `sysadmin` (si necesita RESTORE)
- Alternativa menos privilegiada: Crear login con `GRANT BACKUP DATABASE, BACKUP LOG`

**Verificación:**
```sql
-- Test restore (ejecutar semanalmente en ambiente non-prod)
RESTORE VERIFYONLY FROM DISK = 'C:\Backups\Database.bak';
```

### 4.7 Bases de Datos (Oracle, MySQL, PostgreSQL)

**Oracle:**
- RMAN (Recovery Manager) para backups
- Integración con backup solution via RMAN API
- Archivelogs obligatorios para Platino/Oro (permite PITR - Point-in-Time Recovery)

**MySQL/MariaDB:**
- mysqldump para backups lógicos (pequeñas DBs)
- Percona XtraBackup para backups físicos (grandes DBs, hot backup sin locks)
- Binary logs habilitados para PITR

**PostgreSQL:**
- pg_basebackup para backups físicos
- WAL (Write-Ahead Logging) archiving para PITR

### 4.8 Dispositivos de Red (Cisco Switches)

**Método:** Backup de configuración (running-config, startup-config)

**Protocolos soportados:**
- SSH (preferido): `copy running-config scp://user@backupserver/configs/`
- TFTP (legacy): `copy running-config tftp://backupserver/configs/`

**Automatización:**
```bash
# Script ejemplo (Oxidized, Rancid, o custom)
#!/bin/bash
ssh admin@switch-ip "show running-config" > /backups/switch-$(date +%F).cfg
```

**Frecuencia:**
- Backup automático: Diario (detecta cambios con diff)
- Backup bajo demanda: Antes/después de cada cambio (pre/post change)

**Permisos:**
- Usuario SSH: `backup-user` (privilege level 15 o enable password)
- Alternativa: TACACS+ con role read-only (solo `show` commands)

**Versionado:**
- Mantener historial de configuraciones (Git repository recomendado)
- Retención: 30 días de cambios, luego 1 snapshot mensual durante 12 meses

### 4.9 Firewalls (Fortinet FortiGate)

**Método:** Backup de configuración via CLI o API

**Opciones de backup:**
1. **Full config:** `execute backup config management-station <comment>`
2. **VDOM-specific:** `execute backup config management-station <VDOM>`

**Automatización:**
```python
# Ejemplo con API REST
import requests
session = requests.Session()
response = session.post(
    'https://fortigate-ip/api/v2/monitor/system/config/backup',
    params={'scope': 'global', 'access_token': 'YOUR_TOKEN'},
    verify=False
)
with open('fortigate-backup.conf', 'wb') as f:
    f.write(response.content)
```

**Frecuencia:** Diario + pre/post change

**Permisos:**
- Usuario API: `backup-admin` (read-only profile + backup permission)
- Acceso: HTTPS API o SSH

**Encriptación:**
- Configuraciones contienen passwords → encriptar backups (AES-256)

### 4.10 SAP

**Método:** SAP-specific backup (BR*TOOLS)

**Componentes a proteger:**
1. **SAP Database (HANA, Oracle, SQL Server):** Backup database específico
2. **SAP Application files:** `/usr/sap/`, archivos de configuración
3. **SAP Transport directory:** `/usr/sap/trans/`

**Integración:**
- Usar SAP BackInt API para integración con solución de backup enterprise
- Retención: Según compliance (típicamente 7 años para datos financieros)

**Responsabilidad:**
- SAP Team ejecuta y valida backups
- Backup team provee storage target e integración

**Nota:** SAP centralizado (no edge) → backups se gestionan desde datacenter corporativo, NO en plantas.

---

## 5. Requisitos Técnicos de la Solución

### 5.1 Requisitos Funcionales Obligatorios

Cualquier solución de backup seleccionada por una planta DEBE cumplir:

| Req ID | Requisito | Justificación | Verificación |
|--------|-----------|---------------|--------------|
| **REQ-001** | Soportar Windows, Linux, Hyper-V, VMware, Nutanix | Heterogeneidad de plantas | Test en ambiente lab |
| **REQ-002** | Deduplicación inline o post-process | Ahorro de storage (factor 10-20×) | Validar ratio dedup en producción |
| **REQ-003** | Encriptación AES-256 at rest | Compliance (GDPR, ISO 27001) | Auditoría de seguridad |
| **REQ-004** | Soporte NFS, iSCSI, FC, discos locales | Flexibilidad de targets | Test conectividad a diferentes targets |
| **REQ-005** | API REST para monitoring | Integración con SIEM/monitoring corporativo | Revisar documentación API |
| **REQ-006** | Role-Based Access Control (RBAC) | Segregación de permisos por equipo | Configurar roles en ambiente lab |
| **REQ-007** | Alerting configurable (email, SNMP, webhook) | Notificación proactiva de fallos | Test envío de alertas |
| **REQ-008** | Restore granular (file-level, VM-level, DB-level) | Flexibilidad de recovery | Test restore de diferentes tipos |
| **REQ-009** | Changed Block Tracking (CBT) para VMs | Eficiencia de incrementales (90% reducción tiempo) | Validar CBT funciona correctamente |
| **REQ-010** | Support for SQL VSS, Oracle RMAN, MySQL hot backup | Application consistency | Test backup+restore de DBs |

### 5.2 Requisitos No Funcionales

| Req ID | Requisito | Target | Measurement |
|--------|-----------|--------|-------------|
| **REQ-NFR-001** | Performance backup | > 1 TB/hora por backup server | Monitorizar throughput en jobs |
| **REQ-NFR-002** | Performance restore | > 500 GB/hora para Platino/Oro | Test restore y medir tiempo |
| **REQ-NFR-003** | Concurrencia | Mínimo 20 jobs simultáneos | Test con 20 VMs en paralelo |
| **REQ-NFR-004** | Escalabilidad | Soportar 500+ VMs por backup server | Validar en ambiente más grande |
| **REQ-NFR-005** | Disponibilidad del backup server | 99.5% (< 3.6h downtime/mes) | Monitorizar uptime |

### 5.3 Requisitos de Seguridad

| Req ID | Requisito | Implementation |
|--------|-----------|----------------|
| **REQ-SEC-001** | Mínimos puertos abiertos | Solo puertos necesarios (ver Sección 6.2) |
| **REQ-SEC-002** | Least privilege para agents | Documentar permisos mínimos por workload |
| **REQ-SEC-003** | Segregación IT/OT | Agentes OT NO pueden comunicar directamente con IT backup server sin firewall |
| **REQ-SEC-004** | Audit logging | Todos los accesos a backups deben loguearse (quien, cuando, qué) |
| **REQ-SEC-005** | Immutability opcional | Para Platino, opción de backups immutable (no modificables ni borrables por ransomware) |
| **REQ-SEC-006** | MFA para restore | Restores de datos Platino requieren MFA (admin + security approval) |

### 5.4 Interoperabilidad con Azure (Opcional)

Para plantas que opten por Azure como tertiary target:

**Requisitos:**
- Integración nativa con Azure Blob Storage (Cool/Archive tier)
- Uso de Azure Managed Identity o Service Principal (NO storage account keys)
- Transferencia via ExpressRoute o VPN (NO Internet público)
- Lifecycle policies automáticas (Cool → Archive tras 365 días)

**Costes estimados:**
- Upload: Gratis via ExpressRoute
- Storage Cool: $0.01/GB/mes
- Storage Archive: $0.001/GB/mes
- Retrieval Archive: $0.02/GB (solo cuando se necesita restore)

---

## 6. Seguridad y Compliance

### 6.1 Modelo Purdue (IT/OT Segregation)

**Zonas definidas:**
- **Level 0-1 (Field devices):** Sensores, actuadores → NO backup directo
- **Level 2 (Control):** PLCs, DCS → Backup de configuración via OT network
- **Level 3 (Operations):** SCADA, MES, HMI → Backup con agent OT
- **Level 4 (Enterprise IT):** ERP, File Servers, VMs → Backup con agent IT
- **Level 5 (Corporate):** WAN, Internet → Replicación off-site

**Reglas de comunicación:**
```
Level 2-3 (OT) → Firewall → Level 4 (IT Backup Server)
    ↓
Permitido solo:
  • Puerto TCP 10000-10010 (backup traffic — configurable)
  • Origen: IPs whitelisted (agentes OT)
  • Destino: IP backup server IT
  • Protocolo: TLS 1.3 obligatorio
```

**Forbidden:**
- Agentes IT NO pueden acceder a red OT (unidireccional OT → IT solo)
- Backup server IT NO puede iniciar conexiones a OT (agentes pull, no push)

### 6.2 Puertos de Red Requeridos

#### Backup Server → Agents (Management)

| Protocolo | Puerto | Dirección | Uso | Firewall Rule |
|-----------|--------|-----------|-----|---------------|
| HTTPS | TCP 443 | Server → Agent | Agent management, job trigger | Allow from backup server IP |
| SSH | TCP 22 | Server → Linux Agent | Linux agent communication | Allow from backup server IP |
| WinRM | TCP 5985/5986 | Server → Windows Agent | Windows agent communication (alternativa) | Allow from backup server IP |
| Vendor-specific | TCP 6160-6170 | Server ↔ Agent | Backup solution specific (ej: Veeam, Commvault) | Allow bidirectional |

#### Agents → Backup Server (Data transfer)

| Protocolo | Puerto | Dirección | Uso |
|-----------|--------|-----------|-----|
| Backup Data | TCP 10000-10010 | Agent → Server | Encrypted backup data stream |
| Incremental | TCP 10011-10020 | Agent → Server | Changed blocks only |

#### Backup Server → Targets

| Target Type | Protocolo | Puerto | Uso |
|-------------|-----------|--------|-----|
| **NFS** | TCP 2049 | Server → NAS | NFS mount |
| **iSCSI** | TCP 3260 | Server → SAN | iSCSI target |
| **SMB/CIFS** | TCP 445 | Server → NAS | Windows share |
| **FC** | N/A (no TCP) | — | Fibre Channel (direct attach) |
| **Azure Blob** | TCP 443 (HTTPS) | Server → Azure | Cloud storage |

**Principio:** Mínimos puertos posibles. Si solución de backup requiere >15 puertos diferentes, escalar a Security para revisión.

### 6.3 Encriptación

**Obligatoria en:**
- Data in transit: TLS 1.3 (agent ↔ server, server ↔ targets)
- Data at rest: AES-256 (backups en targets)

**Gestión de claves:**
- Claves almacenadas en backup server (database encriptada)
- Rotación de claves: Cada 90 días (automática)
- Backup de claves: Encriptadas y almacenadas en vault separado (Azure Key Vault o HSM on-premise)

**Recovery de claves:**
- Sin clave = sin restore → proceso de recuperación de clave documentado
- Custodios de claves: Mínimo 2 personas (separation of duties)

### 6.4 Compliance

| Regulación | Requisito | Implementación en Backup |
|------------|-----------|-------------------------|
| **GDPR** | Right to deletion | Proceso de borrado seguro de backups (overwrite 3 pases) |
| **GDPR** | Data residency | Backups de plantas EU almacenados en EU (no transferir a US sin safeguards) |
| **ISO 27001** | Access control | RBAC, audit logs, MFA para admin |
| **SOX** | Financial data retention | Backups de sistemas financieros (SAP) 7 años mínimo |
| **HIPAA** | (si aplica) Healthcare data | Encriptación, access logs, BAA (Business Associate Agreement) con vendor |

---

## 7. Monitorización y Alerting

### 7.1 Métricas Clave (KPIs)

| Métrica | Target | Fuente de Datos | Frecuencia |
|---------|--------|----------------|------------|
| **Backup Success Rate** | > 99% | Backup server logs | Diario |
| **Failed Jobs** | < 1% | Job status API | Tiempo real |
| **Average Backup Duration** | Baseline + 20% max | Job metadata | Semanal |
| **Deduplication Ratio** | > 10:1 | Backup server stats | Semanal |
| **Storage Capacity Used** | < 80% | Target storage monitoring | Diario |
| **Recovery Success Rate** | > 99% | Restore tests logs | Mensual |
| **Agent Heartbeat** | 100% online | Agent status API | Cada 15 min |

### 7.2 Alerting

**Severidad de Alertas:**

| Severity | Condición | Notificación | Respuesta Esperada |
|----------|-----------|--------------|-------------------|
| **Critical (P1)** | Backup Platino/Oro falló 2 veces consecutivas | PagerDuty + Email | < 1 hora |
| **Critical (P1)** | Storage capacity > 90% | PagerDuty + Email | < 4 horas |
| **Critical (P1)** | Backup server down > 30 min | PagerDuty + SMS | Inmediato |
| **High (P2)** | Backup Plata falló | Email | < 8 horas |
| **High (P2)** | Dedup ratio < 5:1 (degraded) | Email | < 24 horas |
| **Medium (P3)** | Agent offline > 24h | Email | < 48 horas |
| **Low (P4)** | Job duration > baseline + 50% | Email semanal | Best effort |

**Destinatarios:**
- **P1:** On-call engineer + Backup team lead + IT Manager
- **P2:** Backup team
- **P3:** Backup team (daily digest)
- **P4:** Backup team (weekly report)

### 7.3 Dashboards

**Dashboard Principal (Backup Operations):**
- Jobs últimas 24h (success/failed/running)
- Backup success rate (últimos 7 días)
- Storage capacity trend (últimos 30 días)
- Top 10 jobs más largos
- Agents offline

**Dashboard por Planta:**
- Clonación de dashboard principal filtrado por planta específica
- Acceso: Plant IT Manager (read-only)

**Dashboard Ejecutivo (Management):**
- Backup success rate global (todas las plantas)
- Cumplimiento de SLAs (RPO/RTO)
- Coste por TB protegido
- Failed restores (crítico)

### 7.4 Integración con Monitoring Corporativo

**Plataformas soportadas:**
- Zabbix: Via SNMP traps o API REST
- Prometheus: Exporter custom (backup_jobs_total, backup_duration_seconds, etc.)
- Splunk: Forward logs via syslog
- Azure Monitor: Via Log Analytics agent (si backup server es VM Azure)

**Métricas exportadas (Prometheus ejemplo):**
```
# HELP backup_jobs_total Total backup jobs executed
# TYPE backup_jobs_total counter
backup_jobs_total{tier="platino",status="success"} 1543
backup_jobs_total{tier="platino",status="failed"} 3

# HELP backup_duration_seconds Backup job duration
# TYPE backup_duration_seconds histogram
backup_duration_seconds{tier="oro",workload="vm"} 1834.5

# HELP backup_dedup_ratio Current deduplication ratio
# TYPE backup_dedup_ratio gauge
backup_dedup_ratio{plant="EU01"} 18.7
```

---

## 8. Procedimientos Operacionales

### 8.1 Proceso de Backup

**Workflow estándar:**
```
1. Pre-backup checks (automated)
   ├─ Verificar espacio disponible en targets (> 20% free)
   ├─ Verificar agentes online (heartbeat < 5 min)
   └─ Verificar no hay jobs colgados de backup anterior

2. Backup execution
   ├─ Quiesce application (VSS, snapshot, DB hot backup mode)
   ├─ Transfer data (agent → server → target)
   ├─ Verify backup integrity (checksum, test restore sample)
   └─ Un-quiesce application (remove snapshot, exit backup mode)

3. Post-backup tasks
   ├─ Update catalog (metadata de backup)
   ├─ Trigger copy job (primary → secondary target)
   ├─ Send metrics a monitoring
   └─ Alertar si fallos

4. Cleanup
   ├─ Consolidar snapshots (hypervisors)
   ├─ Truncate transaction logs (databases en modo FULL)
   └─ Rotar backups antiguos (aplicar retention policy)
```

**Ventanas de Backup:**
- **Producción:** 22:00 - 06:00 (8 horas) — aplicación general
- **SAP:** Ventana específica coordinada con SAP team (típicamente domingo 01:00-05:00)
- **Databases grandes:** Backups full en fin de semana, incrementales diarios

### 8.2 Proceso de Restore

**Workflow estándar:**
```
1. Request validation
   ├─ Ticket ServiceNow con justificación
   ├─ Aprobación del data owner (app team)
   └─ Para Platino: Aprobación adicional de IT Manager + Security (MFA)

2. Identificar backup point
   ├─ Determinar fecha/hora del restore deseado
   ├─ Verificar backup existe en catalog
   └─ Si está en tape/Azure Archive: iniciar recall (puede tomar 4-48h)

3. Restore execution
   ├─ Opción A: In-place restore (sobreescribir producción)
   │   └─ CUIDADO: Solo con aprobación explícita, riesgo de pérdida datos actuales
   ├─ Opción B: Restore a ubicación alternativa (recomendado)
   │   ├─ Restore a VM temporal o carpeta separada
   │   ├─ Validar datos restaurados
   │   └─ Copiar manualmente a producción si correcto
   └─ Opción C: Instant recovery (solo VMs)
       └─ VM arranca directo desde backup (thin provisioning), luego migrar a producción

4. Validation
   ├─ Application owner valida datos restaurados
   ├─ Test funcionalidad (si es VM/DB)
   └─ Sign-off formal de que restore fue exitoso

5. Cleanup
   ├─ Si restore fue a ubicación alternativa: borrar después de validación
   └─ Documentar restore en ticket (qué se restauró, de cuándo, resultado)
```

**RTO Tracking:**
- Timer comienza: Cuando se abre ticket de restore
- Timer termina: Cuando application owner confirma datos disponibles y funcionales
- Métrica registrada: tiempo total en hours
- Escalación: Si excede RTO del tier, escalar a IT Manager

### 8.3 Disaster Recovery Testing

**Frecuencia:**
- **Platino:** Trimestral (4× año)
- **Oro:** Semestral (2× año)
- **Plata:** Anual
- **Bronce:** No requiere (archival)

**Metodología:**
```
1. Planning (1 semana antes)
   ├─ Seleccionar sistemas a testar (1-2 VMs o 1 DB por tier)
   ├─ Coordinar con application owners
   ├─ Documentar paso a paso en runbook
   └─ Comunicar ventana de test (típicamente fuera de horario)

2. Execution (día del test)
   ├─ Restore a ambiente aislado (no impactar producción)
   ├─ Medir tiempos (debe estar dentro de RTO)
   ├─ Application owner valida datos y funcionalidad
   └─ Documentar issues encontrados

3. Post-test review
   ├─ Analizar métricas (RTO achieved vs. target)
   ├─ Identificar gaps (ej: restore tomó 6h pero RTO es 4h)
   ├─ Crear action items para remediar gaps
   └─ Reporte a management (% cumplimiento SLA)

4. Continuous improvement
   └─ Actualizar runbooks con lecciones aprendidas
```

**Registro de Tests:**
- Template en SharePoint: "DR Test Report - [Planta] - [Fecha]"
- Incluir: Sistema probado, Backup date, Restore duration, Éxito/Fallo, Issues, Action items

### 8.4 Gestión de Capacidad

**Revisión mensual:**
```sql
-- Query ejemplo (adaptarlo a tu backup solution)
SELECT 
    target_name,
    capacity_total_tb,
    capacity_used_tb,
    (capacity_used_tb / capacity_total_tb * 100) AS used_percent,
    dedup_ratio,
    projected_full_date  -- based on growth rate
FROM backup_storage_capacity
WHERE used_percent > 70  -- alert threshold
ORDER BY projected_full_date ASC;
```

**Acciones según nivel de llenado:**
- **< 70%:** OK (green)
- **70-80%:** Warning (yellow) — Planear expansión en próximos 3 meses
- **80-90%:** Alert (orange) — Expansión urgente (< 1 mes)
- **> 90%:** Critical (red) — Expansión inmediata o habilitar borrado anticipado backups no críticos

**Proyección de crecimiento:**
- Calcular tasa de crecimiento mensual: `(Mes_Actual - Mes_Anterior) / Mes_Anterior * 100`
- Proyectar cuándo se llenará: `Espacio_Libre / Tasa_Crecimiento_Mensual = Meses_Restantes`
- Ejemplo: 20 TB libre, crecimiento 2 TB/mes → lleno en 10 meses

---

## 9. Roles y Responsabilidades (RACI)

### 9.1 Governance

| Actividad | Arquitectura IT | Virtualización | Sistemas OS | BBDD | SAP | Comunicaciones | Industria | Security |
|-----------|----------------|----------------|-------------|------|-----|----------------|-----------|----------|
| **Definir estándar backup** | **R** A | C | C | C | C | C | C | C |
| **Selección solución backup (planta)** | C | **R** A | C | I | I | I | I | C |
| **Aprobación presupuesto** | C | A | I | I | I | I | **R** (Plant Manager) | I |
| **Revisión estándar (anual)** | **R** A | C | C | C | C | C | I | C |

### 9.2 Implementación

| Actividad | Arquitectura IT | Virtualización | Sistemas OS | BBDD | SAP | Comunicaciones | Industria | Security | IT Local |
|-----------|----------------|----------------|-------------|------|-----|----------------|-----------|----------|----------|
| **Instalar backup server** | C | **R** A | C | I | I | I | I | C | C |
| **Configurar firewall rules** | C | I | I | I | I | **R** A | I | A | C |
| **Instalar agentes Windows/Linux** | I | C | **R** A | I | I | I | I | I | C |
| **Configurar backup VMs** | C | **R** A | I | I | I | I | I | I | C |
| **Configurar backup DBs** | I | I | C | **R** A | I | I | I | I | C |
| **Configurar backup SAP** | I | I | I | C | **R** A | I | I | I | I |
| **Configurar backup network devices** | I | I | I | I | I | **R** A | I | I | C |

### 9.3 Operaciones

| Actividad | Virtualización | Sistemas OS | BBDD | SAP | Comunicaciones | IT Local | Security |
|-----------|----------------|-------------|------|-----|----------------|----------|----------|
| **Monitorización diaria backups** | C | C | C | C | C | **R** A | I |
| **Resolver fallos backup Windows/Linux** | I | **R** A | I | I | I | C | I |
| **Resolver fallos backup VMs** | **R** A | C | I | I | I | C | I |
| **Resolver fallos backup DBs** | I | C | **R** A | I | I | C | I |
| **Resolver fallos backup SAP** | I | I | C | **R** A | I | C | I |
| **Resolver fallos backup network** | I | I | I | I | **R** A | C | I |
| **Ejecutar restores Platino/Oro** | C | C | C | C | C | **R** A | A (approval) |
| **DR testing (coordinar)** | C | C | C | C | C | **R** A | I |
| **Gestión capacidad storage** | C | I | I | I | I | **R** A | I |

### 9.4 Compliance y Auditoría

| Actividad | Arquitectura IT | IT Local | Security | Compliance | Audit |
|-----------|----------------|----------|----------|------------|-------|
| **Auditoría de backups (anual)** | C | C | C | **R** | A |
| **Verificar encriptación** | I | I | **R** A | C | I |
| **Verificar retención cumple policy** | C | **R** | C | A | I |
| **Generar reportes compliance** | I | **R** | I | C | I |

**Leyenda:**
- **R** = Responsible (ejecuta)
- **A** = Accountable (aprueba, responsable final)
- **C** = Consulted (se consulta, da input)
- **I** = Informed (se le informa)

---

## 10. Costes y Presupuesto

### 10.1 Modelo de Costes

**CAPEX (inversión inicial por planta):**

| Componente | Small Plant | Medium Plant | Large Plant |
|------------|-------------|--------------|-------------|
| **Backup Server** | $8,000 | $15,000 | $30,000 |
| **Primary Target (Disk)** | $15,000 (20 TB) | $40,000 (60 TB) | $100,000 (150 TB) |
| **Secondary Target** | $10,000 (Tape LTO-8) | $25,000 (NAS remoto) | $60,000 (Tape Library) |
| **Switches/Networking** | $3,000 | $5,000 | $10,000 |
| **Backup Software Licenses** | $15,000 | $40,000 | $80,000 |
| **Professional Services (deploy)** | $5,000 | $10,000 | $15,000 |
| **TOTAL CAPEX** | **~$56,000** | **~$135,000** | **~$295,000** |

**OPEX (anual por planta):**

| Componente | Small | Medium | Large |
|------------|-------|--------|-------|
| **Software Maintenance (20% licenses)** | $3,000 | $8,000 | $16,000 |
| **Hardware Maintenance** | $2,000 | $4,000 | $8,000 |
| **Tape Media (new tapes yearly)** | $1,000 | $2,000 | $5,000 |
| **Cloud Storage (Azure - optional)** | $3,600 (300 GB Archive) | $12,000 (1 TB) | $36,000 (3 TB) |
| **Staff (0.5 FTE backup admin)** | $30,000 | $30,000 | $60,000 (1 FTE) |
| **TOTAL OPEX/año** | **~$40,000** | **~$56,000** | **~$125,000** |

**TCO 5 años:**
- Small: $56k CAPEX + ($40k × 5) OPEX = **$256,000**
- Medium: $135k + ($56k × 5) = **$415,000**
- Large: $295k + ($125k × 5) = **$920,000**

**Coste por TB protegido (estimado):**
- Small (10 TB protegidos): $256k / 5 años / 10 TB = **$5/TB/mes**
- Medium (50 TB): $415k / 5 / 50 = **$1.4/TB/mes**
- Large (200 TB): $920k / 5 / 200 = **$0.77/TB/mes**

### 10.2 Optimización de Costes

**Estrategias:**
1. **Deduplicación agresiva:** Ratio 15:1 reduce storage requerido en 93%
2. **Tiering inteligente:**
   - Hot backups (últimos 7 días): Disk rápido
   - Warm (8-30 días): Disk más económico o tape
   - Cold (>30 días): Tape o Azure Archive (¢0.001/GB)
3. **Consolidación:** 1 backup server para múltiples plantas pequeñas (si están cercanas)
4. **Cloud como secondary/tertiary:** Evita CAPEX de tape library

**ROI de deduplicación:**
```
Sin dedup: 100 TB × $500/TB hardware = $50,000
Con dedup 15:1: 100 TB / 15 = 6.7 TB × $500/TB = $3,350
Ahorro: $46,650 (93%)
```

---

## 11. Roadmap de Implementación

### 11.1 Fases de Rollout

**Fase 1: Piloto (Mes 1-3)**
- Seleccionar 2 plantas (1 small, 1 medium)
- Implementar solución backup completa
- Validar todos los workloads (OS, VMs, DBs, network devices)
- Lessons learned y ajustar estándar

**Fase 2: Rollout Regional (Mes 4-9)**
- Europa: 8 plantas
- América: 5 plantas
- Asia-Pacífico: 3 plantas
- Priorizar plantas Tier 1 (mayor criticidad)

**Fase 3: Rollout Global (Mes 10-18)**
- Plantas restantes (30+)
- Incluir plantas pequeñas/remotas

**Fase 4: Optimización (Mes 19-24)**
- Implementar cloud backup (Azure) en plantas con baja capacidad local
- Consolidar backup servers donde posible
- Automatización avanzada (self-service restore para usuarios)

### 11.2 Criterios de Éxito

| Criterio | Target | Measurement |
|----------|--------|-------------|
| **% plantas con backup compliant** | 100% en 24 meses | Auditoría trimestral |
| **Backup success rate global** | > 99% | Dashboard consolidado |
| **DR tests passed** | > 95% (todos los tests) | Test reports |
| **Coste por TB** | < $50/TB/mes | Financial reports |
| **Satisfacción stakeholders** | > 8/10 | Survey anual |

---

## 12. Apéndices

### 12.1 Glosario

| Término | Definición |
|---------|------------|
| **Backup Full** | Copia completa de todos los datos (baseline) |
| **Backup Incremental** | Copia solo de cambios desde último backup (full o incremental) |
| **Backup Diferencial** | Copia de cambios desde último backup full |
| **CDP (Continuous Data Protection)** | Backup continuo (casi tiempo real, típicamente cada 5-15 min) |
| **Deduplicación** | Eliminación de datos duplicados para ahorrar espacio |
| **RPO (Recovery Point Objective)** | Máximo tiempo de datos que podemos perder |
| **RTO (Recovery Time Objective)** | Máximo tiempo para restaurar servicio |
| **3-2-1 Rule** | 3 copias, 2 medios, 1 off-site |
| **CBT (Changed Block Tracking)** | Tecnología que rastrea qué bloques cambiaron (eficiencia incrementales) |
| **Immutable Backup** | Backup que no puede modificarse ni borrarse (protección ransomware) |

### 12.2 Referencias

- **ISA-95 (Purdue Model):** https://www.isa.org/standards-and-publications/isa-standards/isa-standards-committees/isa95
- **NIST Cybersecurity Framework:** https://www.nist.gov/cyberframework
- **ISO 27001:2013 (Backup & Recovery):** Clauses A.12.3, A.17.1
- **3-2-1 Backup Rule:** https://www.backblaze.com/blog/the-3-2-1-backup-strategy/
- **Azure Blob Storage Pricing:** https://azure.microsoft.com/en-us/pricing/details/storage/blobs/

### 12.3 Documentos Relacionados

- **[Runbook 01] - Configuración Inicial de Backup** (ver carpeta runbooks/)
- **[Runbook 02] - Backup de VMs (Hyper-V/VMware)** (ver carpeta runbooks/)
- **[Runbook 03] - Backup de Bases de Datos SQL** (ver carpeta runbooks/)
- **[Runbook 04] - Backup de Dispositivos de Red** (ver carpeta runbooks/)
- **[Runbook 05] - Procedimiento de Restore** (ver carpeta runbooks/)
- **[Runbook 06] - Troubleshooting Backups** (ver carpeta runbooks/)

---

## 13. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| **CTO** | [Nombre] | ✅ | 2025-02-19 |
| **CISO (Security)** | [Nombre] | ✅ | 2025-02-19 |
| **Infrastructure Director** | [Nombre] | ✅ | 2025-02-19 |
| **Compliance Officer** | [Nombre] | ✅ | 2025-02-19 |

---

## 14. Change History

| Version | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 1.0 | 2025-02-19 | Arquitectura IT | Versión inicial — Estándar global de backup industrial |

---

**Notas Finales:**
- Este estándar debe revisarse **anualmente** o cuando haya cambios significativos (nueva tecnología, cambios regulatorios)
- Excepciones al estándar requieren aprobación formal de CTO + CISO
- Cada planta debe mantener un "Backup Implementation Plan" específico que detalle cómo cumple este estándar
