# Plan de Estandarización de Documentación de Arquitectura IT/SRE

**Empresa:** Multinacional con presencia en múltiples países  
**Plataforma:** Azure DevOps  
**Fecha:** Febrero 2026  
**Autor:** Equipo SRE

---

## 📋 Tabla de Contenidos

1. [Diagnóstico de Situación Actual](#diagnóstico)
2. [Revisión de Documentación Actual vs. Mejores Prácticas](#revisión)
3. [Framework de Documentación Propuesto](#framework)
4. [Implementación con Azure DevOps](#azure-devops)
5. [Proceso de Documentación](#proceso)
6. [Plan de Implementación](#implementación)
7. [Métricas y KPIs](#métricas)
8. [Anexos](#anexos)

---

## 🔍 1. Diagnóstico de Situación Actual {#diagnóstico}

### Problemas Identificados

| Problema | Impacto | Prioridad |
|----------|---------|-----------|
| Documentación no estandarizada entre departamentos | Alto - Inconsistencia operacional | 🔴 Crítica |
| Documentación obsoleta | Alto - Decisiones basadas en info incorrecta | 🔴 Crítica |
| Sin Single Point of Truth (SPOT) | Medio - Pérdida de tiempo buscando info | 🟡 Alta |
| Información incompleta | Alto - Incidentes mal gestionados | 🔴 Crítica |
| Recursos limitados | Medio - No se mantiene la documentación | 🟡 Alta |
| Trabajo operativo domina el tiempo | Medio - No hay tiempo para documentar | 🟡 Alta |

### Entregables Actuales

**✅ Lo que tenéis (buena base):**
1. Diseño de arquitectura (requisitos, diseño lógico, SLAs)
2. Instalación y configuración
3. RACI
4. Monitorización

**❌ Lo que falta (crítico para SRE):**
- Service Level Objectives (SLOs) y Error Budgets
- Runbooks operacionales
- Incident Response Playbooks
- Disaster Recovery Plans
- Change Management procedures
- Dependency mapping
- Cost tracking
- Compliance/regulatory documentation
- Post-mortem templates
- On-call documentation

---

## 📊 2. Revisión: Documentación Actual vs. Mejores Prácticas {#revisión}

### 2.1 Análisis de Entregables Actuales

#### ✅ **Diseño de Arquitectura** - Bueno pero Incompleto

**Lo que está bien:**
- Requisitos funcionales y no funcionales: ✅
- Diseño lógico: ✅
- SLAs: ✅

**Lo que falta según mejores prácticas SRE:**

| Elemento Faltante | Criticidad | Estándar de Referencia |
|-------------------|------------|------------------------|
| **SLOs y SLIs** definidos | 🔴 Crítica | Google SRE Book - SLA != SLO |
| **Error Budget** calculado | 🔴 Crítica | Site Reliability Engineering |
| **Dependency Map** visual | 🟡 Alta | C4 Model, Arc42 |
| **Data Flow Diagrams** | 🟡 Alta | TOGAF, Arc42 |
| **Capacity Planning** | 🟡 Alta | Google SRE - Capacity Planning |
| **Cost Model** | 🟢 Media | FinOps Foundation |
| **Compliance Requirements** | 🔴 Crítica | GDPR, SOC2, ISO27001 |
| **Security Architecture** | 🔴 Crítica | Zero Trust, NIST |

**Recomendación:** Expandir con plantilla Arc42 adaptada + Google SRE principles.

---

#### ⚠️ **Instalación y Configuración** - Debe Evolucionar a IaC

**Problema actual:** Documentación manual de pasos

**Mejores prácticas modernas:**
- **Infrastructure as Code (IaC):** Terraform, ARM Templates, Bicep
- **Configuration as Code:** Ansible, DSC, Puppet
- **Deployment Pipelines:** Azure DevOps YAML pipelines
- **Rollback Procedures:** Automatizadas con blue-green o canary

**Recomendación:**
```
Documentación Manual → IaC + Pipeline + Minimal Docs
```

La documentación debe ser:
1. **Código autodocumentado** (Terraform con comentarios)
2. **Pipeline YAML** con stages claros
3. **README mínimo** con pre-requisitos y comandos
4. **Runbook de rollback** automatizado

---

#### ✅ **RACI** - Bien pero Necesita Actualización Continua

**Está bien tener RACI**, pero:
- Debe vivir en un lugar centralizado (Azure DevOps Wiki)
- Debe actualizarse automáticamente con org charts
- Debe incluir **On-Call Rotation**
- Debe integrarse con PagerDuty/Opsgenie/Azure Monitor

**Mejora propuesta:** RACI 2.0
- RACI tradicional para responsabilidades
- + **On-Call Schedule** (quién responde 24/7)
- + **Escalation Matrix** (a quién escalar)
- + **Communication Tree** (stakeholders por severidad)

---

#### ⚠️ **Monitorización** - Falta SRE Approach

**Lo que tenéis:**
- Umbrales ✅
- Flujos de escalado ✅

**Lo que falta:**

| Elemento | Descripción | Herramienta |
|----------|-------------|-------------|
| **SLIs** | Service Level Indicators medibles | Azure Monitor, Prometheus |
| **Alerting Philosophy** | Symptom-based vs cause-based | Google SRE Book Ch.6 |
| **Alert Fatigue Prevention** | Reducir alertas no actionables | Alert tuning |
| **Dashboards Hierarchy** | Overview → Service → Component | Grafana, Azure Dashboards |
| **Synthetic Monitoring** | Probes externos | Azure Monitor, Pingdom |
| **Distributed Tracing** | End-to-end transaction tracking | Application Insights, Jaeger |

**Recomendación:** Implementar SLO-based alerting

```
❌ Alert: CPU > 80%
✅ Alert: Error Budget consumido > 50% en 1h
```

---

### 2.2 Estándares Modernos de Documentación SRE

Según **Google SRE**, **DORA**, y **Platform Engineering** best practices:

#### **Documentación Mínima Viable (DMV)**

Para cada servicio en producción:

1. **Service Overview** (1 página)
   - Qué hace el servicio
   - Quién es el owner
   - Dónde está deployado
   - Cómo se accede

2. **SLOs & Error Budget** (1 página)
   - SLI: Qué medimos (latency, availability, throughput)
   - SLO: Target (99.9% uptime)
   - Error Budget: Cuánto podemos fallar (43 min/mes)

3. **Architecture Diagram** (1 diagrama)
   - C4 Context o Container level
   - Dependencies claras

4. **Runbook** (2-3 páginas)
   - Troubleshooting común
   - Comandos útiles
   - Dónde buscar logs
   - Cómo hacer rollback

5. **On-Call Guide** (1 página)
   - Qué alerts pueden saltar
   - Severidad y SLA de respuesta
   - Escalation path

**Total: ~5-7 páginas por servicio máximo**

---

## 🏗️ 3. Framework de Documentación Propuesto {#framework}

### 3.1 Estructura de Documentación (Docs-as-Code)

```
azure-devops-project/
├── README.md                          # Overview del servicio
├── docs/
│   ├── architecture/
│   │   ├── ADR/                      # Architecture Decision Records
│   │   │   ├── 001-use-cosmos-db.md
│   │   │   └── 002-multi-region.md
│   │   ├── diagrams/
│   │   │   ├── c4-context.drawio
│   │   │   ├── c4-container.drawio
│   │   │   └── data-flow.png
│   │   ├── architecture-overview.md
│   │   ├── dependencies.md
│   │   └── security-architecture.md
│   ├── operations/
│   │   ├── runbooks/
│   │   │   ├── incident-response.md
│   │   │   ├── deployment.md
│   │   │   ├── rollback.md
│   │   │   └── disaster-recovery.md
│   │   ├── monitoring/
│   │   │   ├── slos.md               # SLOs y SLIs definidos
│   │   │   ├── alerts.md             # Configuración de alertas
│   │   │   └── dashboards.md
│   │   ├── on-call/
│   │   │   ├── on-call-guide.md
│   │   │   ├── escalation-matrix.md
│   │   │   └── incident-severity.md
│   │   └── maintenance/
│   │       ├── patching-schedule.md
│   │       └── backup-procedures.md
│   ├── compliance/
│   │   ├── gdpr-compliance.md
│   │   ├── data-retention.md
│   │   └── audit-logs.md
│   └── templates/
│       ├── post-mortem-template.md
│       ├── change-request-template.md
│       └── rca-template.md
├── infrastructure/                    # IaC
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── README.md                 # Docs mínimas
│   └── azure-pipelines/
│       ├── deploy-pipeline.yml
│       └── rollback-pipeline.yml
└── RACI.md                           # Responsabilidades
```

---

### 3.2 Plantillas Estandarizadas

#### **Plantilla 1: Service Overview** (README.md)

```markdown
# [Nombre del Servicio]

## 📋 Quick Info
- **Owner Team:** [Equipo]
- **On-Call:** [Rotation link]
- **Slack Channel:** #team-[nombre]
- **Status Page:** [URL]

## 🎯 Purpose
[2-3 frases sobre qué hace este servicio]

## 🏗️ Architecture
![Architecture Diagram](docs/architecture/diagrams/c4-context.png)

## 📊 SLOs
| Metric | Target | Error Budget |
|--------|--------|--------------|
| Availability | 99.9% | 43 min/month |
| Latency (p95) | < 200ms | - |

## 🔗 Key Links
- [Architecture Docs](docs/architecture/)
- [Runbooks](docs/operations/runbooks/)
- [Monitoring Dashboard](https://portal.azure.com/...)
- [Incident Log](https://dev.azure.com/.../incident-log)

## 🚀 Getting Started
- [Local Development](docs/development/local-setup.md)
- [Deployment Guide](docs/operations/runbooks/deployment.md)

## 📞 Contact
- **Emergency:** Page via [PagerDuty/Opsgenie]
- **Questions:** #team-[nombre] on Slack
```

---

#### **Plantilla 2: Architecture Decision Record (ADR)**

```markdown
# ADR-XXX: [Título de la Decisión]

**Status:** [Proposed | Accepted | Deprecated | Superseded]  
**Date:** YYYY-MM-DD  
**Deciders:** [Nombres]  
**Consulted:** [Teams]  

## Context
[Describe el problema o situación que requiere una decisión]

## Decision
[Qué se decidió hacer]

## Consequences

### Positive
- [Beneficio 1]
- [Beneficio 2]

### Negative
- [Trade-off 1]
- [Trade-off 2]

### Risks
- [Riesgo 1 + mitigation]

## Alternatives Considered
1. **[Opción A]:** [Por qué se descartó]
2. **[Opción B]:** [Por qué se descartó]

## Compliance Impact
- **GDPR:** [Impacto si aplica]
- **SOC2:** [Impacto si aplica]

## Links
- [Related ADRs]
- [Technical specs]
```

---

#### **Plantilla 3: Runbook Operacional**

```markdown
# Runbook: [Operación]

## 📋 Overview
**Purpose:** [Qué hace esta operación]  
**Frequency:** [Daily/Weekly/On-demand]  
**Duration:** [Tiempo estimado]  
**Risk Level:** [Low/Medium/High]

## ✅ Pre-requisites
- [ ] Access to [sistema]
- [ ] Approval from [quien]
- [ ] Change ticket: [link]

## 🔧 Steps

### Step 1: [Acción]
```bash
# Comando ejemplo
az webapp restart --name myapp --resource-group myRG
```

**Expected Output:**
```
Restarting webapp...
Done
```

**Troubleshooting:**
- If error X: Do Y

### Step 2: [Verificación]
[...]

## 🔄 Rollback Procedure
[Cómo deshacer si algo sale mal]

## 📊 Validation
- [ ] Check dashboard: [URL]
- [ ] Verify logs show no errors
- [ ] Confirm with monitoring team

## 📞 Escalation
If issues persist > 15 min:
1. Contact: [Persona] via [método]
2. Escalate to: [Manager]
```

---

#### **Plantilla 4: Post-Mortem (Blameless)**

```markdown
# Post-Mortem: [Título del Incidente]

**Date:** YYYY-MM-DD  
**Severity:** [SEV1/SEV2/SEV3]  
**Duration:** [HH:MM]  
**Impacted Services:** [Lista]  
**Incident Commander:** [Nombre]

## 📝 Executive Summary
[2-3 párrafos describiendo qué pasó y el impacto]

## 📊 Impact
- **Users Affected:** [número/porcentaje]
- **Revenue Impact:** $[cantidad]
- **Downtime:** [minutos]
- **Error Budget Consumed:** [X%]

## ⏱️ Timeline (UTC)
| Time | Event |
|------|-------|
| 14:23 | Alert fired: High error rate |
| 14:25 | On-call engineer paged |
| 14:30 | Incident declared SEV2 |
| 14:45 | Root cause identified |
| 15:10 | Mitigation deployed |
| 15:15 | Service restored |

## 🔍 Root Cause
[Explicación técnica detallada]

**Contributing Factors:**
- [Factor 1]
- [Factor 2]

## 🛠️ Resolution
[Qué se hizo para resolver]

## 📋 Action Items

| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| [Acción 1] | [Persona] | YYYY-MM-DD | ⏳ In Progress |
| [Acción 2] | [Persona] | YYYY-MM-DD | ✅ Done |

## 📚 Lessons Learned
**What Went Well:**
- [Positivo 1]

**What Went Wrong:**
- [Problema 1]

**Where We Got Lucky:**
- [Suerte 1]

## 🔄 Related Incidents
- [Link a incident similar]
```

---

#### **Plantilla 5: SLO Definition**

```markdown
# SLO: [Nombre del Servicio]

## Service Level Indicators (SLIs)

### 1. Availability
**Definition:** Percentage of successful requests  
**Measurement:** 
```kusto
requests
| where success == true
| summarize SuccessRate = count() * 100.0 / total_requests
```
**SLO Target:** 99.9%  
**Error Budget:** 43.2 minutes/month

### 2. Latency
**Definition:** 95th percentile response time  
**Measurement:**
```kusto
requests
| summarize percentile(duration, 95)
```
**SLO Target:** < 200ms  
**Error Budget:** N/A (threshold-based)

## Error Budget Policy
- **100-90% remaining:** All changes allowed
- **90-50% remaining:** Review risky changes
- **50-0% remaining:** Feature freeze, only reliability work
- **0% remaining:** Incident postmortem required

## Alert Configuration
```yaml
alert: SLO_Availability_Breach
expr: availability < 99.9 AND error_budget_remaining < 50
severity: P1
```

## Review Schedule
- **Weekly:** SRE team reviews burn rate
- **Monthly:** Leadership review of SLO compliance
- **Quarterly:** Adjust SLOs based on business needs
```

---

### 3.3 Single Point of Truth (SPOT)

**Propuesta:** Azure DevOps Wiki + Repos

```
Azure DevOps Organization
├── Project: Platform-Docs (SPOT)
│   ├── Wiki: Knowledge Base
│   │   ├── Home (Index de todos los servicios)
│   │   ├── Architecture Standards
│   │   ├── Operational Procedures
│   │   └── On-Call Playbooks
│   ├── Repos: 
│   │   ├── docs-central (repo con todas las docs)
│   │   └── templates (repo con plantillas)
│   └── Boards:
│       └── Documentation Backlog
└── Projects: [Service-specific]
    └── Each project links back to central docs
```

**Ventajas:**
- ✅ Búsqueda centralizada
- ✅ Control de versiones
- ✅ Revisión por Pull Request
- ✅ Integración con CI/CD
- ✅ Permisos granulares

---

## 🔧 4. Implementación con Azure DevOps {#azure-devops}

### 4.1 Arquitectura de Documentación en Azure DevOps

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure DevOps Organization                 │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Project: SPOT-Documentation (Central)                 │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ • Wiki (Published from Git)                           │  │
│  │   ├─ /standards (plantillas + guías)                  │  │
│  │   ├─ /services (index de todos los servicios)        │  │
│  │   └─ /runbooks (procedimientos comunes)              │  │
│  │                                                         │  │
│  │ • Repo: documentation-central                          │  │
│  │   ├─ templates/                                        │  │
│  │   ├─ standards/                                        │  │
│  │   └─ .azure-pipelines/                                │  │
│  │       └─ validate-docs.yml (linting)                  │  │
│  │                                                         │  │
│  │ • Boards: Documentation Backlog                        │  │
│  │   ├─ Epic: "Migrate Service X docs"                   │  │
│  │   └─ Sprint planning for doc updates                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Project: Service-Payment (Example)                    │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ • Repo: payment-service                                │  │
│  │   ├─ /docs (docs específicas del servicio)            │  │
│  │   │   ├─ README.md → links to SPOT                    │  │
│  │   │   ├─ architecture/                                 │  │
│  │   │   ├─ runbooks/                                     │  │
│  │   │   └─ ADRs/                                         │  │
│  │   ├─ /infrastructure (IaC)                             │  │
│  │   └─ azure-pipelines.yml                               │  │
│  │                                                         │  │
│  │ • Wiki: Service-specific docs (optional)               │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  [Más services...]                                           │
└─────────────────────────────────────────────────────────────┘

External Integrations:
├─ Azure Monitor (Dashboards, Alerts)
├─ Application Insights (Logs, Traces)
├─ PagerDuty/Opsgenie (On-call)
└─ Slack (Notifications)
```

---

### 4.2 Setup Paso a Paso

#### **Fase 1: Crear el SPOT (Semana 1-2)**

```bash
# 1. Crear proyecto central
az devops project create \
  --name "SPOT-Documentation" \
  --description "Single Point of Truth for all documentation" \
  --visibility private

# 2. Crear repo central
az repos create \
  --name "documentation-central" \
  --project "SPOT-Documentation"

# 3. Estructura inicial
cd documentation-central
mkdir -p templates/{service,runbook,adr,postmortem}
mkdir -p standards/{architecture,operations,compliance}
mkdir -p services

# 4. Copiar plantillas
cp /path/to/templates/* templates/

# 5. Crear Wiki desde Git
az devops wiki create \
  --name "Knowledge Base" \
  --type projectwiki \
  --mapped-path / \
  --repository documentation-central \
  --project "SPOT-Documentation"
```

#### **Fase 2: Pipeline de Validación Automática**

Crear `azure-pipelines/validate-docs.yml`:

```yaml
# Pipeline que valida documentación en cada PR
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - docs/**
    - templates/**

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Validate
  jobs:
  - job: LintMarkdown
    steps:
    - task: UseNode@2
      inputs:
        versionSpec: '18.x'
    
    - script: |
        npm install -g markdownlint-cli
        markdownlint '**/*.md' --ignore node_modules
      displayName: 'Markdown Linting'
    
    - script: |
        # Validar que existen secciones obligatorias
        ./scripts/validate-readme.sh
      displayName: 'Validate README Structure'

  - job: CheckLinks
    steps:
    - script: |
        npm install -g markdown-link-check
        find . -name "*.md" -exec markdown-link-check {} \;
      displayName: 'Check Broken Links'

  - job: ValidateADRs
    steps:
    - script: |
        # Validar que ADRs siguen el template
        python scripts/validate-adr-format.py
      displayName: 'Validate ADR Format'

- stage: Publish
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - job: PublishToWiki
    steps:
    - task: PublishToWiki@1
      inputs:
        repository: 'documentation-central'
        branch: 'main'
```

#### **Fase 3: Scripts de Automatización**

**Script 1: Generar índice de servicios automáticamente**

`scripts/generate-service-index.py`:
```python
#!/usr/bin/env python3
"""
Genera automáticamente un índice de todos los servicios
leyendo los README.md de cada proyecto
"""
import os
import re
from azure.devops.connection import Connection
from msrest.authentication import BasicAuthentication

# Configuración
ORG_URL = "https://dev.azure.com/your-org"
PAT = os.environ['AZURE_DEVOPS_PAT']

credentials = BasicAuthentication('', PAT)
connection = Connection(base_url=ORG_URL, creds=credentials)

# Obtener todos los proyectos
core_client = connection.clients.get_core_client()
projects = core_client.get_projects()

services = []

for project in projects:
    # Buscar README.md en el repo
    try:
        git_client = connection.clients.get_git_client()
        repos = git_client.get_repositories(project.id)
        
        for repo in repos:
            try:
                item = git_client.get_item(
                    repository_id=repo.id,
                    path='/README.md',
                    project=project.id
                )
                
                # Extraer metadata del README
                content = git_client.get_item_text(
                    repository_id=repo.id,
                    path='/README.md',
                    project=project.id
                )
                
                # Parse para extraer owner, on-call, etc.
                owner = re.search(r'\*\*Owner Team:\*\* (.+)', content)
                slo = re.search(r'Availability.*?(\d+\.\d+%)', content)
                
                services.append({
                    'name': project.name,
                    'repo': repo.name,
                    'owner': owner.group(1) if owner else 'Unknown',
                    'slo': slo.group(1) if slo else 'N/A',
                    'url': f"{ORG_URL}/{project.name}"
                })
            except:
                continue
    except:
        continue

# Generar el índice en Markdown
with open('services/INDEX.md', 'w') as f:
    f.write('# Service Catalog\n\n')
    f.write('> Auto-generated index of all services\n\n')
    f.write('| Service | Owner | SLO | Links |\n')
    f.write('|---------|-------|-----|-------|\n')
    
    for svc in sorted(services, key=lambda x: x['name']):
        f.write(f"| [{svc['name']}]({svc['url']}) | {svc['owner']} | {svc['slo']} | "
                f"[Docs]({svc['url']}/_git/{svc['repo']}?path=/docs) |\n")

print(f"✅ Generated index with {len(services)} services")
```

**Script 2: Validar estructura de README**

`scripts/validate-readme.sh`:
```bash
#!/bin/bash
# Valida que README.md tiene las secciones obligatorias

check_section() {
    local file=$1
    local section=$2
    
    if ! grep -q "## $section" "$file"; then
        echo "❌ Missing section: $section in $file"
        exit 1
    fi
}

for readme in $(find . -name "README.md" -not -path "./node_modules/*"); do
    echo "Checking $readme..."
    
    check_section "$readme" "Quick Info"
    check_section "$readme" "Purpose"
    check_section "$readme" "Architecture"
    check_section "$readme" "SLOs"
    check_section "$readme" "Key Links"
    
    echo "✅ $readme is valid"
done
```

---

### 4.3 Integración con Azure Monitor y Alerting

**Script 3: Auto-generar documentación de SLOs desde Azure Monitor**

`scripts/generate-slo-docs.py`:
```python
"""
Extrae SLOs configurados en Azure Monitor y genera docs automáticamente
"""
from azure.monitor.query import LogsQueryClient, MetricsQueryClient
from azure.identity import DefaultAzureCredential
import yaml

credential = DefaultAzureCredential()
logs_client = LogsQueryClient(credential)

# Query para obtener availability SLI
query = """
requests
| where timestamp > ago(30d)
| summarize 
    total_requests = count(),
    failed_requests = countif(success == false)
| extend availability = (total_requests - failed_requests) * 100.0 / total_requests
| project availability
"""

result = logs_client.query_workspace(
    workspace_id="<workspace-id>",
    query=query,
    timespan=timedelta(days=30)
)

availability = result.tables[0].rows[0][0]
slo_target = 99.9
error_budget_remaining = (availability - slo_target) / (100 - slo_target) * 100

# Generar documento
slo_doc = f"""
# SLO Report - {datetime.now().strftime('%Y-%m-%d')}

## Current Status
- **Availability (30d):** {availability:.3f}%
- **SLO Target:** {slo_target}%
- **Status:** {'✅ Meeting SLO' if availability >= slo_target else '❌ Breaching SLO'}
- **Error Budget Remaining:** {error_budget_remaining:.1f}%

## Trend
[Graph auto-generated]

## Actions Required
{'⚠️ Feature freeze until SLO is restored' if error_budget_remaining < 0 else 'No action needed'}
"""

with open('docs/slos/current-status.md', 'w') as f:
    f.write(slo_doc)
```

---

### 4.4 Automatización de Alertas → Documentación

**Configurar Azure Monitor Alert Action Group para crear Issues**

```yaml
# alert-to-issue.yml - Azure Logic App
trigger:
  - type: "When an Azure Monitor alert is triggered"

actions:
  - name: "Create Work Item"
    type: "Azure DevOps"
    inputs:
      organization: "your-org"
      project: "SPOT-Documentation"
      work_item_type: "Task"
      title: "Update runbook for: {alert_name}"
      description: |
        An alert was triggered that may indicate missing documentation:
        
        Alert: {alert_name}
        Severity: {alert_severity}
        Resource: {resource_id}
        
        Please update the runbook:
        - [ ] Add troubleshooting steps for this alert
        - [ ] Update SLO documentation if applicable
        - [ ] Review escalation procedures
      assigned_to: "{alert_owner}"
      tags: "documentation,runbook,auto-generated"
```

---

### 4.5 Integración On-Call con PagerDuty

**Sincronizar rotación de On-Call con Azure DevOps Wiki**

`scripts/sync-oncall-schedule.py`:
```python
"""
Sincroniza el schedule de PagerDuty con Azure DevOps Wiki
para que la documentación siempre refleje quién está on-call
"""
import requests
from datetime import datetime, timedelta

PAGERDUTY_API_KEY = os.environ['PD_API_KEY']
SCHEDULE_ID = 'P123ABC'

headers = {
    'Authorization': f'Token token={PAGERDUTY_API_KEY}',
    'Accept': 'application/vnd.pagerduty+json;version=2'
}

# Obtener on-call actual y próximos
response = requests.get(
    f'https://api.pagerduty.com/schedules/{SCHEDULE_ID}/users',
    headers=headers,
    params={'since': datetime.now().isoformat(), 'until': (datetime.now() + timedelta(days=14)).isoformat()}
)

oncalls = response.json()['users']

# Generar página de Wiki
wiki_content = f"""
# On-Call Schedule

Last updated: {datetime.now().strftime('%Y-%m-%d %H:%M UTC')}

## Current On-Call
**{oncalls[0]['summary']}**
- 📱 Phone: {oncalls[0].get('contact_methods', [{}])[0].get('address', 'N/A')}
- 📧 Email: {oncalls[0]['email']}
- ⏰ Shift ends: {oncalls[0]['end']}

## Next 2 Weeks
| Date | Engineer | Contact |
|------|----------|---------|
"""

for oncall in oncalls[1:]:
    start = datetime.fromisoformat(oncall['start'].replace('Z', '+00:00'))
    wiki_content += f"| {start.strftime('%Y-%m-%d')} | {oncall['summary']} | {oncall['email']} |\n"

# Actualizar Wiki via Git
# (código para commit a repo documentation-central)
```

---

## 📋 5. Proceso de Documentación {#proceso}

### 5.1 Workflow para Nuevos Servicios

```
┌─────────────────────────────────────────────────────────────┐
│                  New Service Checklist                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ✅ FASE 1: Planificación (Semana -2)                        │
│  ┌──────────────────────────────────────────────┐           │
│  │ 1. Crear ADR con decisión de arquitectura    │           │
│  │ 2. Definir SLOs preliminares                │           │
│  │ 3. Identificar dependencies                  │           │
│  │ 4. Mapear requisitos compliance              │           │
│  └──────────────────────────────────────────────┘           │
│                                                               │
│  ✅ FASE 2: Implementación (Semana 0-4)                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ 1. IaC documented (Terraform con comments)   │           │
│  │ 2. Deployment pipeline creado                │           │
│  │ 3. Monitoring configurado                    │           │
│  │ 4. Alerts definidas (SLO-based)             │           │
│  └──────────────────────────────────────────────┘           │
│                                                               │
│  ✅ FASE 3: Pre-Production (Semana 5)                        │
│  ┌──────────────────────────────────────────────┐           │
│  │ 1. README.md completo                        │           │
│  │ 2. Runbooks operacionales (deploy, rollback) │           │
│  │ 3. On-call playbook                          │           │
│  │ 4. Disaster recovery plan                    │           │
│  │ 5. RACI definido                             │           │
│  └──────────────────────────────────────────────┘           │
│                                                               │
│  ✅ FASE 4: Production Launch (Semana 6)                     │
│  ┌──────────────────────────────────────────────┐           │
│  │ 1. Gate check: Docs completas al 100%        │           │
│  │ 2. On-call rotation configured                │           │
│  │ 3. Service registrado en SPOT catalog        │           │
│  │ 4. Equipo entrenado en runbooks              │           │
│  └──────────────────────────────────────────────┘           │
│                                                               │
│  🚫 NO DEPLOYMENT WITHOUT DOCS                               │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

**Policy enforcement:** Azure Pipeline gate

```yaml
# En el deployment pipeline
- task: CheckDocumentation@1
  inputs:
    validateReadme: true
    validateRunbooks: true
    validateSLOs: true
    failOnMissing: true  # ❌ Deployment falla si docs incompletas
```

---

### 5.2 Workflow para Actualización de Docs

```
┌──────────────────────────────────────────────┐
│  Trigger para Actualizar Documentación      │
├──────────────────────────────────────────────┤
│                                              │
│  1. Change Management                        │
│     └─> Major change → ADR requerido        │
│     └─> New feature → Update README         │
│                                              │
│  2. Incident Response                        │
│     └─> Post-mortem → Update runbook        │
│     └─> New failure mode → Update playbook  │
│                                              │
│  3. SLO Review (Quarterly)                   │
│     └─> Adjust SLOs → Update docs           │
│     └─> New metrics → Update dashboards     │
│                                              │
│  4. Compliance Audit                         │
│     └─> Gap found → Update compliance docs  │
│                                              │
│  5. Dependency Change                        │
│     └─> Update architecture diagram         │
│     └─> Update runbooks with new endpoints  │
│                                              │
└──────────────────────────────────────────────┘

Process:
1. Create Branch: feature/update-docs-incident-123
2. Make Changes to /docs
3. Create Pull Request
4. Automated Validation (CI pipeline)
5. Peer Review (2 approvers required)
6. Merge → Auto-publish to Wiki
7. Notify stakeholders (Slack/Teams)
```

---

### 5.3 Mantenimiento Continuo (Docs as Living Documents)

#### **Automatización de Freshness Checks**

```yaml
# Scheduled pipeline que corre semanalmente
schedules:
- cron: "0 9 * * MON"  # Cada lunes 9 AM
  branches:
    include:
    - main

jobs:
- job: CheckDocumentationFreshness
  steps:
  - script: |
      # Encuentra docs no actualizadas en 90 días
      find docs/ -name "*.md" -mtime +90 -print > stale-docs.txt
      
      if [ -s stale-docs.txt ]; then
        echo "⚠️ Found stale documentation:"
        cat stale-docs.txt
        
        # Crear work items para cada doc obsoleta
        while read doc; do
          az boards work-item create \
            --title "Review stale documentation: $doc" \
            --type "Task" \
            --project "SPOT-Documentation" \
            --assigned-to "@<Relationship:DocumentOwner>" \
            --description "This document hasn't been updated in 90 days. Please review and update."
        done < stale-docs.txt
      fi
    displayName: 'Check for stale documentation'

- job: ValidateLinksFreshness
  steps:
  - script: |
      # Validar que links internos siguen válidos
      markdown-link-check docs/**/*.md --config link-check-config.json
    displayName: 'Validate documentation links'

- job: SyncWithProduction
  steps:
  - script: |
      # Comparar configuración documentada vs real
      ./scripts/validate-docs-vs-prod.sh
    displayName: 'Validate docs match production'
```

---

### 5.4 Roles y Responsabilidades en el Proceso

| Rol | Responsabilidad | Frecuencia |
|-----|----------------|------------|
| **Service Owner** | Crear/actualizar docs de su servicio | On demand |
| **SRE Team** | Mantener standards y templates | Mensual |
| **On-Call Engineer** | Actualizar runbooks después de incidentes | After cada incident |
| **Architect** | Revisar ADRs y arquitectura | Trimestral |
| **Compliance Officer** | Validar docs de compliance | Anual |
| **Documentation Champion** (nuevo rol) | Coordinar proceso, métricas | Semanal |

**Nuevo rol propuesto: Documentation Champion**
- Dedicación: 20% de un SRE senior
- Responsabilidades:
  - Onboarding de equipos al proceso
  - Mantener templates actualizados
  - Generar reportes de coverage
  - Automatizar lo automatizable
  - Evangelizar buenas prácticas

---

## 🚀 6. Plan de Implementación {#implementación}

### 6.1 Roadmap de 6 Meses

```
┌───────────────────────────────────────────────────────────────────────┐
│                       Implementation Roadmap                           │
├───────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  MONTH 1: Setup & Pilot                                               │
│  ├─ Week 1-2: Setup SPOT en Azure DevOps                             │
│  │   ├─ Crear proyecto central                                        │
│  │   ├─ Configurar Wiki                                               │
│  │   ├─ Subir templates                                               │
│  │   └─ Configurar pipelines de validación                            │
│  │                                                                     │
│  ├─ Week 3-4: Piloto con 2 servicios críticos                        │
│  │   ├─ Migrar docs de Service A y B                                  │
│  │   ├─ Training del equipo                                           │
│  │   └─ Recoger feedback                                              │
│  └─ ✅ Deliverable: SPOT funcional + 2 servicios documentados        │
│                                                                         │
│  MONTH 2: Scale Phase 1                                               │
│  ├─ Migrar 10 servicios más (12 total)                               │
│  ├─ Iterar templates basado en feedback                               │
│  ├─ Configurar integraciones (PagerDuty, Azure Monitor)               │
│  └─ ✅ Deliverable: 12 servicios con docs completas                  │
│                                                                         │
│  MONTH 3: Automation                                                   │
│  ├─ Implementar pipelines de auto-generación                          │
│  ├─ Freshness checks automáticos                                      │
│  ├─ Deployment gates (no deploy sin docs)                             │
│  └─ ✅ Deliverable: Automatización 80% completa                      │
│                                                                         │
│  MONTH 4: Scale Phase 2                                               │
│  ├─ Migrar resto de servicios críticos (SEV1/SEV2)                   │
│  ├─ Training masivo a todos los equipos                               │
│  ├─ Establecer Documentation Champions                                │
│  └─ ✅ Deliverable: 50+ servicios documentados                       │
│                                                                         │
│  MONTH 5: Compliance & Governance                                      │
│  ├─ Documentar compliance para GDPR, SOC2, etc.                       │
│  ├─ Auditoría de completeness                                         │
│  ├─ Políticas de enforcement                                          │
│  └─ ✅ Deliverable: Compliance docs + governance framework           │
│                                                                         │
│  MONTH 6: Optimization & Long-term                                     │
│  ├─ Analizar métricas de adopción                                     │
│  ├─ Optimizar proceso basado en datos                                 │
│  ├─ Roadmap de mejoras continuas                                      │
│  └─ ✅ Deliverable: Plan de sustainability                           │
│                                                                         │
└───────────────────────────────────────────────────────────────────────┘
```

---

### 6.2 Plan Detallado - Mes 1 (Pilot)

#### **Semana 1: Setup Infraestructura**

**Day 1-2: Azure DevOps Configuration**
```bash
# Script de setup automático
#!/bin/bash

# 1. Crear organización/project
az devops project create --name "SPOT-Documentation"

# 2. Crear repos
az repos create --name "documentation-central"
az repos create --name "documentation-templates"

# 3. Importar templates
git clone <your-templates-repo>
cd documentation-templates
git init
git remote add origin https://dev.azure.com/yourorg/documentation-templates
git push -u origin main

# 4. Configurar Wiki
az devops wiki create \
  --name "Knowledge Base" \
  --type projectwiki \
  --mapped-path /

# 5. Configurar permisos
az devops security permission update \
  --allow-bit 8 \  # Contribute
  --group "Contributors"

# 6. Crear pipelines
az pipelines create \
  --name "Validate-Documentation" \
  --yaml-path /azure-pipelines/validate-docs.yml
```

**Day 3-4: Templates y Standards**
- Subir plantillas al repo
- Crear guía de onboarding
- Documentar el proceso nuevo

**Day 5: Testing interno**
- Probar todo el flujo con un servicio dummy
- Validar pipelines funcionan
- Ajustar

#### **Semana 2: Piloto - Servicios Críticos**

**Selección de servicios piloto:**
1. **Service A:** Servicio crítico con docs existentes (para migración)
2. **Service B:** Servicio nuevo (para crear desde cero)

**Criterios de selección:**
- Criticidad: SEV1 o SEV2
- Team willing: Equipo motivado
- Complexity: Representativo (no trivial, no ultra-complejo)

**Activities:**
```
Monday:
  - Kickoff con equipos piloto
  - Workshop: "New documentation framework"
  - Assign: Doc champion por servicio

Tuesday-Wednesday:
  - Equipos trabajan en migrar/crear docs
  - SRE team disponible para consultas
  - Iteración en templates según feedback

Thursday:
  - Review de docs creadas
  - Pull requests + peer review
  - Ajustes

Friday:
  - Merge a main
  - Publicación en Wiki
  - Retrospective
  - 📊 Métricas: Time spent, feedback score
```

#### **Semana 3-4: Refinamiento**

- Iterar templates basado en learnings
- Crear FAQ de preguntas comunes
- Grabar video tutorials
- Preparar para scale-up

---

### 6.3 Change Management

#### **Estrategia de Adopción**

**Champions Network:**
```
1. Identificar 1 champion por equipo (10-15 personas)
2. Training intensivo de champions (2 días)
3. Champions entrenan a sus equipos
4. Champions son el punto de contacto
```

**Communication Plan:**

| Week | Action | Audience | Medium |
|------|--------|----------|--------|
| -2 | Anuncio del proyecto | All Eng | Email + Town Hall |
| 0 | Kickoff piloto | Pilot teams | Workshop |
| 4 | Demo de resultados piloto | All Eng | Demo session |
| 8 | Anuncio de rollout general | All Eng | Email |
| 12 | Mid-point review | Leadership | Presentation |
| 24 | Success stories | All Eng | Blog post |

**Incentivos:**

- 🏆 **Documentation Quality Awards** (trimestral)
  - Best architected docs
  - Best runbooks
  - Most improved service docs
  
- 📊 **Public Dashboard** de metrics por equipo
  - Healthy competition
  - Visibility de progreso

- 💰 **KPI de documentación** en performance reviews
  - "Did you maintain up-to-date documentation?"
  - Peso: 10% de eng performance

---

### 6.4 Risk Mitigation

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| **Resistencia al cambio** | Alta | Alto | Champions network, incentivos, leadership buy-in |
| **Docs quedan obsoletas rápido** | Alta | Alto | Automation, freshness checks, deployment gates |
| **Overhead en development** | Media | Medio | Templates ultra-simples, automation, clear minimum |
| **Fragmentación (equipos no adoptan)** | Media | Alto | Enforcement policies, make it easy, remove alternatives |
| **Falta de recursos** | Alta | Alto | Dedicar 1 FTE como Doc Champion, automation reduce load |
| **Compliance gaps** | Baja | Alto | Early involvement de Compliance team, templates incluyen requirements |

---

## 📊 7. Métricas y KPIs {#métricas}

### 7.1 Métricas de Adopción

```
┌────────────────────────────────────────────────────────┐
│            Documentation Maturity Model                 │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Level 0: Tribal Knowledge (0-20%)                     │
│  ├─ No documentation                                    │
│  ├─ Knowledge in people's heads                         │
│  └─ ⚠️ Riesgo: Single points of failure                │
│                                                          │
│  Level 1: Ad-hoc Docs (20-40%)                         │
│  ├─ Some README exists                                  │
│  ├─ Scattered across wikis, emails, Confluence          │
│  └─ ⚠️ Riesgo: Outdated, hard to find                  │
│                                                          │
│  Level 2: Structured Docs (40-60%)                     │
│  ├─ Docs follow templates                               │
│  ├─ Centralized in SPOT                                 │
│  └─ ⚠️ Still some gaps                                  │
│                                                          │
│  Level 3: Living Docs (60-80%)                         │
│  ├─ Docs auto-updated from code/infra                   │
│  ├─ Regular review process                              │
│  └─ ✅ Good state                                       │
│                                                          │
│  Level 4: Documentation as Culture (80-100%)           │
│  ├─ Docs part of Definition of Done                     │
│  ├─ Can't deploy without docs                           │
│  ├─ Automated freshness checks                          │
│  └─ 🏆 Target state                                     │
│                                                          │
└────────────────────────────────────────────────────────┘
```

**KPI Dashboard (Azure DevOps Widgets):**

```yaml
# Widget 1: Documentation Coverage
metrics:
  - name: Services Documented
    query: COUNT(services WITH complete_docs)
    target: 100%
    current: 45%
    trend: +15% MoM

# Widget 2: Documentation Freshness
metrics:
  - name: Docs Updated Last 30 Days
    query: COUNT(docs WHERE last_modified < 30 days)
    target: 90%
    current: 67%

# Widget 3: Compliance
metrics:
  - name: Services Compliant
    query: COUNT(services WHERE compliance_docs_complete)
    target: 100%
    current: 78%

# Widget 4: Quality Score
metrics:
  - name: Avg Doc Quality Score
    description: Based on completeness, freshness, links working
    target: 8.5/10
    current: 7.2/10
```

---

### 7.2 Quality Metrics

**Doc Quality Score (0-10):**

```python
def calculate_doc_quality_score(service_docs):
    """
    Calcula un score de calidad de documentación
    """
    score = 0
    max_score = 10
    
    # 1. Completeness (4 pts)
    required_sections = [
        'README.md',
        'docs/architecture/overview.md',
        'docs/operations/runbooks/deployment.md',
        'docs/operations/monitoring/slos.md'
    ]
    completeness = sum(1 for doc in required_sections if exists(doc)) / len(required_sections)
    score += completeness * 4
    
    # 2. Freshness (2 pts)
    age_days = days_since_last_update(service_docs)
    if age_days < 30:
        score += 2
    elif age_days < 90:
        score += 1
    # else: 0 pts
    
    # 3. Link Health (1 pt)
    broken_links = count_broken_links(service_docs)
    if broken_links == 0:
        score += 1
    
    # 4. ADR Coverage (1 pt)
    if count_adrs(service_docs) >= 3:
        score += 1
    
    # 5. Runbook Quality (1 pt)
    runbooks = get_runbooks(service_docs)
    if all(has_troubleshooting(rb) and has_rollback(rb) for rb in runbooks):
        score += 1
    
    # 6. SLO Defined (1 pt)
    if has_slos_defined(service_docs):
        score += 1
    
    return round(score, 1)
```

---

### 7.3 Business Impact Metrics

**Correlación con Métricas de Negocio:**

```sql
-- Query para analizar impacto de buena documentación

WITH incident_mttr AS (
  SELECT 
    service_name,
    AVG(time_to_resolution) as avg_mttr_hours,
    COUNT(*) as incident_count
  FROM incidents
  WHERE created_date > DATEADD(month, -3, GETDATE())
  GROUP BY service_name
),
doc_quality AS (
  SELECT
    service_name,
    doc_quality_score,
    has_complete_runbooks
  FROM documentation_metrics
)
SELECT 
  i.service_name,
  i.avg_mttr_hours,
  d.doc_quality_score,
  -- Correlación: Mejor docs = Menor MTTR
  CASE 
    WHEN d.doc_quality_score > 8 THEN 'High Quality Docs'
    WHEN d.doc_quality_score > 6 THEN 'Medium Quality Docs'
    ELSE 'Low Quality Docs'
  END as doc_tier
FROM incident_mttr i
JOIN doc_quality d ON i.service_name = d.service_name
ORDER BY avg_mttr_hours ASC
```

**Expected Results:**
```
Service A | 0.5h MTTR | 9.2 score | High Quality Docs  ✅
Service B | 1.2h MTTR | 7.8 score | Medium Quality Docs
Service C | 4.5h MTTR | 4.1 score | Low Quality Docs   ❌

Hypothesis: Good docs → 75% faster incident resolution
```

---

### 7.4 Dashboard de Reporting

**Executive Dashboard (PowerBI + Azure DevOps):**

```
┌─────────────────────────────────────────────────────────┐
│         Documentation Health - Executive View           │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  📊 Overall Maturity: Level 3 (Living Docs)             │
│  Progress: ▓▓▓▓▓▓▓▓▓░░ 78%                              │
│                                                           │
│  🎯 Key Metrics                                          │
│  ├─ Services Documented: 78/100 ✅ (+18 vs last Q)     │
│  ├─ Avg Doc Quality: 7.2/10 ⚠️ (Target: 8.5)          │
│  ├─ Docs < 30d old: 67% ⚠️ (Target: 90%)              │
│  └─ Compliance Complete: 78/78 ✅                       │
│                                                           │
│  📈 Business Impact                                      │
│  ├─ MTTR Reduction: -35% (correlated with docs)         │
│  ├─ Onboarding Time: -50% (new engineers)               │
│  └─ Incident Escalations: -40% (on-call resolved)       │
│                                                           │
│  ⚠️ Action Items                                         │
│  ├─ 12 services need doc refresh (>90 days old)         │
│  ├─ 5 services missing SLO documentation                 │
│  └─ 3 teams below quality threshold (score < 6)          │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

**Engineer Dashboard (Azure DevOps Wiki - Auto-generated):**

```markdown
# My Services Documentation Status

## Services I Own

| Service | Docs Quality | Last Updated | Action Required |
|---------|--------------|--------------|-----------------|
| Payment API | 8.5/10 ✅ | 5 days ago | None |
| Auth Service | 6.2/10 ⚠️ | 120 days ago | 🔴 **Update runbooks** |
| Notification | 9.1/10 ✅ | 2 days ago | None |

## Pending Tasks
- [ ] Update Auth Service runbooks (overdue 30 days)
- [ ] Review Payment API SLOs (quarterly review due)
- [ ] Add disaster recovery plan to Notification service

## Recent Activity
- ✅ Merged PR #456: Update deployment runbook
- 📝 Created ADR-023: Switch to Cosmos DB
- 🔍 Participated in Payment API post-mortem
```

---

## 📚 8. Anexos {#anexos}

### Anexo A: Checklist Pre-Production

```markdown
# Pre-Production Documentation Checklist

Service Name: ___________________
Owner: ___________________
Target Launch Date: ___________________

## 🏗️ Architecture
- [ ] README.md created with Quick Info, Purpose, Architecture
- [ ] C4 Context diagram created
- [ ] C4 Container diagram created (if complex)
- [ ] Data flow diagram created
- [ ] Dependencies documented
- [ ] Security architecture reviewed
- [ ] At least 3 ADRs created for major decisions

## 📊 SRE & Operations
- [ ] SLIs defined and measurable
- [ ] SLOs defined with business buy-in
- [ ] Error budget calculated
- [ ] Monitoring dashboards created
- [ ] Alerts configured (SLO-based, not threshold-based)
- [ ] On-call playbook created
- [ ] Runbook: Deployment procedure
- [ ] Runbook: Rollback procedure
- [ ] Runbook: Common troubleshooting
- [ ] Disaster recovery plan documented
- [ ] Backup/restore procedures tested
- [ ] Capacity planning documented

## 👥 Team & Process
- [ ] RACI matrix defined
- [ ] On-call rotation configured (PagerDuty/Opsgenie)
- [ ] Team trained on runbooks
- [ ] Escalation matrix defined
- [ ] Incident severity levels defined
- [ ] Post-mortem template prepared

## 🔒 Compliance & Security
- [ ] GDPR compliance documented (if handles PII)
- [ ] Data retention policies documented
- [ ] Security controls documented
- [ ] Access control matrix defined
- [ ] Audit logging configured

## ✅ Validation
- [ ] All docs reviewed by peer
- [ ] Docs validated by SRE team
- [ ] Runbooks tested in staging
- [ ] Deployment gate passes (automated check)
- [ ] Service registered in SPOT catalog

---

**Sign-off:**

Service Owner: _______________ Date: ___________
SRE Lead: _______________ Date: ___________
Security Lead: _______________ Date: ___________ (if required)

**Result:** [ ] APPROVED FOR PRODUCTION | [ ] NEEDS WORK
```

---

### Anexo B: Quick Reference - Azure DevOps CLI Commands

```bash
# ========================================
# Azure DevOps CLI - Documentation Cheatsheet
# ========================================

# 1. CREATE PROJECT
az devops project create \
  --name "MyService" \
  --description "Payment processing service" \
  --source-control git \
  --visibility private

# 2. CREATE WIKI
az devops wiki create \
  --name "Service Documentation" \
  --type projectwiki \
  --project "MyService"

# 3. CREATE WORK ITEM (Doc Task)
az boards work-item create \
  --title "Update deployment runbook" \
  --type "Task" \
  --assigned-to "john@company.com" \
  --description "Runbook outdated, needs refresh" \
  --project "MyService"

# 4. QUERY WORK ITEMS
az boards query \
  --wiql "SELECT [System.Id], [System.Title] FROM WorkItems WHERE [System.WorkItemType] = 'Task' AND [System.Tags] CONTAINS 'documentation'" \
  --project "MyService"

# 5. CREATE PIPELINE
az pipelines create \
  --name "Validate-Documentation" \
  --description "Validates docs on every PR" \
  --repository "MyService" \
  --branch main \
  --yaml-path "/azure-pipelines/validate-docs.yml" \
  --project "MyService"

# 6. TRIGGER PIPELINE MANUALLY
az pipelines run \
  --name "Validate-Documentation" \
  --project "MyService"

# 7. LIST ALL WIKIS
az devops wiki list --project "MyService"

# 8. SHOW WIKI PAGE
az devops wiki page show \
  --wiki "Service Documentation" \
  --path "/runbooks/deployment" \
  --project "MyService"

# 9. CREATE WIKI PAGE
az devops wiki page create \
  --wiki "Service Documentation" \
  --path "/runbooks/new-procedure" \
  --content "# New Procedure\n\nSteps..." \
  --project "MyService"

# 10. LIST REPOS
az repos list --project "MyService"

# 11. SHOW REPO
az repos show \
  --repository "MyService" \
  --project "MyService"

# 12. SEARCH CODE (find docs)
az devops search code \
  --search-text "SLO" \
  --project "MyService"
```

---

### Anexo C: Template - Post-Mortem Meeting Agenda

```markdown
# Post-Mortem Meeting Agenda

**Incident:** [INC-12345] Payment API Outage  
**Date:** 2026-02-10  
**Duration:** 14:23 - 15:15 UTC (52 minutes)  
**Meeting Date:** 2026-02-12 10:00 UTC  
**Facilitator:** [Name] (SRE Lead)  
**Attendees:** Service owner, on-call engineer, stakeholders

---

## Meeting Ground Rules (5 min)
- ✅ **Blameless culture** - Focus on systems, not people
- ✅ **Objective facts** - Use timeline, data, logs
- ✅ **Forward-looking** - What can we improve?
- ❌ No blame, no finger-pointing
- ❌ No politics, no personal attacks

---

## Agenda

### 1. Timeline Review (15 min)
- Walk through incident timeline
- Clarify any gaps or uncertainties
- Questions from attendees

### 2. Root Cause Analysis (20 min)
- Technical root cause
- Contributing factors
- Why didn't we catch it earlier?

### 3. What Went Well (10 min)
- Praise good responses
- Effective processes/tools
- Individual actions that helped

### 4. What Went Wrong (15 min)
- Gaps in monitoring
- Gaps in documentation
- Process failures
- Communication breakdowns

### 5. Where We Got Lucky (5 min)
- What could have been worse?
- What prevented bigger impact?

### 6. Action Items (15 min)
- Concrete, actionable items
- Assign owners and due dates
- Prioritize: Critical → Important → Nice-to-have

### 7. Follow-up (5 min)
- Who writes the post-mortem doc?
- When is the next review?
- Communication to stakeholders

---

## Expected Outputs
- [ ] Completed post-mortem document
- [ ] Action items in Azure DevOps with owners
- [ ] Updated runbooks/docs
- [ ] Communication sent to affected teams

---

## Action Items Template

| ID | Action | Owner | Due Date | Priority | Status |
|----|--------|-------|----------|----------|--------|
| PM-001 | Add monitoring for X | SRE Team | 2026-02-20 | P0 | 🔴 Not Started |
| PM-002 | Update runbook with troubleshooting | Service Owner | 2026-02-15 | P1 | 🟡 In Progress |
| PM-003 | Review capacity planning | Architect | 2026-03-01 | P2 | 🔴 Not Started |
```

---

### Anexo D: Glossary

| Término | Definición |
|---------|------------|
| **ADR** | Architecture Decision Record - Documento que registra una decisión de arquitectura importante |
| **Error Budget** | Cantidad de errores "permitidos" sin violar SLO (e.g., 43 min downtime/mes para 99.9%) |
| **IaC** | Infrastructure as Code - Definir infraestructura en código (Terraform, ARM, Bicep) |
| **MTTR** | Mean Time To Resolution - Tiempo promedio para resolver un incidente |
| **Runbook** | Documento operacional con procedimientos paso a paso |
| **SLI** | Service Level Indicator - Métrica que medimos (e.g., latency, error rate) |
| **SLO** | Service Level Objective - Target para un SLI (e.g., 99.9% availability) |
| **SLA** | Service Level Agreement - Contrato legal con consecuencias (e.g., credits si <99.9%) |
| **SPOT** | Single Point of Truth - Lugar centralizado para toda la documentación |
| **C4 Model** | Context, Container, Component, Code - Framework para diagramas de arquitectura |

---

## 🎯 Conclusión y Próximos Pasos

### Resumen Ejecutivo

Este plan proporciona un **framework completo y práctico** para estandarizar la documentación de arquitectura IT/SRE en una organización multinacional, con:

✅ **Diagnóstico claro** de problemas actuales  
✅ **Revisión** de documentación vs. mejores prácticas modernas  
✅ **Framework estandarizado** con plantillas probadas  
✅ **Integración nativa** con Azure DevOps  
✅ **Automatización** para reducir overhead manual  
✅ **Plan de implementación** de 6 meses  
✅ **Métricas** para medir progreso y ROI  

### Beneficios Esperados

| Beneficio | Timeline | Métrica |
|-----------|----------|---------|
| Reducir MTTR | 3 meses | -35% en incidentes |
| Mejorar onboarding | 2 meses | -50% tiempo para nuevos engineers |
| Aumentar reliability | 6 meses | +15% en SLO compliance |
| Reducir escalaciones | 4 meses | -40% en pages a senior engineers |
| Compliance audit-ready | 5 meses | 100% de servicios compliant |

### Inversión Requerida

**Recursos:**
- 1 FTE de SRE senior como Documentation Champion (6 meses)
- 0.5 FTE de cada equipo (distributed)
- Budget para tooling: Azure DevOps ya disponible ✅

**ROI Esperado:**
- Costo: ~$150K (salarios + tiempo)
- Ahorro: ~$400K/año (menos incidentes, onboarding más rápido)
- **Payback period: 4-5 meses**

### Primeros 3 Pasos

1. **Semana 1:** Presentar este plan a leadership para buy-in
2. **Semana 2:** Identificar Documentation Champion y piloto de 2 servicios
3. **Semana 3-4:** Ejecutar piloto, iterar, y preparar para scale-up

---

**¡Empecemos! 🚀**

Para preguntas o soporte:
- 📧 Email: [sre-team@company.com]
- 💬 Slack: #documentation-project
- 📅 Office Hours: Martes 10-11 AM UTC

---

*Documento creado: 2026-02-15*  
*Última actualización: 2026-02-15*  
*Versión: 1.0*  
*Owner: SRE Team*
