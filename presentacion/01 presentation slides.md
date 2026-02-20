---
marp: true
theme: default
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
header: 'Estandarización de Documentación IT'
footer: 'Architecture Team | 2024'
---

<!-- _class: lead -->
# 📚 Estandarización de Documentación IT
## Plan de Implementación 2024

**Single Source of Truth para Arquitectura**

*30 minutos | Incluye Q&A*

---

# 🎯 Agenda

1. **El Problema** - Por qué necesitamos esto (3 min)
2. **La Solución** - Qué proponemos (5 min)
3. **Templates** - Los 8 documentos clave (5 min)
4. **Plan 12 Meses** - Roadmap de implementación (5 min)
5. **Estructura del Repo** - Cómo nos organizamos (5 min)
6. **Primeros Pasos** - Qué hacemos YA (2 min)
7. **Q&A** - Preguntas (5 min)

---

<!-- _class: lead -->
# 1️⃣ El Problema Actual

---

# 🚨 Situación Actual

### Documentación fragmentada y obsoleta

| Problema | Impacto |
|----------|---------|
| 📂 **Dispersa** | 5+ lugares diferentes (Wiki, SharePoint, emails, Slack) |
| ⏰ **Obsoleta** | ~60% no actualizada en 6+ meses |
| ❌ **Incompleta** | 40% servicios sin documentación básica |
| 🔍 **No se encuentra** | 8h/semana buscando información |
| 🚫 **Sin ownership** | No está claro quién mantiene qué |

---

# 💰 Coste Real del Problema

```
Incidentes sin runbook: 15/mes
MTTR promedio: 2 horas
Coste por hora downtime: €1,000

→ Coste mensual: €30,000
→ Coste anual: €360,000
```

### Otros costes ocultos
- ⏱️ Onboarding lento (4 semanas vs. 1.5 ideal)
- 🔄 Conocimiento tribal → riesgo de pérdida
- 📋 Compliance: auditorías complejas
- 🎯 Decisiones lentas por falta de contexto

---

<!-- _class: lead -->
# 2️⃣ La Solución Propuesta

---

# ✨ Visión: Single Source of Truth

## Un repositorio centralizado donde:

✅ **Toda** la arquitectura está documentada
✅ **Siempre** actualizada (automatización)
✅ **Fácil** de encontrar (estructura clara)
✅ **Clara** ownership (CODEOWNERS)
✅ **Viva** (parte del workflow diario)

### Principio fundamental:
> "Si no está en el repo, no existe"

---

# 🎯 Objetivos del Proyecto

| Objetivo | Actual | Meta 12M | Mejora |
|----------|--------|----------|--------|
| **MTTR** | 2h | 45 min | **-62%** |
| **Onboarding** | 4 sem | 1.5 sem | **-62%** |
| **Servicios documentados** | 40% | 90% | **+125%** |
| **Incidentes sin runbook** | 60% | <10% | **-83%** |
| **Tiempo buscando info** | 8h/sem | 2h/sem | **-75%** |

### ROI: €215k Año 1 (payback 6 meses)

---

# 🏗️ Los 3 Pilares

### 1. **Templates Estandarizados** 📝
- 8 templates profesionales
- Basados en SRE best practices
- Listos para usar

### 2. **Proceso Simple** ⚙️
- Integrado en CI/CD
- Automatizado donde sea posible
- 80% menos esfuerzo manual

### 3. **Cultura "Doc-First"** 🌱
- Parte del Definition of Done
- Incentivos y reconocimiento
- Training y soporte continuo

---

<!-- _class: lead -->
# 3️⃣ Los 8 Templates

---

# 📄 Template 1: Architecture Design

### ¿Qué contiene?
- Requisitos funcionales y no funcionales
- **SLOs** (Service Level Objectives) - no solo SLAs
- Diagramas C4 (Context, Containers, Components)
- Dependencias upstream/downstream
- Capacity planning y DR strategy
- Compliance (GDPR, SOC2, etc.)

### ¿Cuándo usar?
Todo servicio crítico debe tener este documento

---

# 📄 Template 2: Deployment & Runbooks

