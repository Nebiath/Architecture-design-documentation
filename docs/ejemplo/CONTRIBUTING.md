# CONTRIBUTING.md
# Guía de Contribución - Architecture Documentation

> **Versión:** 1.0  
> **Última revisión:** 2024-03-15  
> **Mantenido por:** Architecture Team — [`#documentation`](https://teams.microsoft.com) en Teams

Bienvenido/a al repositorio de documentación de arquitectura IT. Esta guía explica **cómo contribuir correctamente** según tu rol y el tipo de cambio que quieres hacer. Léela completa antes de tu primera contribución. Te llevará unos 10 minutos y te ahorrará muchos más.

---

## 📋 Índice

1. [Principios Fundamentales](#1-principios-fundamentales)
2. [Quién Puede Contribuir](#2-quién-puede-contribuir)
3. [Tipos de Cambio y su Proceso](#3-tipos-de-cambio-y-su-proceso)
4. [Setup del Entorno Local](#4-setup-del-entorno-local)
5. [Workflow General Paso a Paso](#5-workflow-general-paso-a-paso)
6. [Workflows Específicos por Caso de Uso](#6-workflows-específicos-por-caso-de-uso)
   - [Documentar un Nuevo Servicio](#61-documentar-un-nuevo-servicio)
   - [Actualizar Documentación Existente](#62-actualizar-documentación-existente)
   - [Cambios de Infraestructura Departamental](#63-cambios-de-infraestructura-departamental)
   - [Proponer una Architecture Decision (ADR)](#64-proponer-una-architecture-decision-adr)
   - [Cambios a Templates](#65-cambios-a-templates)
   - [Cambios Cross-Departamentales](#66-cambios-cross-departamentales)
   - [Migrar un Documento desde Word](#67-migrar-un-documento-desde-word)
7. [Estructura del Repositorio](#7-estructura-del-repositorio)
8. [Ownership y Aprobaciones (CODEOWNERS)](#8-ownership-y-aprobaciones-codeowners)
9. [Estándares de Escritura](#9-estándares-de-escritura)
10. [Convenciones de Nomenclatura](#10-convenciones-de-nomenclatura)
11. [Pull Requests (Azure DevOps)](#11-pull-requests-azure-devops)
12. [Pipeline de Validación Automática](#12-pipeline-de-validación-automática)
13. [Definition of Done](#13-definition-of-done)
14. [Resolución de Conflictos](#14-resolución-de-conflictos)
15. [Preguntas Frecuentes](#15-preguntas-frecuentes)
16. [Contacto y Soporte](#16-contacto-y-soporte)

---

## 1. Principios Fundamentales

Antes de cualquier workflow, estos son los principios que guían toda contribución:

### "Si no está en el repo, no existe"
La documentación en este repositorio es la **única fuente de verdad**. Si el documento está en SharePoint, en tu disco local, o en tu cabeza, no cuenta como documentado.

### "El propietario escribe, el experto revisa"
Cada carpeta tiene un equipo responsable definido en CODEOWNERS. Ese equipo **escribe y mantiene** su documentación. Otros equipos pueden sugerir cambios, pero el propietario siempre aprueba.

### "Automatiza lo tedioso, humaniza lo importante"
El pipeline valida formato, links y ortografía por ti. Tú te centras en el contenido técnico correcto.

### "Docs como código"
Mismo proceso que para el código: branch → PR → review → merge. Nunca editar directamente en `main`.

### "Documentación viva, no documento muerto"
Una doc que no se actualiza en 90 días se considera obsoleta. Los service owners son responsables de mantenerla actualizada.

---

## 2. Quién Puede Contribuir

### Todo el mundo puede:
- ✅ Leer toda la documentación
- ✅ Abrir un Pull Request con cambios propuestos en **cualquier** área
- ✅ Comentar en Pull Requests de otros equipos
- ✅ Reportar errores o información desactualizada (issues en Azure DevOps)

### El equipo propietario puede:
- ✅ Todo lo anterior
- ✅ Aprobar Pull Requests en **su propia área** (CODEOWNERS)
- ✅ Mergear PRs una vez aprobados
- ✅ Crear y eliminar archivos en su carpeta

### Architecture Team puede:
- ✅ Todo lo anterior
- ✅ Aprobar cambios a `00-templates/` y `01-global/`
- ✅ Modificar CODEOWNERS
- ✅ Resolver conflictos de ownership
- ✅ Forzar merge en casos excepcionales documentados

### Technical Writer puede:
- ✅ Editar **cualquier** documento para mejoras de redacción/formato
- ✅ Requerir cambios de estilo sin requerir aprobación técnica
- ✅ Gestionar el proceso general de documentación

---

## 3. Tipos de Cambio y su Proceso

Elige el tipo de cambio correcto **antes** de empezar. Determinará qué proceso seguir.

| Tipo | Ejemplos | Aprobaciones necesarias | Tiempo estimado |
|------|----------|------------------------|-----------------|
| **Typo / Fix menor** | Corregir error ortográfico, arreglar link roto | 1 × propietario | < 1 hora |
| **Actualización de contenido** | Actualizar IPs, cambiar responsable, nueva versión | 1 × propietario | < 1 día |
| **Nuevo documento** | Nuevo servicio, nueva sección de infra | 1 × propietario + pipeline verde | 1-3 días |
| **Cambio estructural** | Mover carpetas, renombrar secciones | Architecture Team | 1 semana |
| **Cambio a template** | Modificar plantilla existente o añadir nueva | Architecture Team + Technical Writer | 1-2 semanas |
| **ADR (Architecture Decision)** | Proponer decisión arquitectónica | Architecture Team + Principal Engineers | 1-2 semanas |
| **Cross-departamental** | Afecta a 2+ departamentos | Todos los propietarios afectados | 1 semana |

> **Regla de oro:** Cuando tengas dudas sobre el tipo, empieza un PR y menciona a `@architecture-team`. Ellos te orientarán.

---

## 4. Setup del Entorno Local

Configura tu entorno **una sola vez** antes de tu primera contribución.

### 4.1 Requisitos

```bash
# Verificar que tienes Git instalado
git --version   # Necesitas >= 2.30

# Verificar Node.js (para linters)
node --version  # Necesitas >= 18.x

# Verificar Python (para scripts de utilidad)
python3 --version  # Necesitas >= 3.9
```

Si te falta algo:
- **Git:** https://git-scm.com/downloads
- **Node.js:** https://nodejs.org (LTS version)
- **Python:** https://python3.org

### 4.2 Clonar el Repositorio

```bash
# Clonar vía Azure DevOps (HTTPS)
git clone https://dev.azure.com/{org}/Architecture%20Documentation/_git/architecture-docs
cd architecture-docs

# O vía SSH (si tienes clave SSH configurada en Azure DevOps)
git clone git@ssh.dev.azure.com:v3/{org}/Architecture%20Documentation/architecture-docs
cd architecture-docs
```

### 4.3 Instalar Linters Locales

```bash
# Instalar linters globalmente (una sola vez)
npm install -g markdownlint-cli markdown-link-check cspell

# Verificar instalaciones
markdownlint --version    # >= 0.33
markdown-link-check --version
cspell --version
```

### 4.4 Configurar Git

```bash
# Configurar tu identidad (usa el mismo email que en Azure DevOps)
git config user.name "Tu Nombre"
git config user.email "tu.email@empresa.com"

# Configurar el editor por defecto (opcional)
git config core.editor "code --wait"   # VS Code
# git config core.editor "vim"         # vim

# Verificar configuración
git config --list
```

### 4.5 Configurar Pre-commit Hook (Recomendado)

El hook valida tu Markdown antes de cada commit, evitando que el pipeline falle.

```bash
# Crear hook de pre-commit
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
# Pre-commit hook: valida archivos Markdown modificados

echo "🔍 Validando archivos Markdown..."

# Obtener archivos .md en staging
STAGED_MD=$(git diff --cached --name-only --diff-filter=ACM | grep "\.md$")

if [ -z "$STAGED_MD" ]; then
  echo "✅ No hay archivos Markdown que validar"
  exit 0
fi

# Ejecutar markdownlint solo en los archivos staged
echo "$STAGED_MD" | xargs markdownlint -c .markdownlint.json
LINT_EXIT=$?

if [ $LINT_EXIT -ne 0 ]; then
  echo ""
  echo "❌ Hay errores de linting en tus archivos Markdown."
  echo "   Ejecuta: markdownlint --fix <archivo> para corregirlos automáticamente"
  echo "   O revisa los errores arriba y corrígelos manualmente."
  exit 1
fi

echo "✅ Validación local superada"
exit 0
EOF

# Dar permisos de ejecución
chmod +x .git/hooks/pre-commit
```

### 4.6 Verificar Setup

```bash
# Test completo de entorno
echo "# Test" > /tmp/test-setup.md
markdownlint /tmp/test-setup.md && echo "✅ markdownlint OK"
cspell /tmp/test-setup.md && echo "✅ cspell OK"
rm /tmp/test-setup.md

echo ""
echo "✅ Entorno configurado correctamente"
```

---

## 5. Workflow General Paso a Paso

Este es el proceso base que aplica a **cualquier tipo de contribución**. Los workflows específicos de la Sección 6 añaden pasos adicionales según el caso.

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO DE CONTRIBUCIÓN                    │
│                                                             │
│  1. Sync        2. Branch      3. Editar     4. Validar    │
│  main ──────▶  feature/ ──▶  Escribir ──▶  Local lint    │
│                                                             │
│  5. Commit      6. Push        7. PR          8. Review    │
│  git add ──▶   origin ──────▶ Crear PR ──▶  CODEOWNERS   │
│                                                             │
│  9. Pipeline   10. Approve    11. Merge      12. Done      │
│  CI checks ─▶  Aprobado ───▶  Squash ─────▶  🎉          │
└─────────────────────────────────────────────────────────────┘
```

### Paso 1: Sincronizar con main

```bash
# Siempre empieza desde main actualizado
git checkout main
git pull origin main
```

### Paso 2: Crear tu Branch

```bash
# Formato: <tipo>/<departamento-o-scope>/<descripcion-breve>
git checkout -b docs/devops/update-kubernetes-architecture

# Más ejemplos de nombres válidos:
# docs/lan/add-vlan-design
# fix/windows/broken-link-ad-doc
# docs/services/new-payment-gateway
# adr/cloud-azure/use-azure-monitor
# update/global/security-standards-2024
```

**Tipos de branch:**

| Prefijo | Cuándo usar |
|---------|-------------|
| `docs/` | Nuevo documento o sección completa |
| `update/` | Actualización de contenido existente |
| `fix/` | Corrección de typos, links rotos, errores menores |
| `adr/` | Architecture Decision Record nuevo |
| `refactor/` | Reorganización sin cambio de contenido |
| `template/` | Cambios a templates |

### Paso 3: Escribir y Editar

Trabaja en tu documento. Consulta la [Sección 9 (Estándares de Escritura)](#9-estándares-de-escritura) si tienes dudas de formato.

```bash
# Abrir archivo existente
code 02-infrastructure/devops/architecture/kubernetes-architecture.md

# O crear nuevo con el script de utilidad
./09-tools-scripts/generators/new-service.sh mi-servicio mi-equipo
```

### Paso 4: Validar Localmente

```bash
# Validar solo tus archivos modificados
git diff --name-only | grep ".md$" | xargs markdownlint -c .markdownlint.json

# Validar un directorio completo
markdownlint 02-infrastructure/devops/ -c .markdownlint.json

# Auto-fix errores simples (espacios, líneas vacías, etc.)
markdownlint --fix 02-infrastructure/devops/architecture/kubernetes-architecture.md

# Comprobar links (puede tardar si hay muchos)
markdown-link-check 02-infrastructure/devops/architecture/kubernetes-architecture.md

# Comprobar ortografía
cspell 02-infrastructure/devops/architecture/kubernetes-architecture.md
```

### Paso 5: Commit

```bash
# Ver qué has cambiado
git status
git diff

# Añadir tus cambios
git add 02-infrastructure/devops/architecture/kubernetes-architecture.md

# Commit con mensaje descriptivo (ver Sección 10.3)
git commit -m "docs(devops): update Kubernetes architecture for v1.28 upgrade"
```

**Formato de commit message:**

```
<tipo>(<scope>): <descripción breve en presente>

[cuerpo opcional: qué cambió y por qué]

[referencia opcional: Closes AB#1234]
```

| Tipo | Cuándo |
|------|--------|
| `docs` | Nuevo contenido o contenido actualizado |
| `fix` | Corrección de errores en docs existentes |
| `refactor` | Reorganización sin cambio de contenido |
| `chore` | Cambios de configuración, scripts, CI |

### Paso 6: Push

```bash
# Primera vez (crea la rama en remoto)
git push -u origin docs/devops/update-kubernetes-architecture

# Pushes siguientes en la misma rama
git push
```

### Paso 7: Crear Pull Request

1. **Ir a:** Azure DevOps → Repos → Pull Requests → **New Pull Request**
2. **Source branch:** `docs/devops/update-kubernetes-architecture`
3. **Target branch:** `main`
4. **Título:** Descriptivo y claro (ver [Sección 11](#11-pull-requests-azure-devops))
5. **Descripción:** Completar el template con qué cambió y por qué
6. **Reviewers:** Se asignan automáticamente vía CODEOWNERS
7. Click **Create**

### Paso 8: Responder al Review

El pipeline ejecuta automáticamente. Mientras, los reviewers asignados revisarán el contenido.

```bash
# Si el reviewer pide cambios, hazlos en la misma rama
git checkout docs/devops/update-kubernetes-architecture
# ... editar ...
git add .
git commit -m "fix(devops): address review comments on K8s arch doc"
git push
# El PR se actualiza automáticamente
```

### Paso 9: Merge

Una vez que:
- ✅ Pipeline de CI pasa (linting, links, estructura)
- ✅ Al menos 1 CODEOWNER ha aprobado
- ✅ Todos los comentarios están resueltos

El PR puede ser mergeado. Por defecto usamos **Squash merge** para mantener el historial limpio.

---

## 6. Workflows Específicos por Caso de Uso

### 6.1 Documentar un Nuevo Servicio

**Cuándo:** Cuando hay que documentar un servicio que aún no existe en `03-services/`

**Tiempo estimado:** 4-8 horas de trabajo distribuidas en 1-3 días

```bash
# 1. Sincronizar main
git checkout main && git pull origin main

# 2. Crear branch
git checkout -b docs/services/new-<nombre-servicio>

# 3. Usar el script de scaffold (genera la estructura completa)
./09-tools-scripts/generators/new-service.sh <nombre-servicio> <equipo-owner>

# Ejemplo:
./09-tools-scripts/generators/new-service.sh payment-gateway payments-team

# Esto crea automáticamente:
# 03-services/payment-gateway/
# ├── README.md              ← Rellena con descripción del servicio
# ├── 01-architecture-design.md
# ├── 02-deployment-runbook.md
# ├── 03-service-ownership.md
# ├── 04-observability.md
# ├── 05-service-catalog.md
# ├── 06-incident-management.md
# ├── 07-change-management.md
# ├── 08-api-documentation.md
# └── diagrams/

# 4. Abrir y rellenar los templates
# OBLIGATORIOS (sin estos el PR no se aprueba):
code 03-services/payment-gateway/01-architecture-design.md
code 03-services/payment-gateway/02-deployment-runbook.md
code 03-services/payment-gateway/03-service-ownership.md
code 03-services/payment-gateway/04-observability.md

# OPCIONALES (según aplique al servicio):
code 03-services/payment-gateway/05-service-catalog.md
code 03-services/payment-gateway/06-incident-management.md
code 03-services/payment-gateway/07-change-management.md
code 03-services/payment-gateway/08-api-documentation.md

# 5. Añadir diagramas
# - Formato: PNG o SVG
# - Máximo recomendado: 1 MB por imagen
# - Nombrar: architecture.png, data-flow.png, network.png
cp mi-diagrama.png 03-services/payment-gateway/diagrams/architecture.png

# 6. Actualizar el CODEOWNERS (el script lo hace, verifica que está correcto)
cat .azuredevops/CODEOWNERS | grep payment-gateway
# Debe mostrar: /03-services/payment-gateway/ @payments-team

# 7. Validar todo el directorio del servicio
markdownlint 03-services/payment-gateway/ -c .markdownlint.json

# 8. Commit y push
git add 03-services/payment-gateway/ .azuredevops/CODEOWNERS
git commit -m "docs(services): add payment-gateway initial documentation"
git push -u origin docs/services/new-payment-gateway

# 9. Crear PR con template de nuevo servicio (ver Sección 11.2)
```

**Checklist específico para Nuevo Servicio:**

```
Contenido mínimo obligatorio:
□ Architecture Design con SLOs definidos
□ Service Owner identificado (persona, no equipo)
□ On-call rotation o escalation path
□ Al menos 1 runbook operacional
□ Dashboard de monitoring linkado (o descripción de qué monitorizar)

Calidad:
□ No hay secciones con "[TBD]" en documentos obligatorios
□ Todos los links funcionan
□ Hay al menos 1 diagrama
□ El README del servicio tiene Quick Links a todos los docs
□ CODEOWNERS actualizado con el nuevo servicio
```

---

### 6.2 Actualizar Documentación Existente

**Cuándo:** El servicio ya existe pero algo ha cambiado (nueva versión, nuevo responsable, nuevos SLOs, etc.)

```bash
# 1. Sincronizar
git checkout main && git pull origin main

# 2. Branch descriptivo de qué estás actualizando
git checkout -b update/devops/k8s-cluster-prod-eu-v1.28

# 3. Editar el documento
code 02-infrastructure/devops/clusters/prod-cluster-eu.md

# 4. Añadir entrada en el changelog del documento
# (Todos los docs tienen una sección ## Changelog al final)
# Formato: | YYYY-MM-DD | Tu Nombre | Descripción breve |

# 5. Actualizar la fecha en el frontmatter/header
# > **Última actualización:** 2024-03-15

# 6. Validar y commit
markdownlint update/devops/clusters/prod-cluster-eu.md -c .markdownlint.json
git add 02-infrastructure/devops/clusters/prod-cluster-eu.md
git commit -m "update(devops): upgrade prod-cluster-eu docs to K8s v1.28"
git push -u origin update/devops/k8s-cluster-prod-eu-v1.28

# 7. PR con tipo "Actualización de contenido"
```

**Triggers de actualización (cuándo actualizar OBLIGATORIAMENTE):**

| Evento | Documentos a actualizar |
|--------|------------------------|
| Cambio de versión mayor | Architecture Design, Deployment Runbook |
| Cambio de responsable/on-call | Service Ownership & RACI |
| Cambio de umbrales de alerta | Observability & Monitoring |
| Nuevo endpoint de API | API Documentation |
| Incidente post-mortem | Incident Management, Runbooks |
| Cambio de infraestructura | Architecture Design, Deployment |
| Cambio de proveedor/herramienta | Todos los afectados |

---

### 6.3 Cambios de Infraestructura Departamental

**Cuándo:** Cambios en la documentación de uno de los 8 departamentos de infraestructura

**Departamentos y sus rutas:**

| Departamento | Ruta | CODEOWNERS |
|---|---|---|
| Hardware | `02-infrastructure/hardware/` | `@infrastructure-team @hardware-lead` |
| Virtualización | `02-infrastructure/virtualization/` | `@virtualization-team @vmware-admin` |
| WAN | `02-infrastructure/wan/` | `@network-team @wan-lead` |
| LAN | `02-infrastructure/lan/` | `@network-team @lan-lead` |
| Windows | `02-infrastructure/windows/` | `@windows-team @ad-admin` |
| Linux | `02-infrastructure/linux/` | `@linux-team @linux-lead` |
| DevOps / K8s | `02-infrastructure/devops/` | `@devops-team @k8s-admin` |
| Cloud Azure | `02-infrastructure/cloud-azure/` | `@cloud-team @azure-architect` |

```bash
# Ejemplo: Añadir documentación de nueva VLAN al departamento LAN

# 1. Branch
git checkout main && git pull origin main
git checkout -b docs/lan/add-vlan-200-dmz

# 2. Crear nuevo documento (o editar existente)
# Si es nuevo, ¿qué carpeta corresponde?
# 02-infrastructure/lan/
# ├── architecture/     ← Diseños y decisiones
# ├── segmentation/     ← VLANs, subnets, access control
# ├── services/         ← DHCP, DNS, NTP
# └── runbooks/         ← Procedimientos operacionales

# Este caso va en segmentation/
code 02-infrastructure/lan/segmentation/vlan-200-dmz.md

# 3. Seguir el template de infraestructura:
# - Descripción y propósito
# - Diagrama de red (obligatorio para VLANs)
# - Tabla de subnets y rangos
# - Equipos afectados / ACLs
# - Procedimiento de cambio

# 4. Asegurar que el README del departamento menciona el nuevo doc
code 02-infrastructure/lan/README.md

# 5. Validar y commit
markdownlint 02-infrastructure/lan/ -c .markdownlint.json
git add 02-infrastructure/lan/
git commit -m "docs(lan): add VLAN 200 DMZ design documentation"
git push -u origin docs/lan/add-vlan-200-dmz

# 6. PR → revisado por @network-team @lan-lead
```

**Consideraciones por departamento:**

```
Hardware:
  - Incluir siempre: Número de serie, modelo, ubicación física (datacenter, rack, U)
  - Diagrama de rack si aplica

Virtualización:
  - Incluir siempre: Cluster, datastore, red de gestión
  - Compatibilidad con hardware subyacente

WAN:
  - Incluir siempre: CID del circuito, proveedor, SLA del proveedor
  - Diagrama de conectividad extremo a extremo

LAN:
  - Incluir siempre: VLAN ID, subnet, gateway, DHCP scope
  - Tabla de puertos de switch si aplica

Windows:
  - Incluir siempre: OU en AD, GPO aplicadas, licencias
  - Impacto en Domain Controllers

Linux:
  - Incluir siempre: Distribución y versión exacta, método de gestión (Ansible/Puppet)
  - Configuración de sudo y acceso

DevOps / Kubernetes:
  - Incluir siempre: Versión de K8s, nodos (conteo y tipo), namespace
  - Estrategia de actualización (blue/green, rolling, etc.)

Cloud Azure:
  - Incluir siempre: Subscription ID, Resource Group, región(es)
  - Tagging strategy aplicada
  - Implicaciones de coste (estimación mensual)
```

---

### 6.4 Proponer una Architecture Decision (ADR)

**Cuándo:** Se va a tomar una decisión técnica importante que afecta a la arquitectura general o a un servicio crítico.

**Regla:** Cualquier cambio que sea difícil o costoso de revertir **debe** tener un ADR.

```bash
# 1. Branch
git checkout main && git pull origin main
git checkout -b adr/cloud-azure/use-azure-monitor-for-observability

# 2. Obtener el siguiente número de ADR
ls 06-decisions/ | grep "^[0-9]" | sort | tail -1
# Output: 005-microservices-decomposition.md
# → Siguiente: 006

# 3. Crear el ADR desde el template
cp 06-decisions/template.md 06-decisions/006-use-azure-monitor-for-observability.md

# 4. Rellenar el ADR
code 06-decisions/006-use-azure-monitor-for-observability.md
```

**Estructura del ADR (rellenar este template):**

```markdown
# ADR-006: Usar Azure Monitor como plataforma de observabilidad

**Estado:** Propuesto | En revisión | Aceptado | Deprecado | Rechazado  
**Fecha:** 2024-03-15  
**Autores:** @tu-usuario  
**Revisores:** @architecture-team @sre-team  
**Área afectada:** cloud-azure, devops, todos los servicios Tier 1

---

## Contexto

[Describe el problema o situación que requiere esta decisión.
Qué está pasando, por qué hay que decidir ahora.]

## Decisión

[Describe claramente QUÉ se ha decidido hacer.
Sé específico: qué herramienta, qué versión, qué configuración.]

## Alternativas Consideradas

### Opción A: [Nombre]
- **Pros:** ...
- **Contras:** ...

### Opción B: [Nombre]
- **Pros:** ...
- **Contras:** ...

### Opción C (Elegida): [Nombre]
- **Pros:** ...
- **Contras:** ...

## Consecuencias

### Positivas
- [Qué mejora]

### Negativas / Trade-offs
- [Qué se sacrifica o qué riesgo se asume]

### Neutras
- [Qué cambia sin ser mejor ni peor]

## Plan de Implementación

[Pasos concretos para llevar a cabo la decisión]

## Criterios de Revisión

[Cuándo y cómo se revisará si esta decisión fue correcta]

## Referencias

- [Links a documentos relevantes, benchmarks, PoCs, etc.]
```

```bash
# 5. Commit
git add 06-decisions/006-use-azure-monitor-for-observability.md
git commit -m "adr(cloud-azure): propose Azure Monitor as observability platform"
git push -u origin adr/cloud-azure/use-azure-monitor-for-observability

# 6. PR con al menos 2 semanas de review period para ADRs

# 7. Una vez aprobado, cambiar estado a "Aceptado" y mergear
```

**Estados de un ADR:**

```
Propuesto → En revisión → Aceptado → [Deprecado / Superseded]
                       → Rechazado
```

---

### 6.5 Cambios a Templates

**Cuándo:** Necesitas modificar o añadir un template en `00-templates/`

> ⚠️ **Atención:** Los cambios a templates afectan a **todos los equipos**. Son los cambios de mayor impacto y requieren el proceso más cuidadoso.

```bash
# 1. Branch
git checkout main && git pull origin main
git checkout -b template/add-database-design-template

# 2. Si es un template NUEVO
cp 00-templates/01-architecture-design.md 00-templates/09-database-design.md
code 00-templates/09-database-design.md

# 3. Si es una MODIFICACIÓN del template existente
# - No cambies la estructura general
# - Añadir secciones, no eliminarlas
# - Si eliminas algo, justificarlo en el PR
code 00-templates/02-deployment-runbook.md

# 4. IMPORTANTE: Actualizar el README de templates
code 00-templates/README.md

# 5. Si el cambio al template requiere que todos los servicios
# existentes lo adopten, crear un issue de seguimiento:
# Azure DevOps → Boards → New Issue
# Título: "Adopt new template section: [nombre sección] in all Tier 1 services"
# Asignar a: Technical Writer

# 6. Commit y PR dirigido a @architecture-team @technical-writer
git add 00-templates/
git commit -m "template: add database design template for DB-intensive services"
git push -u origin template/add-database-design-template
```

**Reglas para cambios de templates:**

- ✅ Se pueden añadir secciones opcionales con `> Opcional: ...`
- ✅ Se pueden mejorar las instrucciones/ejemplos dentro de secciones
- ✅ Se puede añadir un template nuevo numerado
- ⚠️ Cambiar secciones obligatorias requiere plan de migración
- ❌ No se pueden eliminar secciones sin votación del Architecture Team
- ❌ No cambiar la numeración de templates existentes

---

### 6.6 Cambios Cross-Departamentales

**Cuándo:** El cambio afecta a la documentación de dos o más departamentos, o a `04-cross-cutting/`

```bash
# Ejemplo: Nuevo estándar de monitorización que afecta a DevOps y Cloud Azure

# 1. Branch
git checkout main && git pull origin main
git checkout -b update/cross/unified-monitoring-standards-2024

# 2. Editar todos los documentos afectados
code 04-cross-cutting/monitoring/monitoring-strategy.md
code 02-infrastructure/devops/observability/prometheus-setup.md
code 02-infrastructure/cloud-azure/services/aks-architecture.md

# 3. Para cambios en 04-cross-cutting/
# Los CODEOWNERS son: @sre-team @devops-team @architecture-team
# Necesitarás aprobación de todos los propietarios

# 4. Para cambios en múltiples departamentos
# Menciona explícitamente en el PR a todos los owners afectados:
# "Este PR afecta a DevOps (@devops-team) y Cloud Azure (@cloud-team).
# Ambos equipos deben revisar y aprobar."

# 5. Commit con scope "cross" para indicar que afecta a múltiples áreas
git add .
git commit -m "update(cross): align monitoring standards across DevOps and Azure"
git push -u origin update/cross/unified-monitoring-standards-2024

# 6. PR con todos los owners como reviewers obligatorios
```

**Tips para PRs cross-departamentales:**

- Organiza una breve sesión de alineamiento **antes** de abrir el PR (15-30 min)
- Menciona el motivo del cambio cross-departamental claramente
- Da más tiempo de review (5 días hábiles en lugar de los 2 habituales)
- Considera un PR por departamento si los cambios son independientes

---

### 6.7 Migrar un Documento desde Word

**Cuándo:** Hay documentación existente en Word (.docx) que hay que migrar al repo.

```bash
# Pre-requisito: tener Pandoc instalado
# pandoc --version   # debe mostrar versión >= 2.x
# Si no: ver documento 17-word-to-markdown-conversion.md

# 1. Branch
git checkout main && git pull origin main
git checkout -b docs/windows/migrate-ad-architecture-from-word

# 2. Convertir con Pandoc
mkdir -p 02-infrastructure/windows/architecture/diagrams

pandoc "Arquitectura AD.docx" \
  -f docx -t markdown \
  -o 02-infrastructure/windows/architecture/active-directory.md \
  --wrap=none \
  --extract-media=02-infrastructure/windows/architecture/diagrams \
  --markdown-headings=atx \
  --standalone

# 3. Limpiar el Markdown generado
# Pandoc hace un buen trabajo, pero siempre revisar:

# a) Aplicar auto-fix de markdownlint
markdownlint --fix 02-infrastructure/windows/architecture/active-directory.md

# b) Revisar manualmente (spot-check):
#    - Tablas (especialmente si tenían merged cells)
#    - Código en bloques correctos (```)
#    - Links internos apuntando a rutas correctas
#    - Imágenes extraídas y referenciadas correctamente
#    - Secciones que quedaron con "[TBD]" del Word original

# c) Añadir frontmatter/header estándar si Pandoc no lo generó:
# Al inicio del archivo añadir:
# > **Migrado desde:** Arquitectura AD.docx  
# > **Fecha de migración:** 2024-03-15  
# > **Última actualización original:** [fecha del Word]

# 4. Verificar imágenes
ls 02-infrastructure/windows/architecture/diagrams/
# Renombrar si Pandoc dio nombres genéricos como "image1.png"
mv 02-infrastructure/windows/architecture/diagrams/image1.png \
   02-infrastructure/windows/architecture/diagrams/ad-topology.png

# Actualizar la referencia en el .md:
sed -i 's/diagrams\/image1.png/diagrams\/ad-topology.png/g' \
  02-infrastructure/windows/architecture/active-directory.md

# 5. Commit
git add 02-infrastructure/windows/architecture/
git commit -m "docs(windows): migrate Active Directory architecture from Word"
git push -u origin docs/windows/migrate-ad-architecture-from-word

# 6. En el PR, mencionar:
# - Nombre del archivo Word original
# - Fecha del documento Word
# - Si hay secciones que no se pudieron migrar bien
```

> **Nota:** El archivo `.docx` original NO se sube al repositorio. Se guarda en SharePoint o se archiva, pero no en Git. Git es solo para Markdown.

---

## 7. Estructura del Repositorio

Referencia rápida. Para la guía completa ver `09-repository-structure-guide.md`.

```
architecture-docs/
├── .azuredevops/
│   ├── CODEOWNERS              ← Propietarios por path
│   ├── pull_request_template.md
│   └── pipelines/
│       └── azure-pipelines.yml ← CI/CD de validación
├── 00-templates/               ← Templates (no editar sin proceso)
├── 01-global/                  ← Políticas company-wide
├── 02-infrastructure/          ← Por departamento
│   ├── hardware/
│   ├── virtualization/
│   ├── wan/
│   ├── lan/
│   ├── windows/
│   ├── linux/
│   ├── devops/
│   └── cloud-azure/
├── 03-services/                ← Por servicio/aplicación
├── 04-cross-cutting/           ← Concerns transversales
├── 05-projects/                ← Proyectos y migraciones en curso
├── 06-decisions/               ← Architecture Decision Records
├── 07-integration/             ← Docs de integración entre dominios
├── 08-decommissioned/          ← Sistemas retirados (no borrar, archivar)
└── 09-tools-scripts/           ← Herramientas y automatización
```

**Regla de ubicación:** Si no sabes dónde va un documento, pregúntate:

1. ¿Es un diseño de un servicio concreto? → `03-services/<nombre-servicio>/`
2. ¿Es sobre la infraestructura de un departamento? → `02-infrastructure/<departamento>/`
3. ¿Afecta a todos o a varios equipos? → `04-cross-cutting/<tema>/`
4. ¿Es una decisión arquitectónica? → `06-decisions/`
5. ¿Es un proyecto temporal? → `05-projects/`
6. Si aún tienes dudas → Pregunta en `#documentation` antes de crear el archivo

---

## 8. Ownership y Aprobaciones (CODEOWNERS)

El fichero `.azuredevops/CODEOWNERS` define **quién debe aprobar cada tipo de cambio**.

### Resumen de Ownership

| Área | Propietario Principal | Co-propietario |
|------|----------------------|----------------|
| `00-templates/` | `@architecture-team` | `@technical-writer` |
| `01-global/` | `@architecture-team` | `@security-team` |
| `02-infrastructure/hardware/` | `@infrastructure-team` | `@hardware-lead` |
| `02-infrastructure/virtualization/` | `@virtualization-team` | `@vmware-admin` |
| `02-infrastructure/wan/` | `@network-team` | `@wan-lead` |
| `02-infrastructure/lan/` | `@network-team` | `@lan-lead` |
| `02-infrastructure/windows/` | `@windows-team` | `@ad-admin` |
| `02-infrastructure/linux/` | `@linux-team` | `@linux-lead` |
| `02-infrastructure/devops/` | `@devops-team` | `@k8s-admin` |
| `02-infrastructure/cloud-azure/` | `@cloud-team` | `@azure-architect` |
| `03-services/*/` | `@<equipo-del-servicio>` | `@<service-owner>` |
| `04-cross-cutting/` | `@sre-team` | `@architecture-team` |
| `06-decisions/` | `@architecture-team` | `@principal-engineers` |

### Proceso cuando eres el Reviewer

Cuando Azure DevOps te asigna como reviewer:

1. **Tienes 2 días hábiles** para revisar (5 días para ADRs y cambios de template)
2. **Qué revisar:**
   - ¿El contenido es técnicamente correcto?
   - ¿Están todas las secciones obligatorias?
   - ¿Los links funcionan? (el pipeline lo verifica, pero échale un ojo)
   - ¿La información es suficientemente clara para alguien externo al equipo?
3. **Opciones:**
   - ✅ **Approve:** Listo para mergear
   - 💬 **Comment:** Sugerencia no bloqueante
   - 🔄 **Request Changes:** Hay algo que corregir antes de aprobar

### Cómo Pedir Review Urgente

Si necesitas aprobación urgente (p.ej. doc relacionada con incidente activo):

```
1. Mergear el contenido mínimo con el Service Owner urgentemente
2. Crear un follow-up PR para completar el resto
3. Mencionar en Teams: @reviewer necesito aprobación urgente en PR #123
   porque [razón]. Gracias.
```

---

## 9. Estándares de Escritura

### 9.1 Idioma

- **Español:** Para todos los documentos de infraestructura, servicios y procesos internos
- **Inglés:** Para ADRs técnicos donde los términos en inglés son estándar de industria, y para comentarios en código/scripts
- **Términos técnicos:** Usar el término en inglés (kubernetes, cluster, pipeline...) sin traducir

### 9.2 Formato General

```markdown
# Solo un H1 por documento (el título del documento)

## Secciones principales con H2

### Subsecciones con H3

#### Sub-subsecciones con H4 (usar con moderación)
```

- **Párrafos:** Escritura en prosa, no listas para todo
- **Listas:** Solo cuando son realmente enumeraciones, no para sustituir párrafos
- **Código:** Siempre en bloques con el lenguaje especificado

```bash
# Código bash: siempre especificar el lenguaje
echo "ejemplo"
```

```yaml
# Código YAML
key: value
```

- **Negrita:** Para términos importantes o énfasis real. No decorativa.
- **Cursiva:** Para términos técnicos en su primera mención
- **Tablas:** Para comparaciones o datos estructurados. No para texto corrido.

### 9.3 Header Obligatorio en Todo Documento

Todo documento debe empezar con este bloque:

```markdown
# Título del Documento

> **Propietario:** @equipo-responsable  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado | Obsoleto  
> **Versión:** 1.0

Breve descripción de una o dos frases sobre qué contiene este documento.

---
```

### 9.4 Changelog Obligatorio al Final

```markdown
---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| 2024-03-15 | @tu-usuario | Creación inicial |
| 2024-04-01 | @otro-usuario | Actualizado SLOs según Q1 review |
```

### 9.5 Información Sensible

**Nunca incluir en el repositorio:**
- ❌ Contraseñas, tokens, API keys
- ❌ IPs privadas exactas de producción (usar rangos o nombres)
- ❌ Datos personales de clientes
- ❌ Información que no deba salir de la empresa

**Cómo referenciar información sensible:**

```markdown
# ✅ Correcto
La API key se obtiene de Azure Key Vault: `docs-api-key`
El servidor de base de datos es: `db-prod-01.internal` (IP en Vault)

# ❌ Incorrecto
API key: sk-a1b2c3d4e5f6...
IP servidor: 10.50.100.25
```

---

## 10. Convenciones de Nomenclatura

### 10.1 Archivos

```
# Formato: kebab-case, todo minúsculas, extensión .md
kubernetes-architecture.md       ✅
active-directory-design.md       ✅
KubernetesArchitecture.md        ❌ (no CamelCase)
active directory design.md       ❌ (no espacios)
k8s_arch.md                      ❌ (no underscores)
k8sArch.md                       ❌ (no camelCase)
```

**Prefijos numéricos** para ordenar documentos dentro de un servicio:

```
01-architecture-design.md
02-deployment-runbook.md
03-service-ownership.md
04-observability.md
```

### 10.2 Carpetas

```
# Formato: kebab-case, todo minúsculas
authentication-service/           ✅
cloud-azure/                      ✅
AuthenticationService/            ❌
cloud_azure/                      ❌
```

### 10.3 Branches

```
# Formato: <tipo>/<scope>/<descripcion-breve>
docs/devops/add-argocd-runbook           ✅
update/lan/vlan-100-gateway-ip           ✅
fix/windows/broken-link-in-ad-doc        ✅
adr/services/use-graphql-api             ✅

feature/doc                              ❌ (demasiado genérico)
my-branch                                ❌ (no descriptivo)
DOCS/Devops/Update                       ❌ (no mayúsculas)
```

### 10.4 Imágenes

```
# Nombrar según lo que muestra, no genérico
architecture-overview.png        ✅
network-topology-wan.svg         ✅
k8s-cluster-design.png           ✅

image1.png                       ❌
screenshot.png                   ❌
Diagrama Final v3.png            ❌
```

---

## 11. Pull Requests (Azure DevOps)

### 11.1 Título del PR

```
# Formato: <tipo>(<scope>): <descripción breve>

docs(devops): add Kubernetes 1.28 upgrade runbook
update(lan): update VLAN 100 gateway IP after network change
fix(windows): fix broken link in Active Directory architecture
adr(cloud-azure): propose Azure Monitor as observability platform
template(services): add optional compliance section to architecture template
```

### 11.2 Template de Descripción

El PR template está en `.azuredevops/pull_request_template.md`. Azure DevOps lo carga automáticamente. Siempre completarlo:

```markdown
## ¿Qué cambia este PR?

<!-- Descripción clara de qué documentación se añade/modifica/elimina -->

## ¿Por qué es necesario?

<!-- Contexto: ¿qué evento o decisión lo motiva? -->

## Tipo de cambio

- [ ] 🆕 Nuevo documento
- [ ] ✏️ Actualización de contenido existente
- [ ] 🔧 Corrección de errores / typos
- [ ] 🏗️ Cambio estructural / reorganización
- [ ] 📋 Cambio a template
- [ ] 🗳️ ADR (Architecture Decision Record)

## Checklist

- [ ] El documento tiene header con propietario y fecha
- [ ] El changelog del documento está actualizado
- [ ] Los links internos apuntan a rutas correctas
- [ ] Las imágenes están en la carpeta `diagrams/`
- [ ] No hay información sensible (passwords, IPs exactas de prod, tokens)
- [ ] El pipeline de CI pasa sin errores
- [ ] CODEOWNERS actualizado (si es documento nuevo)

## ¿Qué deben revisar los reviewers?

<!-- Orienta a los reviewers: ¿dónde centrar la atención? -->

## Documentos relacionados

<!-- Links a PRs, issues, incidentes, ADRs o docs relacionados -->
```

### 11.3 Buenas Prácticas de PR

**Tamaño:** Un PR debe ser fácil de revisar. Regla general: **< 400 líneas cambiadas**. Si es más grande, dividirlo.

**Un PR, un propósito:** No mezclar la migración de Word con la actualización de contenido. Son dos PRs separados.

**Draft PRs:** Si quieres feedback temprano antes de terminar, crea el PR como **Draft**. Cuando esté listo, marca como "Ready for Review".

**Self-review:** Antes de pedir review, léete el PR tú mismo en la vista de diffs de Azure DevOps. Detectarás cosas que en el editor no ves.

---

## 12. Pipeline de Validación Automática

El pipeline se ejecuta automáticamente en cada PR y en cada push a `main`. No puedes mergear si el pipeline falla (salvo en casos de emergencia aprobados por Architecture Team).

### Qué valida el pipeline

| Check | Herramienta | ¿Bloqueante? | Qué detecta |
|-------|-------------|-------------|-------------|
| Markdown lint | markdownlint | ✅ Sí | Formato incorrecto, headings, listas |
| Links rotos | markdown-link-check | ⚠️ Warning | Links internos y externos caídos |
| Ortografía | cspell | ⚠️ Warning | Errores ortográficos |
| Estructura | Script custom | ✅ Sí | Carpetas obligatorias, templates presentes |
| YAML | yamllint | ✅ Sí | Ficheros de config malformados |

### Si el pipeline falla

```bash
# 1. Ver el error en Azure DevOps → Pipelines → [tu build]

# 2. Reproducir localmente:
# Para markdown lint:
markdownlint tu-archivo.md -c .markdownlint.json

# Para links:
markdown-link-check tu-archivo.md -c .markdown-link-check.json

# Para spelling:
cspell tu-archivo.md

# 3. Auto-fix lo que se pueda:
markdownlint --fix tu-archivo.md

# 4. Lo que no se puede auto-fix, corrígelo manualmente

# 5. Commit el fix y push
git add .
git commit -m "fix: address pipeline lint errors"
git push
# El pipeline volverá a ejecutarse automáticamente
```

### Palabras técnicas en el diccionario

Si cspell marca como error una palabra técnica válida, añádela al diccionario:

```json
// cspell.json → words array
{
  "words": [
    "kubernetes",
    "terraform",
    "ansible",
    "tu-nueva-palabra"
  ]
}
```

Abrir PR con ese cambio hacia `@architecture-team`.

---

## 13. Definition of Done

Un documento se considera **"Done"** cuando cumple **todos** estos criterios:

### Para cualquier documento

- [ ] Tiene el header estándar (propietario, fecha, estado, versión)
- [ ] Tiene changelog actualizado
- [ ] No hay secciones con `[TBD]`, `[PENDIENTE]` o `[TODO]`
- [ ] El pipeline de CI pasa sin errores bloqueantes
- [ ] Al menos 1 CODEOWNER ha aprobado
- [ ] Todos los comentarios del PR están resueltos
- [ ] El PR está mergeado en `main`

### Para un servicio nuevo (adicional)

- [ ] Los 4 documentos obligatorios están completos (`01`, `02`, `03`, `04`)
- [ ] Service Owner identificado (persona concreta, no solo equipo)
- [ ] Hay al menos 1 diagrama de arquitectura
- [ ] CODEOWNERS actualizado con el nuevo servicio
- [ ] El README del servicio enlaza a todos los documentos

### Para un ADR (adicional)

- [ ] Estado cambiado de "Propuesto" a "Aceptado" o "Rechazado"
- [ ] Al menos 2 miembros de `@architecture-team` han aprobado
- [ ] Las alternativas consideradas están documentadas
- [ ] El plan de implementación es concreto y accionable

---

## 14. Resolución de Conflictos

### Conflicto de contenido técnico

Si un reviewer y el autor no se ponen de acuerdo sobre el contenido técnico:

1. Intentar resolverlo en los comentarios del PR (máximo 3 rondas)
2. Si no hay acuerdo, escalar al Tech Lead de ambos equipos (mención directa en el PR)
3. Si sigue sin resolverse, el Architecture Team decide en la siguiente Architecture Review

### Conflicto de ownership

Si no está claro qué equipo debe ser propietario de un documento:

1. Preguntar en `#documentation` en Teams con contexto claro
2. Si no hay acuerdo en 24h, decidirlo en el Architecture Review semanal
3. El Architecture Team tiene la última palabra en ownership

### Cambio urgente sin reviewer disponible

En casos de incidente activo o cambio de emergencia:

```bash
# Proceso de emergencia (documentar siempre el motivo):
# 1. Hacer el cambio en una branch normal
# 2. Mergear con aprobación del Architecture Team Lead (anula CODEOWNERS)
# 3. Crear issue de follow-up para revisión posterior por el equipo propietario
# 4. Registrar en el changelog: "Cambio de emergencia - YYYY-MM-DD - [motivo]"
```

---

## 15. Preguntas Frecuentes

**P: ¿Puedo editar directamente en `main` si es solo un typo?**

R: No. Siempre branch + PR, incluso para un typo. Es rápido y garantiza trazabilidad. Además, el pipeline se ejecuta en el PR, no solo en `main`.

---

**P: ¿Qué hago si encuentro un documento desactualizado que no es de mi equipo?**

R: Dos opciones según tu tiempo:
- **Opción rápida:** Crea un issue en Azure DevOps asignado al equipo propietario con "Documento desactualizado: [link]"
- **Opción directa:** Crea un PR con la corrección. El equipo propietario lo revisará y aprobará.

---

**P: ¿Puedo usar capturas de pantalla de herramientas en los documentos?**

R: Sí, pero con moderación. Las capturas se quedan obsoletas rápido. Preferir:
1. Descripción textual + link a la herramienta
2. Diagrama propio en Mermaid o Draw.io exportado a SVG/PNG
3. Captura solo si es imprescindible, y anotar la versión de la herramienta

---

**P: ¿En qué formato hago los diagramas?**

R: En orden de preferencia:
1. **Mermaid** (texto en el .md, se renderiza en Azure DevOps): para diagramas simples de flujo
2. **SVG** exportado de Draw.io / Visio: para diagramas complejos de red/arquitectura
3. **PNG** de alta resolución: si SVG no es posible

Los archivos de diagrama fuente (`.drawio`, `.vsdx`) van en la misma carpeta `diagrams/` para que otros puedan editarlos.

---

**P: ¿Con qué frecuencia debo revisar y actualizar mi documentación?**

R: Mínimo:
- **Tier 1 (crítico):** Review trimestral obligatorio
- **Tier 2:** Review semestral
- **Tier 3:** Review anual

Además, actualizar **siempre** que haya un cambio técnico real. El bot de Azure DevOps te recordará cuando un documento lleve más de 90 días sin actualización.

---

**P: ¿Qué pasa si mi PR lleva más de 5 días sin review?**

R: Escala así:
1. Menciona al reviewer en el PR con un recordatorio amable
2. Si en 24h no hay respuesta, menciona en `#documentation` en Teams
3. Si es urgente, contacta directamente al Tech Lead del equipo propietario

---

**P: ¿Tengo que documentar servicios legacy que nadie mantiene?**

R: Sí, con el nivel mínimo:
- `05-service-catalog.md` con contactos y estado "Legacy / Mantenimiento mínimo"
- `02-deployment-runbook.md` con el procedimiento de arranque/parada
- `03-service-ownership.md` con quién tiene que responder si falla

Un servicio legacy sin documentación es una bomba de tiempo.

---

**P: ¿Puedo usar IA para generar borradores de documentación?**

R: Sí, es una buena práctica para ahorrar tiempo. Pero:
- El contenido generado debe ser revisado y validado por un humano técnico
- No copies información técnica confidencial a herramientas de IA externas (IPs, credenciales, arquitecturas sensibles)
- El service owner es responsable de la corrección del contenido, independientemente de cómo se generó el borrador

---

## 16. Contacto y Soporte

### Canales por tipo de consulta

| Tipo de consulta | Canal |
|-----------------|-------|
| Dudas sobre cómo contribuir | `#documentation` en Teams |
| Reportar doc desactualizada | Issue en Azure DevOps |
| Problema con el pipeline de CI | `#devops` en Teams |
| Conflicto de ownership | `#documentation` + mención a `@architecture-team` |
| Cambio urgente / emergencia | `@architecture-team` en Teams (DM directo) |

### Office Hours de Documentación

**Cada martes de 15:00 a 16:00 UTC**
- Canal de Teams: Architecture Documentation — Office Hours
- Open floor: cualquier pregunta sobre el proceso
- Sesiones grabadas y disponibles en el canal

### Responsables

| Rol | Responsabilidad | Contacto |
|-----|----------------|---------|
| **Technical Writer** | Proceso, templates, calidad | `@technical-writer` |
| **Architecture Team Lead** | Decisiones de ownership, ADRs | `@architecture-lead` |
| **SRE Lead** | Pipeline CI/CD, scripts | `@sre-lead` |
| **Documentation Champions** | Primer punto de contacto por departamento | Ver tabla en `01-global/architecture-principles.md` |

---

> **¿Ves algo mejorable en esta guía?**  
> Abre un PR con el cambio en `CONTRIBUTING.md` dirigido a `@architecture-team`.  
> Esta guía también está viva. 🌱
