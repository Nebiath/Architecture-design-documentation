# Windows — Sistemas Operativos Windows Server

> **Propietario:** @windows-team @windows-lead  
> **Última actualización:** YYYY-MM-DD  
> **Versión:** 1.0

Documentación completa del departamento Windows: Active Directory, Group Policy, roles de servidor, patching, seguridad y procedimientos operacionales.

**Estándares:** Windows Server 2019 y Windows Server 2022

---

## 📋 Documentos

### 🏗️ Arquitectura
- [Active Directory](./architecture/active-directory.md) — Diseño de AD, DCs, sites, replication
- [Group Policy Design](./architecture/group-policy-design.md) — Estructura de OUs, GPOs corporativas
- [Certificate Services](./architecture/certificate-services.md) — PKI corporativa, CA hierarchy

### 🖥️ Server Roles
- [File Servers](./server-roles/file-servers.md) — DFS-N, DFS-R, quotas, permisos NTFS
- [Print Servers](./server-roles/print-servers.md) — Print Management, driver deployment
- [IIS Architecture](./server-roles/iis-architecture.md) — IIS 10 en Server 2019/2022

### 🔄 Patching
- [WSUS Architecture](./patching/wsus-architecture.md) — Windows Server Update Services
- [Patch Management](./patching/patch-management.md) — Proceso mensual, testing, rollback

### 🔒 Seguridad
- [Baseline Configuration](./security/baseline-configuration.md) — CIS Benchmark Windows Server
- [Antivirus Strategy](./security/antivirus-strategy.md) — Defender, EDR, exclusions

### 📘 Runbooks
- [Server Build](./runbooks/server-build.md) — Provisioning, sysprep, templates
- [Domain Controller Maintenance](./runbooks/domain-controller-maintenance.md) — DC health, FSMO roles

---

## 🖥️ Inventario de Servidores Windows

| Entorno | Server 2022 | Server 2019 | Server 2016 | Total |
|---------|-------------|-------------|-------------|-------|
| Producción | [N] | [N] | [N] | [N] |
| Pre-producción | [N] | [N] | [N] | [N] |
| Desarrollo | [N] | [N] | [N] | [N] |
| **Total** | [N] | [N] | [N] | **[N]** |

---

## 📐 Stack Tecnológico Windows

| Componente | Versión | Notas |
|-----------|---------|-------|
| **OS Estándar** | Windows Server 2022 / 2019 | Datacenter Edition |
| Active Directory | Functional Level 2016+ | DCs en 2019/2022 |
| Group Policy | ADMX Central Store | — |
| Patching | WSUS / SCCM | Update compliance |
| Antivirus | Microsoft Defender + EDR | — |
| Backup | Azure Backup / Veeam | — |
| Monitoring | SCOM / Zabbix / Azure Monitor | — |

---

## 📊 KPIs del Departamento

| Métrica | SLA Objetivo | Medición |
|---------|-------------|----------|
| Disponibilidad DCs | 99,99% | SCOM |
| AD Replication latency | < 15 min | repadmin |
| Patch compliance | 95% en 30 días | WSUS reports |
| GPO apply time | < 5 min | gpresult |
| MTTR incidente Windows | < 2 horas | ServiceNow |

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @windows-lead | Creación inicial |