### ¿Qué contiene?
- **Infrastructure as Code** como fuente de verdad
- Procedimientos de deployment (normal, hotfix, rollback)
- Runbooks operacionales
- Health checks y troubleshooting
- Database migrations

### ¿Cuándo usar?
Cualquier cosa que se despliegue en producción

### Clave: "Runbooks testeables"

---

# 📄 Template 3: Service Ownership & RACI

### ¿Qué contiene?
- Service Owner identificado
- Matriz RACI completa
- **On-call rotation** y procedimientos
- Escalation paths claros
- Team structure y contactos

### ¿Cuándo usar?
Todo servicio productivo necesita ownership

### Clave: "Si no hay owner, ¿quién responde a las 3 AM?"

---

# 📄 Template 4: Observability & Monitoring

### ¿Qué contiene?
- **SLIs** definidos (métricas medibles)
- Golden Signals (Latency, Traffic, Errors, Saturation)
- Dashboards (links)
- **Alerting** con runbooks asociados
- Logs, traces, metrics

### ¿Cuándo usar?
Si no lo puedes medir, no lo puedes operar

### Clave: "Alert → Runbook → Action"

---

# 📄 Templates 5-8: Soporte

### **5. Service Catalog Entry**
- Registro centralizado de cada servicio
- Metadata, links, contacts

### **6. Incident Management**
- Severity matrix, response workflows
- Post-mortem templates

### **7. Change Management**
- Tipos de cambios, aprobaciones
- RFC templates, CAB procedures

### **8. API Documentation**
- Si tu servicio expone APIs
- OpenAPI/Swagger standards

---

<!-- _class: lead -->
# 4️⃣ Plan de Implementación
## 12 Meses, 4 Fases

---

# 📅 Roadmap Overview

```
Q1 (Meses 1-3): FOUNDATION
├─ 100% Tier 1 documentado
├─ Templates validados
└─ Automated checks operativos

Q2 (Meses 4-6): EXPANSION
├─ 40% servicios total
├─ Training completado
└─ Self-service portal

Q3 (Meses 7-9): SCALE
├─ 60% servicios total
├─ Compliance ready
└─ Cultura "doc-first"

Q4 (Meses 10-12): EXCELLENCE
├─ 90% servicios total
├─ Process optimizado
└─ Sustainable (0.5 FTE)
```

---

# 📊 Fase 1: Foundation (Q1)

### Mes 1: Setup & Piloto
- ✅ Formar Documentation Task Force
- ✅ 3 servicios piloto documentados
- ✅ Tooling setup (Confluence/Notion + GitHub)

### Mes 2: Expansión Tier 1
- ✅ 5 servicios adicionales
- ✅ Automated checks (linting, links)
- ✅ CI/CD integration

### Mes 3: Completar Tier 1
- ✅ 100% Tier 1 documentado (10-15 servicios)
- ✅ Templates refinados v1.1
- ✅ **Go/No-Go decision** para Q2

---

# 📊 Fases 2-4 (Q2-Q4)

### Q2: Expansion
- 20 servicios documentados (total: 30)
- Automation tools deployed
- Training program operational

### Q3: Scale
- 35 servicios más (total: 65)
- Compliance templates
- Policy & mandates

### Q4: Excellence
- 25 servicios finales (total: 90)
- Process optimization
- Año 2 planning

---

# 💰 Inversión y ROI

### Inversión Año 1: **€280k**
- Personal: €200k (1 Tech Writer, 0.5 SRE, 0.25 Arch, 0.25 PM)
- Tooling: €50k (Confluence, automation)
- Training: €30k

### Ahorros Año 1: **€495k**
- Reducción MTTR: €225k
- Onboarding efficiency: €120k
- Compliance risk: €100k
- Knowledge retention: €50k

### **ROI Neto: +€215k**
### **Payback: 6 meses**

---

<!-- _class: lead -->
# 5️⃣ Estructura del Repositorio

---

# 🗂️ Organización por Dominios

