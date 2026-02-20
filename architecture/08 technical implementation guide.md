# Plan de Implementación Técnica - Infraestructura de Documentación
## Ubuntu Server 24.04 LTS + GitLab + Notion + Teams

> **Objetivo:** Levantar infraestructura completa para gestión de documentación  
> **Duración estimada:** 1-2 días  
> **Skill level requerido:** Sysadmin intermedio

---

## 📋 Índice

1. [Preparación del Servidor](#1-preparación-del-servidor)
2. [Instalación de GitLab CE](#2-instalación-de-gitlab-ce)
3. [Configuración de GitLab CI](#3-configuración-de-gitlab-ci)
4. [Linting Automatizado](#4-linting-automatizado)
5. [Cadena de Aprobación](#5-cadena-de-aprobación)
6. [Integración con Teams](#6-integración-con-teams)
7. [Notion Setup](#7-notion-setup)
8. [Backup y Monitoring](#8-backup-y-monitoring)
9. [Troubleshooting](#9-troubleshooting)

---

## 🎯 Arquitectura Overview

```
┌─────────────────────────────────────────────────────┐
│                  Ubuntu Server 24.04                │
│                                                     │
│  ┌──────────────┐    ┌────────────┐   ┌─────────┐ │
│  │   GitLab CE  │───▶│ GitLab CI  │──▶│ Runners │ │
│  │   (Git repo) │    │ (Pipeline) │   │         │ │
│  └──────┬───────┘    └─────┬──────┘   └─────────┘ │
│         │                  │                        │
│         │                  ▼                        │
│         │           ┌──────────────┐                │
│         │           │   Linters    │                │
│         │           │ - Markdown   │                │
│         │           │ - Links      │                │
│         │           │ - Spelling   │                │
│         │           └──────────────┘                │
│         │                                           │
│         ▼                                           │
│  ┌──────────────┐                                   │
│  │ CODEOWNERS   │ ← Approval workflow              │
│  │ Protection   │                                   │
│  └──────┬───────┘                                   │
└─────────┼───────────────────────────────────────────┘
          │
          ├──────────────▶ Microsoft Teams (Webhooks)
          │
          └──────────────▶ Notion (Manual sync o API)
```

---

## ⚙️ Requisitos Previos

### Hardware (VM)
- **CPU:** 4 cores (mínimo 2)
- **RAM:** 8 GB (mínimo 4 GB)
- **Disco:** 50 GB (SSD recomendado)
- **Red:** IP estática o DHCP reservation

### Software
- Ubuntu Server 24.04 LTS (instalación limpia)
- Acceso root o sudo
- Dominio/subdomain (ej: `gitlab.company.com`)
- Certificado SSL (Let's Encrypt recomendado)

### Accesos
- [ ] Cuenta Notion (workspace admin)
- [ ] Cuenta Microsoft 365 (Teams admin para webhooks)
- [ ] DNS configurado apuntando a tu VM
- [ ] Firewall rules permitiendo puertos 80/443/22

---

## 1. Preparación del Servidor

### 1.1 Actualizar Sistema

```bash
# SSH al servidor
ssh user@gitlab.company.com

# Actualizar paquetes
sudo apt update && sudo apt upgrade -y

# Instalar dependencias básicas
sudo apt install -y curl openssh-server ca-certificates \
  tzdata perl postfix git wget software-properties-common
```

**Nota:** Cuando instale `postfix`, seleccionar "Internet Site" y configurar hostname.

### 1.2 Configurar Hostname y Firewall

```bash
# Configurar hostname
sudo hostnamectl set-hostname gitlab.company.com

# Verificar
hostnamectl

# Configurar firewall (UFW)
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
sudo ufw status

# Expected output:
# Status: active
# To                         Action      From
# --                         ------      ----
# 22/tcp                     ALLOW       Anywhere
# 80/tcp                     ALLOW       Anywhere
# 443/tcp                    ALLOW       Anywhere
```

### 1.3 Configurar Swap (Importante para 8GB RAM)

```bash
# Crear swap de 4GB
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Hacer permanente
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verificar
sudo swapon --show
free -h
```

---

## 2. Instalación de GitLab CE

### 2.1 Añadir Repositorio de GitLab

```bash
# Añadir repositorio oficial de GitLab
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

# Verificar que se añadió
ls /etc/apt/sources.list.d/gitlab_gitlab-ce.list
```

### 2.2 Instalar GitLab CE

```bash
# Instalar GitLab Community Edition
# IMPORTANTE: Reemplazar EXTERNAL_URL con tu dominio
sudo EXTERNAL_URL="https://gitlab.company.com" apt install gitlab-ce

# Este proceso toma 5-10 minutos
# Al terminar verás:
# 
#        *.                  *.
#       ***                 ***
#      *****               *****
#     .******             *******
#     ********            ********
#    ,,,,,,,,,***********,,,,,,,,,
#   ,,,,,,,,,,,*********,,,,,,,,,,,
#   .,,,,,,,,,,,*******,,,,,,,,,,,,
#       ,,,,,,,,,*****,,,,,,,,,.
#          ,,,,,,,****,,,,,,
#             .,,,***,,,,
#                 ,*,.
#
#     _______ __  __          __
#    / ____(_) /_/ /   ____ _/ /_
#   / / __/ / __/ /   / __ `/ __ \
#  / /_/ / / /_/ /___/ /_/ / /_/ /
#  \____/_/\__/_____/\__,_/_.___/
```

### 2.3 Obtener Password Inicial

```bash
# GitLab genera un password aleatorio
sudo cat /etc/gitlab/initial_root_password

# Output example:
# Password: 5iveL!fe=wkE4BaJLhESRZ...
# 
# IMPORTANTE: Este archivo se borra en 24h
# Guarda el password en un gestor de contraseñas
```

### 2.4 Configurar SSL con Let's Encrypt

```bash
# Editar configuración
sudo nano /etc/gitlab/gitlab.rb

# Buscar y modificar estas líneas:
external_url 'https://gitlab.company.com'
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['admin@company.com']
letsencrypt['auto_renew'] = true

# Guardar (Ctrl+X, Y, Enter)

# Reconfigurar GitLab
sudo gitlab-ctl reconfigure

# Este proceso toma 2-3 minutos
# Let's Encrypt obtendrá certificado automáticamente
```

### 2.5 Verificar Instalación

```bash
# Verificar estado de servicios
sudo gitlab-ctl status

# Expected output (todos running):
# run: alertmanager: (pid 12345) 123s; run: log: (pid 12346) 123s
# run: gitaly: (pid 12347) 123s; run: log: (pid 12348) 123s
# run: gitlab-exporter: (pid 12349) 123s; run: log: (pid 12350) 123s
# run: gitlab-workhorse: (pid 12351) 123s; run: log: (pid 12352) 123s
# run: logrotate: (pid 12353) 123s; run: log: (pid 12354) 123s
# run: nginx: (pid 12355) 123s; run: log: (pid 12356) 123s
# run: postgres-exporter: (pid 12357) 123s; run: log: (pid 12358) 123s
# run: postgresql: (pid 12359) 123s; run: log: (pid 12360) 123s
# run: prometheus: (pid 12361) 123s; run: log: (pid 12362) 123s
# run: puma: (pid 12363) 123s; run: log: (pid 12364) 123s
# run: redis: (pid 12365) 123s; run: log: (pid 12366) 123s
# run: redis-exporter: (pid 12367) 123s; run: log: (pid 12368) 123s
# run: sidekiq: (pid 12369) 123s; run: log: (pid 12370) 123s

# Acceder vía navegador
# https://gitlab.company.com
# User: root
# Password: [del archivo initial_root_password]
```

### 2.6 Configuración Inicial de GitLab (Web UI)

1. **Login** con root y password
2. **Cambiar password:** Settings → Password
3. **Configurar email:**
   - Admin Area → Settings → Email
   - From address: gitlab@company.com
   - Display name: GitLab Documentation
4. **Deshabilitar sign-ups públicos:**
   - Admin Area → Settings → General
   - Sign-up restrictions → Desmarcar "Sign-up enabled"
5. **Configurar integración LDAP/AD (Opcional):**
   - Si tienes Active Directory, ver Anexo A

---

## 3. Configuración de GitLab CI

### 3.1 Crear Proyecto de Documentación

**Via Web UI:**

1. Login en GitLab
2. Click "New project"
3. "Create blank project"
   - Project name: `architecture-docs`
   - Project URL: `https://gitlab.company.com/documentation/architecture-docs`
   - Visibility: Internal
   - Initialize with README: ✅
4. Create project

### 3.2 Instalar GitLab Runner

```bash
# En el servidor GitLab (o VM separada)
# Añadir repositorio de GitLab Runner
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# Instalar
sudo apt install gitlab-runner

# Verificar
gitlab-runner --version
# Output: Version 16.x.x
```

### 3.3 Registrar Runner

**Obtener registration token:**
1. En proyecto → Settings → CI/CD → Runners
2. Copy registration token

**Registrar runner:**

```bash
sudo gitlab-runner register

# Responder preguntas:
# Enter the GitLab instance URL:
https://gitlab.company.com

# Enter the registration token:
[pegar token copiado]

# Enter a description for the runner:
docs-validation-runner

# Enter tags for the runner (comma-separated):
docs,validation,markdown

# Enter optional maintenance note:
[dejar vacío, Enter]

# Enter an executor:
shell

# Runner registered successfully
```

### 3.4 Configurar Runner

```bash
# Editar config del runner
sudo nano /etc/gitlab-runner/config.toml

# Modificar para añadir concurrent jobs:
concurrent = 4

[[runners]]
  name = "docs-validation-runner"
  url = "https://gitlab.company.com"
  token = "xxxxx"
  executor = "shell"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]

# Guardar y reiniciar
sudo gitlab-runner restart
```

### 3.5 Instalar Herramientas de Linting

```bash
# En el servidor donde corre el runner
# Instalar Node.js (para markdown linters)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verificar
node --version  # v20.x.x
npm --version   # 10.x.x

# Instalar linters globalmente
sudo npm install -g \
  markdownlint-cli \
  markdown-link-check \
  cspell

# Instalar Python y herramientas
sudo apt install -y python3 python3-pip
pip3 install proselint yamllint

# Verificar instalaciones
markdownlint --version
markdown-link-check --version
cspell --version
```

### 3.6 Crear Pipeline de CI/CD

**Crear archivo `.gitlab-ci.yml` en el proyecto:**

```bash
# En tu máquina local, clonar el repo
git clone https://gitlab.company.com/documentation/architecture-docs.git
cd architecture-docs

# Crear .gitlab-ci.yml
cat > .gitlab-ci.yml << 'EOF'
# GitLab CI Pipeline para Documentación
# Version: 1.0

# Definir stages
stages:
  - validate
  - lint
  - test
  - notify

# Variables globales
variables:
  GIT_DEPTH: 10
  FAIL_ON_WARNINGS: "false"

# Template para jobs de validación
.validation_template: &validation_template
  stage: validate
  tags:
    - docs
  only:
    - merge_requests
    - main

# Template para jobs de linting
.lint_template: &lint_template
  stage: lint
  tags:
    - docs
  only:
    - merge_requests
    - main
  allow_failure: false

# ==========================================
# STAGE: VALIDATE
# ==========================================

validate:structure:
  <<: *validation_template
  script:
    - echo "📁 Validating repository structure..."
    - |
      # Check required directories exist
      REQUIRED_DIRS="00-templates 01-global 02-infrastructure 03-services"
      for dir in $REQUIRED_DIRS; do
        if [ ! -d "$dir" ]; then
          echo "❌ Missing required directory: $dir"
          exit 1
        fi
      done
    - echo "✅ Structure validation passed"

validate:templates:
  <<: *validation_template
  script:
    - echo "📄 Validating template completeness..."
    - |
      # Check all required templates exist
      TEMPLATES=(
        "00-templates/01-architecture-design.md"
        "00-templates/02-deployment-runbook.md"
        "00-templates/03-service-ownership.md"
        "00-templates/04-observability.md"
      )
      for template in "${TEMPLATES[@]}"; do
        if [ ! -f "$template" ]; then
          echo "❌ Missing template: $template"
          exit 1
        fi
      done
    - echo "✅ Templates validation passed"

# ==========================================
# STAGE: LINT
# ==========================================

lint:markdown:
  <<: *lint_template
  script:
    - echo "📝 Linting markdown files..."
    - |
      # Find all markdown files (excluding node_modules, vendor, etc.)
      find . -type f -name "*.md" \
        -not -path "*/node_modules/*" \
        -not -path "*/vendor/*" \
        -not -path "*/.git/*" \
        > /tmp/md_files.txt
      
      # Lint markdown files
      if [ -s /tmp/md_files.txt ]; then
        cat /tmp/md_files.txt | xargs markdownlint \
          --config .markdownlint.json \
          --ignore node_modules \
          --ignore vendor
      else
        echo "No markdown files found"
      fi
    - echo "✅ Markdown linting passed"
  artifacts:
    when: on_failure
    paths:
      - markdownlint-errors.txt
    expire_in: 1 week

lint:links:
  <<: *lint_template
  script:
    - echo "🔗 Checking for broken links..."
    - |
      # Find all markdown files
      find . -type f -name "*.md" \
        -not -path "*/node_modules/*" \
        -not -path "*/.git/*" \
        > /tmp/md_files.txt
      
      # Check links in each file
      EXIT_CODE=0
      while IFS= read -r file; do
        echo "Checking links in: $file"
        markdown-link-check "$file" -c .markdown-link-check.json || EXIT_CODE=1
      done < /tmp/md_files.txt
      
      exit $EXIT_CODE
    - echo "✅ Link checking passed"
  allow_failure: true  # Allows false positives
  artifacts:
    when: on_failure
    paths:
      - link-check-errors.txt
    expire_in: 1 week

lint:spelling:
  <<: *lint_template
  script:
    - echo "📖 Checking spelling..."
    - |
      # Check spelling in markdown files
      find . -type f -name "*.md" \
        -not -path "*/node_modules/*" \
        -not -path "*/.git/*" \
        -exec cspell {} \; || true
    - echo "✅ Spelling check completed"
  allow_failure: true  # Spelling can have false positives

lint:yaml:
  <<: *lint_template
  script:
    - echo "📋 Linting YAML files..."
    - |
      find . -type f \( -name "*.yml" -o -name "*.yaml" \) \
        -not -path "*/.git/*" \
        -not -path "*/node_modules/*" \
        -exec yamllint -c .yamllint {} \;
    - echo "✅ YAML linting passed"

# ==========================================
# STAGE: TEST
# ==========================================

test:completeness:
  stage: test
  tags:
    - docs
  script:
    - echo "🔍 Testing documentation completeness..."
    - |
      # Check that service docs have all required files
      for service_dir in 03-services/*/; do
        if [ -d "$service_dir" ]; then
          service=$(basename "$service_dir")
          echo "Checking service: $service"
          
          # Required files
          FILES=(
            "01-architecture-design.md"
            "02-deployment-runbook.md"
            "03-service-ownership.md"
            "04-observability.md"
          )
          
          for file in "${FILES[@]}"; do
            if [ ! -f "$service_dir/$file" ]; then
              echo "⚠️  Missing $file in $service"
            fi
          done
        fi
      done
    - echo "✅ Completeness test passed"
  allow_failure: true
  only:
    - main

# ==========================================
# STAGE: NOTIFY
# ==========================================

notify:teams:success:
  stage: notify
  tags:
    - docs
  script:
    - echo "📢 Sending success notification to Teams..."
    - |
      # Teams webhook (set as CI/CD variable)
      if [ -n "$TEAMS_WEBHOOK_URL" ]; then
        curl -H "Content-Type: application/json" \
          -d "{
            \"@type\": \"MessageCard\",
            \"@context\": \"https://schema.org/extensions\",
            \"summary\": \"Documentation Pipeline Passed\",
            \"themeColor\": \"28a745\",
            \"title\": \"✅ Documentation Pipeline Passed\",
            \"sections\": [{
              \"facts\": [
                {\"name\": \"Project\", \"value\": \"$CI_PROJECT_NAME\"},
                {\"name\": \"Branch\", \"value\": \"$CI_COMMIT_REF_NAME\"},
                {\"name\": \"Commit\", \"value\": \"$CI_COMMIT_SHORT_SHA\"},
                {\"name\": \"Author\", \"value\": \"$GITLAB_USER_NAME\"}
              ]
            }],
            \"potentialAction\": [{
              \"@type\": \"OpenUri\",
              \"name\": \"View Pipeline\",
              \"targets\": [{
                \"os\": \"default\",
                \"uri\": \"$CI_PIPELINE_URL\"
              }]
            }]
          }" \
          "$TEAMS_WEBHOOK_URL"
      fi
    - echo "✅ Notification sent"
  only:
    - main
  when: on_success

notify:teams:failure:
  stage: notify
  tags:
    - docs
  script:
    - echo "📢 Sending failure notification to Teams..."
    - |
      if [ -n "$TEAMS_WEBHOOK_URL" ]; then
        curl -H "Content-Type: application/json" \
          -d "{
            \"@type\": \"MessageCard\",
            \"@context\": \"https://schema.org/extensions\",
            \"summary\": \"Documentation Pipeline Failed\",
            \"themeColor\": \"dc3545\",
            \"title\": \"❌ Documentation Pipeline Failed\",
            \"sections\": [{
              \"facts\": [
                {\"name\": \"Project\", \"value\": \"$CI_PROJECT_NAME\"},
                {\"name\": \"Branch\", \"value\": \"$CI_COMMIT_REF_NAME\"},
                {\"name\": \"Commit\", \"value\": \"$CI_COMMIT_SHORT_SHA\"},
                {\"name\": \"Author\", \"value\": \"$GITLAB_USER_NAME\"}
              ]
            }],
            \"potentialAction\": [{
              \"@type\": \"OpenUri\",
              \"name\": \"View Failed Pipeline\",
              \"targets\": [{
                \"os\": \"default\",
                \"uri\": \"$CI_PIPELINE_URL\"
              }]
            }]
          }" \
          "$TEAMS_WEBHOOK_URL"
      fi
    - echo "✅ Failure notification sent"
  only:
    - main
  when: on_failure
EOF

# Commit y push
git add .gitlab-ci.yml
git commit -m "Add CI/CD pipeline"
git push origin main
```

### 3.7 Verificar Pipeline

1. **Ver pipeline en GitLab:**
   - Proyecto → CI/CD → Pipelines
   - Debería aparecer pipeline ejecutándose

2. **Ver logs:**
   - Click en pipeline → Ver cada job
   - Verificar que todos pasan

3. **Solucionar errores comunes:**
   - Si falla por linters no instalados → Reinstalar en runner
   - Si falla por permisos → Verificar gitlab-runner user

---

## 4. Linting Automatizado

### 4.1 Configurar Markdownlint

**Crear `.markdownlint.json` en raíz del proyecto:**

```bash
cat > .markdownlint.json << 'EOF'
{
  "default": true,
  "MD001": true,
  "MD003": { "style": "atx" },
  "MD004": { "style": "dash" },
  "MD007": { "indent": 2 },
  "MD013": {
    "line_length": 120,
    "heading_line_length": 120,
    "code_block_line_length": 120,
    "code_blocks": false,
    "tables": false,
    "headings": true,
    "strict": false,
    "stern": false
  },
  "MD024": { "siblings_only": true },
  "MD025": true,
  "MD026": { "punctuation": ".,;:!" },
  "MD029": { "style": "ordered" },
  "MD033": {
    "allowed_elements": [
      "br",
      "details",
      "summary",
      "kbd",
      "sub",
      "sup"
    ]
  },
  "MD034": false,
  "MD041": false,
  "MD046": { "style": "fenced" },
  "MD049": { "style": "underscore" },
  "MD050": { "style": "asterisk" }
}
EOF

git add .markdownlint.json
git commit -m "Add markdownlint config"
git push
```

### 4.2 Configurar Link Checker

**Crear `.markdown-link-check.json`:**

```bash
cat > .markdown-link-check.json << 'EOF'
{
  "ignorePatterns": [
    {
      "pattern": "^http://localhost"
    },
    {
      "pattern": "^https://localhost"
    },
    {
      "pattern": "^http://127.0.0.1"
    },
    {
      "pattern": "example.com"
    },
    {
      "pattern": "company.com"
    }
  ],
  "replacementPatterns": [
    {
      "pattern": "^/",
      "replacement": "{{BASEURL}}/"
    }
  ],
  "httpHeaders": [
    {
      "urls": ["https://"],
      "headers": {
        "Accept": "text/html",
        "User-Agent": "Mozilla/5.0"
      }
    }
  ],
  "timeout": "10s",
  "retryOn429": true,
  "retryCount": 3,
  "fallbackRetryDelay": "30s",
  "aliveStatusCodes": [200, 206, 301, 302, 307, 308]
}
EOF

git add .markdown-link-check.json
git commit -m "Add link checker config"
git push
```

### 4.3 Configurar Spell Checker

**Crear `cspell.json`:**

```bash
cat > cspell.json << 'EOF'
{
  "version": "0.2",
  "language": "en,es",
  "words": [
    "gitlab",
    "kubernetes",
    "devops",
    "cicd",
    "yaml",
    "nginx",
    "postgres",
    "redis",
    "dockerfile",
    "readme",
    "chmod",
    "sudo",
    "mkdir",
    "terraform",
    "ansible",
    "vmware",
    "hyperv",
    "onprem"
  ],
  "ignorePaths": [
    "node_modules/**",
    ".git/**",
    "vendor/**",
    "*.min.*",
    "package-lock.json"
  ],
  "flagWords": [],
  "dictionaries": [
    "companies",
    "softwareTerms",
    "misc"
  ]
}
EOF

git add cspell.json
git commit -m "Add spell checker config"
git push
```

### 4.4 Configurar YAML Linter

**Crear `.yamllint`:**

```bash
cat > .yamllint << 'EOF'
---
extends: default

rules:
  line-length:
    max: 120
    level: warning
  indentation:
    spaces: 2
  document-start: disable
  truthy:
    allowed-values: ['true', 'false', 'yes', 'no']
  comments:
    min-spaces-from-content: 1
EOF

git add .yamllint
git commit -m "Add YAML linter config"
git push
```

### 4.5 Crear Pre-commit Hook (Opcional local)

**Para developers que quieran validar antes de push:**

```bash
# En el repositorio local
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

echo "🔍 Running pre-commit checks..."

# Check markdown files
echo "📝 Linting markdown..."
git diff --cached --name-only --diff-filter=ACM | grep "\.md$" | \
  xargs markdownlint 2>/dev/null || {
    echo "❌ Markdown linting failed"
    echo "Run: markdownlint --fix <file> to auto-fix"
    exit 1
  }

echo "✅ Pre-commit checks passed"
exit 0
EOF

chmod +x .git/hooks/pre-commit
```

---

## 5. Cadena de Aprobación

### 5.1 Configurar Branch Protection

**En GitLab Web UI:**

1. **Ir a Settings → Repository → Protected Branches**

2. **Proteger `main` branch:**
   - Branch: `main`
   - Allowed to merge: Maintainers
   - Allowed to push: No one
   - Allowed to force push: No
   - Code owner approval: ✅ Enable
   - Click "Protect"

3. **Proteger `production` branch (si existe):**
   - Misma configuración que main

### 5.2 Crear CODEOWNERS File

**Crear archivo `CODEOWNERS` en raíz:**

```bash
cat > CODEOWNERS << 'EOF'
# GitLab CODEOWNERS File
# Syntax: path/to/file @username @group
# Docs: https://docs.gitlab.com/ee/user/project/code_owners.html

# ==========================================
# GLOBAL OWNERS
# ==========================================
# These users can approve any change
* @architecture-team @tech-lead

# ==========================================
# TEMPLATES
# ==========================================
# Changes to templates require architecture team approval
/00-templates/ @architecture-team @technical-writer

# ==========================================
# GLOBAL POLICIES
# ==========================================
/01-global/ @architecture-team @security-team @compliance-team
/01-global/security-standards.md @security-team
/01-global/compliance/ @compliance-team @legal-team

# ==========================================
# INFRASTRUCTURE BY DEPARTMENT
# ==========================================

# Hardware
/02-infrastructure/hardware/ @infrastructure-team @hardware-lead

# Virtualization
/02-infrastructure/virtualization/ @virtualization-team @vmware-admin

# WAN
/02-infrastructure/wan/ @network-team @wan-lead

# LAN
/02-infrastructure/lan/ @network-team @lan-lead

# Windows
/02-infrastructure/windows/ @windows-team @ad-admin

# Linux
/02-infrastructure/linux/ @linux-team @linux-lead

# DevOps / Kubernetes
/02-infrastructure/devops/ @devops-team @k8s-admin

# Cloud Azure
/02-infrastructure/cloud-azure/ @cloud-team @azure-architect

# ==========================================
# SERVICES
# ==========================================
# Each service has its owner
/03-services/authentication-service/ @auth-team @auth-owner
/03-services/payment-gateway/ @payments-team @payments-owner
/03-services/user-management/ @users-team @users-owner

# Add more services as needed
# /03-services/new-service/ @team @owner

# ==========================================
# CROSS-CUTTING CONCERNS
# ==========================================
/04-cross-cutting/security/ @security-team
/04-cross-cutting/monitoring/ @sre-team @devops-team
/04-cross-cutting/compliance/ @compliance-team @legal-team
/04-cross-cutting/backup-recovery/ @sre-team @infrastructure-team

# ==========================================
# ARCHITECTURE DECISIONS
# ==========================================
# ADRs require architecture team
/06-decisions/ @architecture-team @principal-engineers

# ==========================================
# INTEGRATION DOCS
# ==========================================
# Multiple teams involved
/07-integration/ @architecture-team
/07-integration/cloud-to-onprem/ @cloud-team @network-team
/07-integration/network-integration/ @network-team @infrastructure-team
EOF

git add CODEOWNERS
git commit -m "Add CODEOWNERS file"
git push
```

### 5.3 Crear Grupos y Usuarios

**En GitLab Web UI:**

1. **Crear Grupos:**
   - Admin Area → Groups → New group
   - Crear grupos: `architecture-team`, `network-team`, `devops-team`, etc.

2. **Añadir Usuarios a Grupos:**
   - Grupo → Members → Invite members
   - Asignar roles: Maintainer, Developer, Reporter

3. **Verificar estructura:**
   ```
   documentation/
   ├── architecture-team (Group)
   │   ├── alice (Maintainer)
   │   └── bob (Developer)
   ├── network-team (Group)
   │   ├── charlie (Maintainer)
   │   └── diana (Developer)
   └── devops-team (Group)
       ├── eve (Maintainer)
       └── frank (Developer)
   ```

### 5.4 Configurar Merge Request Approvals

**En proyecto → Settings → Merge requests:**

1. **Merge request approvals:**
   - Approvals required before merging: 1
   - ✅ Prevent approval by author
   - ✅ Prevent approvals by users who add commits
   - ✅ Prevent editing approval rules
   - ✅ Require code owner approval

2. **Merge checks:**
   - ✅ Pipelines must succeed
   - ✅ All threads must be resolved
   - ❌ Status checks must succeed (optional)

3. **Merge suggestions:**
   - ✅ Enable "Delete source branch" option by default
   - Merge method: Merge commit (recommended)

### 5.5 Workflow de Aprobación

**Proceso estándar:**

```
Developer → Crea MR → CI Pipeline → Code Owner Review → Approve → Merge
```

**Ejemplo práctico:**

```bash
# Developer trabaja en su feature branch
git checkout -b docs/update-kubernetes-architecture
vim 02-infrastructure/devops/architecture/kubernetes-architecture.md
git add .
git commit -m "docs: Update Kubernetes architecture for 1.28"
git push origin docs/update-kubernetes-architecture

# En GitLab Web:
# 1. Create Merge Request
# 2. Pipeline ejecuta automáticamente
# 3. CODEOWNERS (devops-team) son notificados automáticamente
# 4. Reviewer revisa y aprueba
# 5. Developer (o reviewer) hace merge
```

---

## 6. Integración con Microsoft Teams

### 6.1 Crear Incoming Webhook en Teams

**Pasos en Teams:**

1. **Abrir Teams → Seleccionar canal (ej: #documentation)**

2. **Click en "..." → Connectors → Configure**

3. **Buscar "Incoming Webhook" → Add**

4. **Configurar webhook:**
   - Name: GitLab Documentation
   - Upload image: (opcional, logo de GitLab)
   - Click "Create"

5. **Copiar webhook URL:**
   ```
   https://company.webhook.office.com/webhookb2/xxx-xxx-xxx/IncomingWebhook/yyy-yyy-yyy
   ```
   
   **⚠️ IMPORTANTE:** Guardar esta URL de forma segura

### 6.2 Configurar Webhook en GitLab

**Opción 1: Como CI/CD Variable (Recomendado)**

En GitLab proyecto → Settings → CI/CD → Variables:

```
Key: TEAMS_WEBHOOK_URL
Value: https://company.webhook.office.com/webhookb2/xxx...
Protected: ✅ Yes
Masked: ✅ Yes
```

**Opción 2: Como Project Webhook**

En Settings → Webhooks:

1. **Add new webhook**
2. URL: (usar un intermediario, ver script abajo)
3. Trigger: Push events, Merge request events, Pipeline events
4. SSL verification: ✅ Enable

### 6.3 Crear Script de Notificación

**Crear `scripts/notify-teams.sh`:**

```bash
mkdir -p scripts
cat > scripts/notify-teams.sh << 'EOF'
#!/bin/bash
# Script para enviar notificaciones a Microsoft Teams
# Usage: ./notify-teams.sh [success|failure|merge|custom]

set -e

# Configuración
WEBHOOK_URL="${TEAMS_WEBHOOK_URL}"
if [ -z "$WEBHOOK_URL" ]; then
  echo "Error: TEAMS_WEBHOOK_URL no está configurado"
  exit 1
fi

# Función para enviar mensaje
send_teams_message() {
  local color=$1
  local title=$2
  local summary=$3
  shift 3
  local facts=("$@")
  
  # Construir JSON de facts
  local facts_json=""
  for fact in "${facts[@]}"; do
    IFS='=' read -r name value <<< "$fact"
    facts_json+="{ \"name\": \"$name\", \"value\": \"$value\" },"
  done
  facts_json=${facts_json%,}  # Remove trailing comma
  
  # Payload JSON
  local payload=$(cat <<JSON
{
  "@type": "MessageCard",
  "@context": "https://schema.org/extensions",
  "summary": "$summary",
  "themeColor": "$color",
  "title": "$title",
  "sections": [{
    "facts": [ $facts_json ]
  }],
  "potentialAction": [{
    "@type": "OpenUri",
    "name": "View in GitLab",
    "targets": [{
      "os": "default",
      "uri": "$CI_PIPELINE_URL"
    }]
  }]
}
JSON
)
  
  # Enviar a Teams
  curl -H "Content-Type: application/json" \
    -d "$payload" \
    "$WEBHOOK_URL"
}

# Procesar argumentos
case "${1:-custom}" in
  success)
    send_teams_message \
      "28a745" \
      "✅ Pipeline Succeeded" \
      "Documentation pipeline passed successfully" \
      "Project=$CI_PROJECT_NAME" \
      "Branch=$CI_COMMIT_REF_NAME" \
      "Commit=$CI_COMMIT_SHORT_SHA" \
      "Author=$GITLAB_USER_NAME" \
      "Duration=$CI_PIPELINE_DURATION seconds"
    ;;
    
  failure)
    send_teams_message \
      "dc3545" \
      "❌ Pipeline Failed" \
      "Documentation pipeline failed" \
      "Project=$CI_PROJECT_NAME" \
      "Branch=$CI_COMMIT_REF_NAME" \
      "Commit=$CI_COMMIT_SHORT_SHA" \
      "Author=$GITLAB_USER_NAME" \
      "Failed Job=$CI_JOB_NAME"
    ;;
    
  merge)
    send_teams_message \
      "007bff" \
      "🔀 Merge Request" \
      "New merge request created" \
      "Project=$CI_PROJECT_NAME" \
      "Branch=$CI_COMMIT_REF_NAME" \
      "Author=$GITLAB_USER_NAME" \
      "Title=$CI_MERGE_REQUEST_TITLE"
    ;;
    
  custom)
    # Custom message (pass as arguments)
    send_teams_message \
      "${2:-007bff}" \
      "${3:-Notification}" \
      "${4:-Custom message from GitLab}" \
      "Project=$CI_PROJECT_NAME" \
      "Author=$GITLAB_USER_NAME"
    ;;
    
  *)
    echo "Usage: $0 [success|failure|merge|custom]"
    exit 1
    ;;
esac

echo "✅ Teams notification sent"
EOF

chmod +x scripts/notify-teams.sh
git add scripts/
git commit -m "Add Teams notification script"
git push
```

### 6.4 Webhook para Eventos Específicos

**Crear webhook intermediario (opcional, para más control):**

```bash
# En el servidor GitLab
sudo nano /usr/local/bin/gitlab-teams-webhook.sh
```

```bash
#!/bin/bash
# Webhook intermediario GitLab → Teams
# Se llama desde GitLab webhook

# Parse JSON input (pasado por GitLab)
read -r payload

# Extraer información
event_type=$(echo "$payload" | jq -r '.object_kind // .event_type')
project_name=$(echo "$payload" | jq -r '.project.name // .project_name')
user_name=$(echo "$payload" | jq -r '.user_name // .user.name')

# Teams webhook URL (from environment)
TEAMS_URL="${TEAMS_WEBHOOK_URL}"

# Enviar según el tipo de evento
case "$event_type" in
  "push")
    ref=$(echo "$payload" | jq -r '.ref')
    commits=$(echo "$payload" | jq -r '.total_commits_count')
    curl -H "Content-Type: application/json" -d "{
      \"@type\": \"MessageCard\",
      \"@context\": \"https://schema.org/extensions\",
      \"summary\": \"New push to $project_name\",
      \"themeColor\": \"0078D4\",
      \"title\": \"📤 Push to $ref\",
      \"sections\": [{
        \"facts\": [
          {\"name\": \"Project\", \"value\": \"$project_name\"},
          {\"name\": \"Branch\", \"value\": \"$ref\"},
          {\"name\": \"Commits\", \"value\": \"$commits\"},
          {\"name\": \"Author\", \"value\": \"$user_name\"}
        ]
      }]
    }" "$TEAMS_URL"
    ;;
    
  "merge_request")
    action=$(echo "$payload" | jq -r '.object_attributes.action')
    mr_title=$(echo "$payload" | jq -r '.object_attributes.title')
    curl -H "Content-Type: application/json" -d "{
      \"@type\": \"MessageCard\",
      \"@context\": \"https://schema.org/extensions\",
      \"summary\": \"Merge Request $action\",
      \"themeColor\": \"28A745\",
      \"title\": \"🔀 MR $action: $mr_title\",
      \"sections\": [{
        \"facts\": [
          {\"name\": \"Project\", \"value\": \"$project_name\"},
          {\"name\": \"Author\", \"value\": \"$user_name\"},
          {\"name\": \"Action\", \"value\": \"$action\"}
        ]
      }]
    }" "$TEAMS_URL"
    ;;
esac
```

**Hacer ejecutable:**

```bash
sudo chmod +x /usr/local/bin/gitlab-teams-webhook.sh
```

**Configurar en GitLab webhook:**
- URL: `http://localhost/gitlab-teams-webhook` (requiere nginx proxy)
- Trigger: Push, MR, Pipeline

### 6.5 Probar Integración

```bash
# Test simple
./scripts/notify-teams.sh success

# Debería aparecer mensaje en Teams channel
```

**Verificar en Teams:**
- Debería aparecer card con información del pipeline

---

## 7. Notion Setup

### 7.1 Crear Workspace de Notion

1. **Crear cuenta:** https://notion.so (free tier suficiente)
2. **Crear workspace:** "Architecture Documentation"
3. **Invitar team members**

### 7.2 Estructura de Notion

**Crear esta jerarquía de páginas:**

```
Architecture Documentation (Workspace)
├── 📚 Documentation Home
│   ├── Getting Started
│   ├── Contributing Guide
│   └── Templates
├── 🏢 Global
│   ├── Architecture Principles
│   ├── Security Standards
│   └── Compliance
├── 🖥️ Infrastructure
│   ├── Hardware
│   ├── Virtualization
│   ├── WAN
│   ├── LAN
│   ├── Windows
│   ├── Linux
│   ├── DevOps
│   └── Cloud Azure
├── 🚀 Services
│   └── [Service entries here]
└── 📊 Metrics Dashboard
    ├── Coverage
    └── Quality Scores
```

### 7.3 Sincronización GitLab → Notion

**Opción 1: Manual Sync (Simple)**

Proceso:
1. Editar en GitLab (Git workflow)
2. Aprobar MR
3. Copiar contenido a Notion
4. Notion sirve como "pretty view" para management

**Opción 2: Notion API (Avanzado)**

```bash
# Requiere Notion API token
# Ver documentación: https://developers.notion.com/

# Ejemplo de script de sync
cat > scripts/sync-to-notion.sh << 'EOF'
#!/bin/bash
# Sync GitLab markdown to Notion pages

NOTION_TOKEN="${NOTION_API_TOKEN}"
NOTION_DATABASE_ID="${NOTION_DATABASE_ID}"

# Función para convertir markdown a Notion blocks
sync_file_to_notion() {
  local file=$1
  local notion_page_id=$2
  
  # Convert markdown to Notion format
  # (Requiere herramienta de conversión como md-to-notion)
  
  echo "Syncing $file to Notion page $notion_page_id"
  # Implementar aquí
}

# Sync all changed files
git diff --name-only HEAD~1 HEAD | grep "\.md$" | while read file; do
  sync_file_to_notion "$file" "page-id"
done
EOF
```

**Nota:** Sincronización automática GitLab → Notion es compleja. Recomiendo:
- Git como source of truth
- Notion como read-only view
- Sync manual o semi-automático

### 7.4 Embeds y Links

**En Notion, enlazar a GitLab:**

```markdown
# En cada página de Notion
[📝 Edit in GitLab](https://gitlab.company.com/documentation/architecture-docs/-/blob/main/path/to/file.md)

[📊 View History](https://gitlab.company.com/documentation/architecture-docs/-/commits/main/path/to/file.md)
```

**En GitLab, enlazar a Notion:**

```markdown
# En README.md
[📖 View in Notion](https://notion.so/company/page-id)
```

---

## 8. Backup y Monitoring

### 8.1 Backup de GitLab

**Configurar backup automático:**

```bash
# Editar configuración de GitLab
sudo nano /etc/gitlab/gitlab.rb

# Añadir/modificar estas líneas:
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
gitlab_rails['backup_keep_time'] = 604800  # 7 días

# Configurar cron para backup diario
sudo crontab -e

# Añadir esta línea (backup a las 2 AM)
0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1

# Reconfigurar
sudo gitlab-ctl reconfigure
```

**Hacer backup manual:**

```bash
# Crear backup
sudo gitlab-backup create

# Backups se guardan en:
# /var/opt/gitlab/backups/

# Listar backups
ls -lh /var/opt/gitlab/backups/
```

**Restaurar backup:**

```bash
# Stop servicios
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq

# Restaurar
sudo gitlab-backup restore BACKUP=1613123456_2024_02_15_16.8.1

# Restart servicios
sudo gitlab-ctl restart

# Verificar
sudo gitlab-rake gitlab:check SANITIZE=true
```

### 8.2 Backup Offsite

**Sync a almacenamiento externo:**

```bash
# Crear script de backup
sudo nano /usr/local/bin/gitlab-backup-offsite.sh
```

```bash
#!/bin/bash
# Sync backups a almacenamiento remoto

BACKUP_DIR="/var/opt/gitlab/backups"
REMOTE_DIR="user@backup-server:/backups/gitlab"

# Sync vía rsync
rsync -avz --delete \
  "$BACKUP_DIR/" \
  "$REMOTE_DIR"

# O usar s3cmd para AWS S3
# s3cmd sync "$BACKUP_DIR/" s3://company-backups/gitlab/

echo "Backup synced to offsite storage"
```

```bash
# Hacer ejecutable
sudo chmod +x /usr/local/bin/gitlab-backup-offsite.sh

# Añadir a cron (después del backup local)
sudo crontab -e
# 0 3 * * * /usr/local/bin/gitlab-backup-offsite.sh
```

### 8.3 Monitoring de GitLab

**Prometheus (incluido en GitLab):**

```bash
# Habilitar métricas de Prometheus
sudo nano /etc/gitlab/gitlab.rb

# Verificar que esté habilitado:
prometheus_monitoring['enable'] = true
prometheus['monitor_kubernetes'] = false

# Reconfigurar
sudo gitlab-ctl reconfigure

# Acceder a Prometheus
# https://gitlab.company.com/-/metrics
```

**Grafana (opcional, externo):**

```bash
# Instalar Grafana
sudo apt install -y software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana

# Iniciar Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Acceder en http://servidor:3000
# User: admin / Password: admin
```

**Añadir GitLab como datasource:**
1. Grafana → Configuration → Data Sources
2. Add data source → Prometheus
3. URL: `http://localhost:9090`
4. Save & Test

**Importar dashboard de GitLab:**
1. Dashboards → Import
2. ID: 9524 (GitLab Omnibus)
3. Load → Import

### 8.4 Alerting

**Crear alertas básicas:**

```bash
# En Prometheus, crear alert rules
sudo nano /var/opt/gitlab/prometheus/alerts.yml
```

```yaml
groups:
  - name: gitlab_docs
    interval: 30s
    rules:
      - alert: GitLabDown
        expr: up{job="gitlab"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "GitLab is down"
          
      - alert: HighCPU
        expr: node_cpu_seconds_total{mode="idle"} < 20
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on GitLab server"
          
      - alert: DiskSpaceLow
        expr: node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space is below 10%"
```

**Configurar notificaciones a Teams:**

```bash
# Prometheus Alertmanager config
sudo nano /var/opt/gitlab/alertmanager/alertmanager.yml
```

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  receiver: 'teams-webhook'

receivers:
  - name: 'teams-webhook'
    webhook_configs:
      - url: 'https://company.webhook.office.com/webhookb2/xxx...'
        send_resolved: true
```

---

## 9. Troubleshooting

### 9.1 GitLab No Inicia

**Síntomas:** `sudo gitlab-ctl status` muestra servicios down

**Soluciones:**

```bash
# 1. Check logs
sudo gitlab-ctl tail

# 2. Verificar memoria
free -h
# Si memoria baja, aumentar swap

# 3. Reconfigure
sudo gitlab-ctl reconfigure

# 4. Restart servicios uno por uno
sudo gitlab-ctl restart postgresql
sudo gitlab-ctl restart redis
sudo gitlab-ctl restart puma

# 5. Check permisos
sudo gitlab-rake gitlab:check

# 6. Si todo falla, reinstalar
sudo apt remove --purge gitlab-ce
# Restore from backup
```

### 9.2 Pipeline Falla

**Síntomas:** CI pipeline muestra errores

**Soluciones:**

```bash
# 1. Verificar runner está activo
sudo gitlab-runner list
sudo gitlab-runner verify

# 2. Check runner logs
sudo journalctl -u gitlab-runner -f

# 3. Verificar herramientas instaladas
markdownlint --version
markdown-link-check --version

# 4. Test manual del pipeline
# Clone repo
git clone https://gitlab.company.com/...
cd architecture-docs

# Run lint manually
markdownlint **/*.md

# 5. Revisar .gitlab-ci.yml syntax
# En GitLab UI: CI/CD → Editor → Validate
```

### 9.3 CODEOWNERS No Funciona

**Síntomas:** MR no requiere aprobación de code owners

**Soluciones:**

```bash
# 1. Verificar branch protection está activa
# Settings → Repository → Protected Branches
# "Code owner approval" debe estar ✅

# 2. Verificar CODEOWNERS syntax
# Debe estar en raíz del repo
# Verificar que usuarios/grupos existen

# 3. Verificar permisos
# Code owners deben tener al menos Developer role

# 4. Test con cambio simple
echo "test" >> test.md
git add test.md
git commit -m "test"
git push
# Crear MR y verificar que pide aprobación
```

### 9.4 Teams Webhook No Envía

**Síntomas:** No llegan notificaciones a Teams

**Soluciones:**

```bash
# 1. Verificar variable está configurada
# GitLab → Settings → CI/CD → Variables
# TEAMS_WEBHOOK_URL debe existir

# 2. Test manual
curl -H "Content-Type: application/json" \
  -d '{"text":"Test from GitLab"}' \
  "$TEAMS_WEBHOOK_URL"
# Debe aparecer en Teams

# 3. Verificar firewall no bloquea outbound
curl -v https://company.webhook.office.com

# 4. Check logs de pipeline
# Ver output de notify:teams jobs

# 5. Verificar webhook no expiró
# Regenerar en Teams si es necesario
```

### 9.5 Let's Encrypt Falla

**Síntomas:** Certificado SSL no se obtiene

**Soluciones:**

```bash
# 1. Verificar DNS apunta a servidor
nslookup gitlab.company.com
ping gitlab.company.com

# 2. Verificar puerto 80 accesible externamente
sudo ufw status
# Port 80 debe estar ALLOW

# 3. Manual cert request
sudo gitlab-ctl renew-le-certs

# 4. Ver logs
sudo gitlab-ctl tail nginx

# 5. Si falla, usar cert manual
# Obtener cert de otro provider
# Configurar en /etc/gitlab/gitlab.rb:
nginx['ssl_certificate'] = "/path/to/cert.pem"
nginx['ssl_certificate_key'] = "/path/to/key.pem"
```

### 9.6 Performance Issues

**Síntomas:** GitLab lento, timeouts

**Soluciones:**

```bash
# 1. Check recursos
top
free -h
df -h

# 2. Optimize PostgreSQL
sudo gitlab-rake gitlab:db:optimize
sudo gitlab-rake db:repack

# 3. Limpiar artifacts viejos
# Settings → CI/CD → Job artifacts → Max size: 1GB
# Settings → CI/CD → Job artifacts → Expire after: 30 days

# 4. Garbage collection
sudo gitlab-rake gitlab:cleanup:repos
sudo gitlab-rake gitlab:cleanup:orphaned_projects

# 5. Aumentar workers
sudo nano /etc/gitlab/gitlab.rb
puma['worker_processes'] = 4
sudo gitlab-ctl reconfigure

# 6. Considerar RAM upgrade si <8GB
```

---

## 📋 Post-Installation Checklist

### Verificación Final

```bash
# 1. GitLab accessible
curl -I https://gitlab.company.com
# Expected: HTTP/2 200

# 2. Puede hacer login
# Browser → https://gitlab.company.com
# Login with root

# 3. Runner registrado y activo
sudo gitlab-runner list
# Expected: 1 runner listed

# 4. Pipeline ejecuta correctamente
# Crear test MR → Ver pipeline pasa

# 5. CODEOWNERS funciona
# Test MR requiere aprobación del owner

# 6. Teams recibe notificaciones
./scripts/notify-teams.sh success
# Check Teams channel

# 7. Backups configurados
sudo crontab -l | grep gitlab-backup
# Expected: cron job existe

# 8. Monitoring accessible
# https://gitlab.company.com/-/metrics
# Prometheus muestra métricas

# 9. SSL válido
openssl s_client -connect gitlab.company.com:443
# Verify certificate

# 10. Notion workspace creado
# Browser → notion.so
# Structure exists
```

---

## 📊 Métricas de Éxito

Después de implementación, trackear:

| Métrica | Meta | Cómo medir |
|---------|------|------------|
| Uptime GitLab | >99% | Prometheus uptime |
| Pipeline success rate | >95% | GitLab → CI/CD → Analytics |
| Avg pipeline duration | <5 min | Pipeline history |
| MR approval time | <24h | MR analytics |
| Backup success rate | 100% | Check cron logs |
| Teams notifications sent | 100% | Check Teams channel |

---

## 🚀 Próximos Pasos

1. **Poblar repositorio con templates**
   ```bash
   # Copiar templates creados anteriormente
   mkdir -p 00-templates
   cp path/to/templates/*.md 00-templates/
   git add 00-templates/
   git commit -m "Add documentation templates"
   git push
   ```

2. **Documentar primer servicio piloto**
   - Crear carpeta en 03-services/
   - Usar templates
   - Crear MR
   - Probar workflow completo

3. **Training para usuarios**
   - Workshop de Git/GitLab básico
   - Demo de crear MR
   - Explicar proceso de aprobación

4. **Iterar basado en feedback**
   - Ajustar CODEOWNERS
   - Refinar pipeline
   - Mejorar linters

---

## 📞 Soporte y Recursos

**Documentación Oficial:**
- GitLab: https://docs.gitlab.com/
- GitLab CI: https://docs.gitlab.com/ee/ci/
- Notion: https://notion.so/help
- Markdownlint: https://github.com/DavidAnson/markdownlint

**Comunidad:**
- GitLab Forum: https://forum.gitlab.com/
- Stack Overflow: Tag [gitlab]

**Contacto Interno:**
- Infra team: infra@company.com
- DevOps team: devops@company.com
- Doc team: docs@company.com

---

**Versión:** 1.0  
**Última actualización:** 2024-03-15  
**Autor:** SRE/DevOps Team  
**Estado:** ✅ Listo para implementación
