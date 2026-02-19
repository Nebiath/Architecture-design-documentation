# Industrial Image Backup to Azure — Service Catalog

> **Version:** 1.0  
> **Fecha:** 2025-02-19  
> **Estado:** Approved  
> **Autor:** Arquitectura IT  
> **Service Owner:** Arquitectura IT  
> **Última Revisión:** 2025-02-19

---

## 1. Service Overview

### 1.1 Service Name
**Industrial Image Backup to Azure**

### 1.2 Service Description
Sistema automatizado de backup de imágenes industriales generadas en redes OT (Operational Technology) de plantas industriales, con transferencia segura a Azure Blob Storage para retención de largo plazo (30 años), cumpliendo con el modelo Purdue de seguridad y optimizando costes mediante tiering Cool/Archive.

### 1.3 Service Type
- **Category:** Infrastructure Services → Backup & DR
- **Tier:** Tier 2 (Important — non-critical but impactful)
- **Deployment Model:** Hybrid (On-premise gateway + Azure cloud storage)

### 1.4 Current Status
- **Status:** ✅ Production (GA — General Availability)
- **Since:** 2025-02-19
- **Lifecycle Stage:** Active development
- **End-of-Life:** No planned (servicio indefinido)

---

## 2. Business Information

### 2.1 Business Purpose
**Problem Statement:**
Plantas industriales generan miles de imágenes diarias (cámaras de inspección, sensores visuales, documentación de procesos) que deben ser retenidas durante 30 años por requisitos de compliance y auditoría. El almacenamiento on-premise es costoso, no escalable y no cumple con requisitos de durabilidad.

**Solution:**
Backup automático a Azure Blob Storage con lifecycle management (Cool → Archive), garantizando:
- Durabilidad 99.999999999%
- Costes optimizados (Archive tier: $0.00099/GB/mes)
- Escalabilidad ilimitada
- Compliance con regulaciones (GDPR, Purdue model)

### 2.2 Business Value
- **Cost Savings:** ~70% vs on-premise NAS para retención 30 años
- **Risk Mitigation:** Evitar pérdida de datos por fallos hardware on-premise
- **Compliance:** Cumplir con auditorías (datos accesibles 30 años)
- **Scalability:** Soportar crecimiento de plantas sin CAPEX

### 2.3 Target Audience
- **Primary Users:** Negocio Industrial (Plant Managers, QA teams)
- **Secondary Users:** IT Local (soporte on-site), Compliance/Audit teams
- **Consumers:** Servicio backend (no interfaz usuario), datos consumidos por sistemas de análisis futuros

---

## 3. Technical Specifications

### 3.1 Service Components
| Component | Technology | Location | Owner |
|-----------|-----------|----------|-------|
| **Gateway VM** | Windows Server 2022 | IT network de cada planta | IT Operations |
| **Automation Scripts** | PowerShell 7.4 | VM Gateway | Public Cloud Team |
| **Transfer Tool** | AzCopy v10 | VM Gateway | Public Cloud Team |
| **Storage** | Azure Blob Storage (Cool/Archive) | Azure West Europe / East US | Public Cloud Team |
| **Networking** | Azure ExpressRoute / VPN | Planta → Azure | Network Team |
| **Firewall** | Checkpoint | OT ↔ IT boundary | Network Team |

### 3.2 Architecture Pattern
**Pattern:** Event-Driven Batch Processing
- Files arrive → Detected (event) → Uploaded (batch) → Archived (lifecycle)

### 3.3 Data Flow
```
[OT Equipment] → [OT File Share] → [Checkpoint FW:445] → [IT VM Gateway:SMB] 
    → [Staging] → [AzCopy:HTTPS:443] → [Azure Blob (Cool)] 
    → [365 days] → [Azure Blob (Archive)]
```