```
architecture-docs/
├── 00-templates/              ← Templates reutilizables
├── 01-global/                 ← Políticas company-wide
├── 02-infrastructure/         ← POR DEPARTAMENTO
│   ├── hardware/
│   ├── virtualization/
│   ├── wan/
│   ├── lan/
│   ├── windows/
│   ├── linux/
│   ├── devops/              ← Kubernetes docs aquí
│   └── cloud-azure/         ← Azure docs aquí
├── 03-services/              ← Por servicio/aplicación
├── 04-cross-cutting/         ← Temas transversales
└── 06-decisions/             ← Architecture Decision Records
```

---

# 🏢 Ejemplo: Departamento DevOps

```
02-infrastructure/devops/
├── README.md                  ← Overview del departamento
├── architecture/
│   ├── kubernetes-architecture.md
│   ├── cluster-design.md
│   └── ingress-strategy.md
├── clusters/
│   ├── prod-cluster-eu.md     ← Inventario por cluster
│   ├── prod-cluster-us.md
│   └── dev-cluster.md
├── cicd/
│   ├── jenkins-architecture.md
│   └── argocd-configuration.md
├── observability/
│   ├── prometheus-setup.md
│   └── grafana-dashboards.md
└── runbooks/
    ├── cluster-upgrade.md
    └── disaster-recovery.md
```

---

# 🔐 Governance: CODEOWNERS

```bash
# Cada departamento "dueño" de su carpeta
/02-infrastructure/hardware/      @infrastructure-team
/02-infrastructure/virtualization/ @virtualization-team
/02-infrastructure/wan/           @network-team @wan-lead
/02-infrastructure/lan/           @network-team @lan-lead
/02-infrastructure/windows/       @windows-team
/02-infrastructure/linux/         @linux-team
/02-infrastructure/devops/        @devops-team @k8s-admin
/02-infrastructure/cloud-azure/   @cloud-team @azure-architect

# Cross-cutting requiere múltiples aprobaciones
/04-cross-cutting/security/       @security-team
/04-cross-cutting/compliance/     @compliance-team @legal-team
```

**→ Los cambios requieren aprobación del equipo dueño**

---

# ⚙️ Automatización desde Día 1

### CI/CD Pipeline (GitHub Actions)

```yaml
On Pull Request:
  1. ✅ Markdown linting
  2. ✅ Broken links check
  3. ✅ Spell check
  4. ✅ Template structure validation
  5. ✅ Required sections present?
  6. 👥 CODEOWNERS approval
  
On Merge to Main:
  1. 📤 Auto-publish to Confluence/Wiki
  2. 📊 Update metrics dashboard
  3. 💬 Notify in Slack
  4. 📈 Generate coverage report
```

### Resultado: 80% menos trabajo manual

---

<!-- _class: lead -->
# 6️⃣ Primeros Pasos
## ¿Qué hacemos AHORA?

---

# 🚀 Esta Semana (Días 1-7)

### Para Leadership:
- [ ] **Aprobar el plan y presupuesto** (€280k)
- [ ] **Asignar executive sponsor** (CTO/VP Eng)
- [ ] **Comunicar prioridad** a toda la organización

### Para Core Team:
- [ ] **Formar Documentation Task Force**
  - 1 Technical Writer (full-time)
  - 1 SRE Lead (50% time)
  - Champions por departamento
- [ ] **Seleccionar 3 servicios piloto** (Tier 1, críticos)
- [ ] **Kickoff meeting** (2h)

---

# 🎯 Próximas 2 Semanas (Días 8-14)

### Setup Técnico:
- [ ] **Crear repositorio** Git (GitHub/GitLab)
- [ ] **Implementar estructura** de carpetas
- [ ] **Setup tooling básico**:
  - Confluence Space / Notion
  - CI/CD pipeline básico
  - Slack channel #documentation

### Piloto:
- [ ] **Workshop con equipos piloto** (4h)
  - Presentar templates
  - Hands-on: Crear primer doc
- [ ] **Documentar 3 servicios** completamente
- [ ] **Iterar templates** basado en feedback

---

# 📋 Mes 1: Objetivos Claros

### Meta: Validar el enfoque

**Entregables:**
- ✅ 3 servicios completamente documentados
- ✅ Templates v1.0 aprobados
- ✅ Wiki/repo central operativo
- ✅ Team trained (10+ personas)
- ✅ Automated checks funcionando

**Decisión crítica al final del Mes 1:**
```
¿Los templates funcionan?
¿Los equipos están satisfechos?
¿Las docs son útiles en incidentes reales?

→ Si SÍ: Continuar a Fase 1 completa
→ Si NO: Pivotar estrategia
```

---

# 👥 Roles y Responsabilidades

### Core Team (Año 1)

| Rol | Time | Responsabilidad Principal |
|-----|------|--------------------------|
| **Technical Writer** | 100% | Standards, training, quality |
| **SRE Lead** | 50% | Automation, runbooks, tech guidance |
| **Architect** | 25% | Architecture docs, ADRs, review |
| **Program Manager** | 25% | Coordination, reporting, stakeholder mgmt |

### Extended Team
- **Department Champions** (10% each): Local evangelists
- **Service Owners** (5% each): Own their service docs

---

# 🎓 Training & Enablement

### Path 1: Service Owners (Obligatorio)
**4 horas | Todos los service owners**
- Intro + Templates overview
- Hands-on workshop
- Tools & automation
- Maintenance procedures

### Path 2: Champions (Avanzado)
**8 horas | 1-2 por departamento**
- Deep dive en todos los templates
- Train the trainer
- Troubleshooting & quality

### Path 3: Leadership (Ejecutivo)
**1 hora | Managers y directores**
- Business case y ROI
- Compliance benefits
- Support needed

---

<!-- _class: lead -->
# 7️⃣ Preguntas Frecuentes

---

# ❓ FAQ 1: ¿No es mucho trabajo?

### Pregunta:
*"Ya estamos sobrecargados. ¿Cómo vamos a encontrar tiempo para documentar?"*

### Respuesta:
- ✅ **Inversión inicial, ahorro continuo**
  - Primera doc de servicio: ~16h
  - Updates después: ~2h/trimestre
  - Ahorro en incidentes: >>100h/año

- ✅ **Automatización reduce 80% del esfuerzo**
  - Auto-generation de diagramas desde IaC
  - Templates pre-poblados
  - CI/CD handles validation

- ✅ **No todos los servicios a la vez**
  - Fase 1: Solo Tier 1 (10-15 servicios)
  - 12 meses para llegar a 90%

---

# ❓ FAQ 2: ¿Por qué no SharePoint/Confluence?

### Pregunta:
*"¿No podemos simplemente usar SharePoint mejor?"*

### Respuesta:
**Sí, vamos a usar Confluence/SharePoint** para visualización

**PERO:**
- 📝 **Fuente de verdad = Git** (versionado, branching, PR reviews)
- 🤖 **CI/CD valida** antes de merge (calidad garantizada)
- 👥 **CODEOWNERS** = ownership claro
- 🔄 **Auto-sync** Git → Confluence/SharePoint

### Mejor de ambos mundos:
- Developers: Editan en Git (su workflow)
- Management: Lee en Confluence (familiar)

---

# ❓ FAQ 3: ¿Y si la gente no lo usa?

### Pregunta:
*"¿Cómo garantizamos adopción?"*

### Respuesta:
**Zanahoria + Palo:**

🥕 **Incentivos positivos:**
- Gamification (leaderboards, badges)
- Recognition (monthly Doc Champion award)
- Make it easy (templates, automation, copilot)

🚫 **Enforcement (gentle):**
- Block production deploy sin docs
- Definition of Done incluye docs
- On-call rotation requiere runbooks
- Promotion criteria incluye docs

📊 **Demostrar valor:**
- "Este runbook nos salvó 2h en el incidente de ayer"
- Testimonials de equipos

---

# ❓ FAQ 4: ¿Quién mantiene esto?

### Pregunta:
*"¿Quién va a mantener toda esta documentación actualizada?"*

### Respuesta:
**Service owners** con soporte automatizado:

- 🤖 **Bot detecta docs obsoletos** (>90 días)
  → Auto-crea issue → Asigna a service owner