### 3.4 Capacity
| Metric | Current | Maximum | Notes |
|--------|---------|---------|-------|
| **Plants supported** | 15 | 50+ | Scalable architecture |
| **Volume per plant** | 100-500 GB/mes | 1 TB/mes | VM disk size limit |
| **Total storage** | 7.5 TB (15 plantas × 500 GB × 1 año) | Unlimited | Azure Blob scales infinitely |
| **Bandwidth required** | 50 Mbps (peak) | 1 Gbps available | ExpressRoute capacity |

---

## 4. Service Levels

### 4.1 Availability
- **Target:** 99.5% (ventana de transferencia 22:00-06:00 daily)
- **Measured:** % de días con al menos 1 upload exitoso
- **Downtime allowance:** 43.2 min/mes (~1.5 días/año)

### 4.2 Performance
| Metric | Target | Current |
|--------|--------|---------|
| **Upload latency (p95)** | < 30 min | 18 min (avg) |
| **Transfer speed** | > 50 MB/min | 120 MB/min (avg) |
| **Files per day** | 50-200 (baseline) | 150 (avg) |

### 4.3 Reliability
- **Error rate:** < 0.5% (995 de 1000 archivos suben exitosamente)
- **Data durability:** 99.999999999% (Azure Blob GRS)
- **RPO:** < 24 horas (máximo 1 día de imágenes perdidas)
- **RTO:** < 8 horas (restaurar servicio completo)

---

## 5. Pricing

### 5.1 Cost Model
**Pricing per Plant:**

| Item | Unit Cost | Quantity | Monthly Cost |
|------|-----------|----------|--------------|
| **VM Gateway** | $140/mes | 1 | $140 |
| **Local Storage** | $135/mes | 1 TB | $135 |
| **Azure Storage (Cool)** | $0.01/GB | 500 GB (año 1) | $5 |
| **Azure Storage (Archive)** | $0.00099/GB | 0 GB (año 1) | $0 |
| **ExpressRoute** | $50/mes | Prorrateado | $50 |
| **Monitoring** | $5/mes | 1 | $5 |
| **TOTAL (Year 1)** | | | **$335/mes** |

**Projected cost Year 5:** ~$366/mes (storage Archive aumenta)  
**Projected cost Year 30:** ~$513/mes (majority en Archive)

### 5.2 Billing
- **Chargeback model:** Por planta (cada planta paga su VM + storage proporcional)
- **Billing frequency:** Monthly
- **Cost center:** Negocio Industrial (cada planta tiene su cost center)

### 5.3 Discounts Available
- **Azure Reserved Instances:** 30-40% descuento en VMs (commitment 1-3 años)
- **Enterprise Agreement:** Descuento ya aplicado en pricing

---

## 6. Access and Permissions

### 6.1 How to Request Access
**Not applicable** — Service is backend-only. Data es automáticamente respaldada desde equipos OT.

### 6.2 Who Has Access
| Role | Access Type | Granted To |
|------|-------------|------------|
| **Service Owner** | Full control (Azure Owner role) | Arquitectura IT |
| **Tech Lead** | Full control (Azure Owner role) | Public Cloud Team |
| **Operations** | VM admin, read-only Azure | IT Operations |
| **IT Local** | VM local admin (via RDP) | IT Local de cada planta |
| **Negocio** | Read-only (via Azure Portal guest) | Plant Managers (on request) |

### 6.3 Data Access
**Retrieval Process:**
1. Business user solicita restauración de imágenes específicas (ticket ServiceNow)
2. Public Cloud Team valida request (compliance, justification)
3. Si datos en Cool tier: acceso inmediato (minutos)
4. Si datos en Archive tier: rehydration requerida (12-48 horas)
5. Datos disponibles vía Azure Portal o Azure Storage Explorer
6. Costo de retrieval: $0.02/GB (Archive) + $0.01/GB (Cool)

---

## 7. Support

### 7.1 Support Hours
| Level | Hours | Contact |
|-------|-------|---------|
| **L1 (IT Operations)** | 24×7 | it-ops@company.com, PagerDuty |
| **L2 (Public Cloud)** | Business hours (CET 09:00-18:00) | cloud-team@company.com |
| **L3 (Arquitectura IT)** | Business hours | architecture-team@company.com |

### 7.2 Response Times (SLA)
| Priority | Response Time | Resolution Target |
|----------|--------------|-------------------|
| **P1** (Service down, data loss risk) | 30 minutes | 4 hours |
| **P2** (Degraded) | 2 hours | 24 hours |
| **P3** (Minor issue) | 24 hours | 1 week |
| **P4** (Question) | 1 week | Best effort |

### 7.3 Support Channels
- **Incidents:** ServiceNow → Assignment Group: "Industrial Backup Support"
- **Questions:** Email → cloud-team@company.com
- **Urgent (P1):** PagerDuty → Service "Industrial Backup"
- **Collaboration:** Slack #industrial-backup

---

## 8. Onboarding

### 8.1 Prerequisites (New Plant)
- [ ] Plant code assigned (ej: EU03)
- [ ] IT network infrastructure ready (VLAN, IP addressing)
- [ ] Checkpoint firewall installed and configured
- [ ] ExpressRoute or VPN to Azure operational
- [ ] Active Directory account for OT upload service
- [ ] Business approval and cost center assigned

### 8.2 Onboarding Process
1. **Submit Request:** ServiceNow → "Request Industrial Backup Service"
2. **Kickoff Meeting:** Public Cloud + IT Local + Negocio (1 hora)
3. **Deployment:** Public Cloud Team ejecuta deployment runbook (4-6 horas)
4. **Testing:** Joint testing with IT Local (2 horas)
5. **Training:** IT Local recibe training de troubleshooting básico (1 hora)
6. **Go-Live:** Business approval formal
7. **Monitoring:** 2 semanas de monitoring intensivo post-go-live

**Timeline:** 2-3 semanas desde request hasta go-live

### 8.3 Onboarding Costs
- **Setup fee:** $0 (included in regular service cost)
- **Training:** $0 (included)
- **Recurring cost:** Starts at $335/mes (plant-specific)

---

## 9. Decommissioning

### 9.1 Offboarding Process (Plant Closure)
1. **Request:** ServiceNow → "Decommission Industrial Backup - Plant XX"
2. **Approval:** Negocio + Arquitectura IT
3. **Data Retention:** 
   - [ ] Confirm 30-year retention still required (typical: YES)
   - [ ] Transfer ownership to corporate IT (not plant IT)
4. **VM Gateway:** Power off (keep for 30 days), then delete
5. **Azure Storage:** 
   - [ ] Change chargeback to corporate cost center
   - [ ] Keep data in Archive tier (mínimo coste)
   - [ ] Set retention policy: auto-delete after 30 años
6. **Documentation:** Update inventario, remover plant de monitoring dashboards

**Cost post-decommission:** ~$5/mes (solo storage Archive, no VM)

---

## 10. Roadmap

### 10.1 Current Features (v1.0 — 2025-02-19)
- ✅ Automated upload OT → Azure
- ✅ Lifecycle Cool → Archive after 365 días
- ✅ Monitoring con Azure Monitor
- ✅ Multi-plant support (15 plantas)

### 10.2 Planned Features (2025 Q2-Q4)
- [ ] **Compression:** Compress images before upload (reduce storage 30%)
- [ ] **Web Portal:** Self-service data retrieval para Negocio (sin ticket IT)
- [ ] **Advanced Analytics:** ML para detectar imágenes corruptas pre-upload
- [ ] **Global Deduplication:** Detectar imágenes duplicadas cross-plant

### 10.3 Under Consideration (2026+)
- [ ] **Edge ML:** Procesar imágenes en edge antes de upload (reduce volume)
- [ ] **Blockchain:** Immutable audit trail de uploads (compliance avanzado)
- [ ] **Multi-cloud:** Replicate a AWS S3 Glacier para DR cross-cloud

---

## 11. Compliance and Security

### 11.1 Compliance Certifications
| Certification | Status | Notes |
|--------------|--------|-------|
| **ISO 27001** | ✅ Compliant | Arquitectura revisada y aprobada |
| **GDPR** | ✅ Compliant | Data residency (EU data en West Europe) |
| **Purdue Model (ISA-95)** | ✅ Compliant | Segmentación OT/IT via Checkpoint FW |
| **SOC 2 Type II** | ⏳ In Progress | Auditoría Q3 2025 |