- 🔔 **Slack reminders** mensuales
  → "Tu doc de X necesita review"

- ✅ **Quarterly reviews** obligatorios (Tier 1)
  → Checklist automatizado

- 📊 **Metrics & visibility**
  → Leaderboard público
  → Executive dashboard

**Año 2: Solo 0.5 FTE para coordinación**

---

# ❓ FAQ 5: ¿Y los equipos externos/vendors?

### Pregunta:
*"¿Qué pasa con sistemas gestionados por vendors o equipos externos?"*

### Respuesta:
**También deben estar documentados:**

- 📄 **Docs mínimos:**
  - Service Catalog entry (contactos, SLAs)
  - Escalation procedures
  - Known issues & workarounds
  - Integration points

- 👥 **Ownership interno:**
  - Alguien interno es "Technical Owner"
  - Responsable de interfaz con vendor
  - Maintains runbooks

- 📋 **Template simplificado:**
  - "Vendor Managed Service" template
  - Menos detail técnico
  - Más focus en operations & escalation

---

<!-- _class: lead -->
# 🎯 Resumen y Llamado a la Acción

---

# 📌 Los 5 Puntos Clave

### 1. **El Problema es Real**
€360k/año en coste de downtime por falta de docs

### 2. **La Solución es Probada**
8 templates basados en SRE best practices de Google, Netflix, etc.

### 3. **El Plan es Realista**
12 meses, 4 fases, con checkpoints

### 4. **El ROI es Claro**
€280k inversión → €495k ahorro = **+€215k Año 1**

### 5. **Empieza Pequeño, Escala Después**
3 servicios piloto → 10 Tier 1 → 90% total

---

# ✅ Decisiones Necesarias HOY

### 1. **Aprobación del Plan**
- [ ] ¿Estamos de acuerdo con el enfoque?
- [ ] ¿Aprobamos el presupuesto €280k?

### 2. **Executive Sponsorship**
- [ ] ¿Quién es el executive sponsor? (CTO/VP Eng)
- [ ] ¿Compromiso visible de leadership?

### 3. **Core Team**
- [ ] ¿Quién será el Technical Writer?
- [ ] ¿Qué SRE/Architect dedicamos?
- [ ] ¿Quiénes son los champions por departamento?

### 4. **Timing**
- [ ] ¿Empezamos en Q2 2024?
- [ ] ¿Cuándo kickoff meeting?

---

# 🚀 Próximos Pasos Concretos

### Esta semana:
1. **Leadership:** Aprobar plan y presupuesto
2. **HR:** Iniciar contratación Tech Writer
3. **Teams:** Nominar department champions
4. **IT:** Setup repo + tooling básico

### Próximas 2 semanas:
5. **Kickoff meeting** con todos stakeholders
6. **Seleccionar 3 servicios piloto**
7. **Workshop** con equipos piloto (4h)
8. **Documentar primer servicio** juntos

### Mes 1:
9. **3 servicios completamente documentados**
10. **Go/No-Go decision** para continuar

---

<!-- _class: lead -->
# 💬 Preguntas y Discusión

**¿Dudas? ¿Preocupaciones? ¿Ideas?**

---

**Contacto:**
📧 docs-team@company.com
💬 Slack: #documentation
📅 Office hours: Martes 15:00 UTC

---

<!-- _class: lead -->
# ¡Gracias!

## 📚 Construyamos juntos el
## Single Source of Truth

**Documentación = Infraestructura**
**Si no está documentado, no existe**

---

# 📎 Recursos Adicionales

### Documentación:
- 📄 Plan completo: `/docs/00-standardization-plan.md`
- 📋 Templates: `/docs/00-templates/`
- 🗂️ Estructura repo: `/docs/09-repository-structure-guide.md`

### Links:
- 🌐 SRE Book: https://sre.google/books/
- 📊 DORA Metrics: https://www.devops-research.com/
- 💻 GitHub repo: [TBD]
- 📈 Dashboard: [TBD]

### Contacto:
- Program Lead: [Nombre] - [email]
- Executive Sponsor: [Nombre] - [email]
- Questions: docs-team@company.com