### 11.2 Security Features
- ✅ Encryption at rest (AES-256)
- ✅ Encryption in transit (TLS 1.3)
- ✅ Network segmentation (Purdue model)
- ✅ Managed Identity (no passwords)
- ✅ Audit logging (all access logged)
- ✅ Soft delete (14 días retention de archivos borrados)

### 11.3 Data Classification
- **Classification:** Internal (not public, but not PII/sensitive)
- **Retention:** 30 años (regulatory requirement)
- **Geolocation:** EU data en EU, US data en US (no cross-border)

---

## 12. Related Services

### 12.1 Dependencies
| Service | Type | Criticality | Contact |
|---------|------|------------|---------|
| **Azure Subscription (Industrial-Ops)** | Upstream | Critical | cloud-team@company.com |
| **Active Directory** | Upstream | Critical | windows-team@company.com |
| **ExpressRoute** | Upstream | Critical | network-team@company.com |
| **Checkpoint Firewall** | Upstream | Critical | security-team@company.com |

### 12.2 Integrated Services
- **Azure Monitor:** Monitoring y alerting
- **Log Analytics:** Centralized logging
- **PagerDuty:** Incident alerting
- **ServiceNow:** Ticketing y change management

---

## 13. Documentation

### 13.1 Available Documentation
| Document | Location | Audience |
|----------|----------|----------|
| **Architecture Design** | 01-architecture-design.md | Technical |
| **Deployment Runbook** | 02-deployment-runbook.md | Public Cloud, IT Ops |
| **RACI Matrix** | 03-service-ownership.md | All stakeholders |
| **Observability Guide** | 04-observability.md | IT Ops, SRE |
| **Service Catalog** | 05-service-catalog.md (this doc) | All stakeholders |

### 13.2 External Links
- **Azure Blob Storage Docs:** https://learn.microsoft.com/en-us/azure/storage/blobs/
- **Purdue Model:** https://www.isa.org/standards-and-publications/isa-standards/isa-standards-committees/isa95
- **AzCopy Docs:** https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10

---

## 14. FAQs

### 14.1 General

**Q: ¿Cuánto tiempo tardan los uploads?**  
A: Promedio 15-30 minutos desde que la imagen llega a la VM Gateway IT hasta confirmación en Azure.

**Q: ¿Qué pasa si falla un upload?**  
A: AzCopy reintenta automáticamente (3 veces). Si falla después de 3 reintentos, se registra error y se alerta a IT Operations.

**Q: ¿Puedo acceder a las imágenes desde Azure?**  
A: Sí, pero requiere ticket de ServiceNow. Si datos en Archive tier, requiere rehydration (12-48h).

**Q: ¿Cuánto cuesta recuperar datos?**  
A: $0.02/GB (Archive) + $0.01/GB (Cool). Ejemplo: recuperar 100 GB de Archive = $2-3 USD.

### 14.2 Technical

**Q: ¿Por qué Windows y no Linux?**  
A: Estándar corporativo IT, integración AD nativa, SMB share nativo, PowerShell expertise in-house.

**Q: ¿Por qué Cool tier 1 año y no Archive inmediato?**  
A: Azure no permite upload directo a Archive. Además, imágenes recientes (< 1 año) ocasionalmente necesitan acceso rápido para troubleshooting.

**Q: ¿Qué pasa si una planta cierra?**  
A: VM se apaga, pero datos se mantienen en Azure durante 30 años (requisito legal). Costo post-cierre: ~$5/mes.

---

## 15. Change History

| Version | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 1.0 | 2025-02-19 | Arquitectura IT | Versión inicial — Service Catalog |

---

**Contacts:**
- **Service Owner:** architecture-team@company.com
- **Support:** it-ops@company.com (24×7)
- **Sales (new plants):** Contactar a Arquitectura IT para onboarding
