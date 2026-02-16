# Scripts de Utilidad - Gestión de Documentación
## Colección de Scripts Helper

> **Propósito:** Scripts auxiliares para gestión diaria de la documentación  
> **Ubicación:** `/scripts/` en el repositorio

---

## 📋 Índice de Scripts

1. [validate-docs.sh](#1-validate-docssh) - Validación completa local
2. [new-service.sh](#2-new-servicesh) - Scaffold servicio nuevo
3. [generate-metrics.py](#3-generate-metricspy) - Dashboard de métricas
4. [check-staleness.sh](#4-check-stalenesssh) - Detectar docs obsoletos
5. [migrate-docs.sh](#5-migrate-docssh) - Migrar docs existentes
6. [search-docs.sh](#6-search-docssh) - Búsqueda rápida
7. [backup-to-s3.sh](#7-backup-to-s3sh) - Backup a cloud

---

## 1. validate-docs.sh

**Propósito:** Ejecutar todas las validaciones localmente antes de push

```bash
#!/bin/bash
# validate-docs.sh - Validación completa de documentación
# Usage: ./scripts/validate-docs.sh [path]

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Default path
PATH_TO_CHECK="${1:-.}"

echo "🔍 Validando documentación en: $PATH_TO_CHECK"
echo ""

# Counter de errores
ERRORS=0

# 1. Markdown linting
echo "📝 1/6 Linting Markdown..."
if find "$PATH_TO_CHECK" -name "*.md" -type f \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  | xargs markdownlint -c .markdownlint.json 2>/dev/null; then
  echo -e "${GREEN}✓${NC} Markdown lint passed"
else
  echo -e "${RED}✗${NC} Markdown lint failed"
  ((ERRORS++))
fi
echo ""

# 2. Link checking
echo "🔗 2/6 Checking links..."
BROKEN_LINKS=0
find "$PATH_TO_CHECK" -name "*.md" -type f \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  | while IFS= read -r file; do
    if ! markdown-link-check "$file" -q -c .markdown-link-check.json 2>/dev/null; then
      echo -e "${RED}✗${NC} Broken links in: $file"
      ((BROKEN_LINKS++))
    fi
  done

if [ $BROKEN_LINKS -eq 0 ]; then
  echo -e "${GREEN}✓${NC} No broken links found"
else
  echo -e "${YELLOW}⚠${NC} Found $BROKEN_LINKS files with broken links"
  ((ERRORS++))
fi
echo ""

# 3. Spell checking
echo "📖 3/6 Checking spelling..."
if find "$PATH_TO_CHECK" -name "*.md" -type f \
  -not -path "*/node_modules/*" \
  | xargs cspell --no-progress 2>/dev/null; then
  echo -e "${GREEN}✓${NC} Spelling check passed"
else
  echo -e "${YELLOW}⚠${NC} Spelling issues found (review manually)"
fi
echo ""

# 4. Structure validation
echo "📁 4/6 Validating structure..."
REQUIRED_DIRS=("00-templates" "01-global" "02-infrastructure" "03-services")
MISSING_DIRS=0

for dir in "${REQUIRED_DIRS[@]}"; do
  if [ ! -d "$dir" ]; then
    echo -e "${RED}✗${NC} Missing required directory: $dir"
    ((MISSING_DIRS++))
  fi
done

if [ $MISSING_DIRS -eq 0 ]; then
  echo -e "${GREEN}✓${NC} Structure validation passed"
else
  echo -e "${RED}✗${NC} Missing $MISSING_DIRS required directories"
  ((ERRORS++))
fi
echo ""

# 5. Template completeness
echo "📄 5/6 Checking template completeness..."
REQUIRED_TEMPLATES=(
  "00-templates/01-architecture-design.md"
  "00-templates/02-deployment-runbook.md"
  "00-templates/03-service-ownership.md"
  "00-templates/04-observability.md"
)
MISSING_TEMPLATES=0

for template in "${REQUIRED_TEMPLATES[@]}"; do
  if [ ! -f "$template" ]; then
    echo -e "${RED}✗${NC} Missing template: $template"
    ((MISSING_TEMPLATES++))
  fi
done

if [ $MISSING_TEMPLATES -eq 0 ]; then
  echo -e "${GREEN}✓${NC} All templates present"
else
  echo -e "${RED}✗${NC} Missing $MISSING_TEMPLATES templates"
  ((ERRORS++))
fi
echo ""

# 6. YAML validation
echo "📋 6/6 Validating YAML files..."
if find "$PATH_TO_CHECK" -name "*.yml" -o -name "*.yaml" -type f \
  | xargs yamllint -c .yamllint 2>/dev/null; then
  echo -e "${GREEN}✓${NC} YAML validation passed"
else
  echo -e "${YELLOW}⚠${NC} YAML validation issues found"
fi
echo ""

# Summary
echo "=================================="
if [ $ERRORS -eq 0 ]; then
  echo -e "${GREEN}✓ All validations passed!${NC}"
  exit 0
else
  echo -e "${RED}✗ $ERRORS validation(s) failed${NC}"
  exit 1
fi
```

**Uso:**

```bash
# Validar todo
./scripts/validate-docs.sh

# Validar carpeta específica
./scripts/validate-docs.sh 02-infrastructure/devops/

# En pre-commit hook
ln -s ../../scripts/validate-docs.sh .git/hooks/pre-commit
```

---

## 2. new-service.sh

**Propósito:** Crear estructura completa para un nuevo servicio

```bash
#!/bin/bash
# new-service.sh - Scaffold para nuevo servicio
# Usage: ./scripts/new-service.sh <service-name> <owner-team>

set -e

# Validar argumentos
if [ $# -lt 2 ]; then
  echo "Usage: $0 <service-name> <owner-team>"
  echo "Example: $0 authentication-service auth-team"
  exit 1
fi

SERVICE_NAME="$1"
OWNER_TEAM="$2"
SERVICE_DIR="03-services/$SERVICE_NAME"

# Verificar que no existe
if [ -d "$SERVICE_DIR" ]; then
  echo "Error: Service directory already exists: $SERVICE_DIR"
  exit 1
fi

echo "🚀 Creating new service: $SERVICE_NAME"
echo "   Owner team: $OWNER_TEAM"
echo ""

# Crear estructura
mkdir -p "$SERVICE_DIR/diagrams"

# Copiar templates y customizar
echo "📄 Creating documentation from templates..."

# 1. Architecture Design
cat > "$SERVICE_DIR/01-architecture-design.md" << EOF
# $SERVICE_NAME - Architecture Design

> **Service Owner:** @$OWNER_TEAM  
> **Version:** 1.0  
> **Date:** $(date +%Y-%m-%d)  
> **Status:** Draft

---

## 1. Executive Summary

**Purpose:**
[Describe the business purpose of this service]

**Scope:**
[What this service does and does NOT do]

**Stakeholders:**
| Role | Name | Team | Contact |
|------|------|------|---------|
| Product Owner | [Name] | [Team] | [Email] |
| Service Owner | [Name] | $OWNER_TEAM | [Email] |
| Tech Lead | [Name] | [Team] | [Email] |

---

## 2. Requirements

### 2.1 Functional Requirements

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-001 | [Description] | Must | 📝 Planned |

### 2.2 Non-Functional Requirements

#### Performance
- **Throughput:** [X requests/second]
- **Latency:** p95 < [X ms]

#### Availability
- **Target:** 99.9%
- **Maintenance Windows:** [Day/time]

#### Security
- **Authentication:** [Method]
- **Compliance:** [GDPR, SOC2, etc.]

---

## 3. Architecture

### 3.1 High-Level Diagram

\`\`\`
[Add architecture diagram]
\`\`\`

### 3.2 Components

[Describe main components]

---

## 4. Service Level Objectives

| SLI | Target | Measurement |
|-----|--------|-------------|
| Availability | 99.9% | 30 days |
| Latency (p95) | < 300ms | 30 days |
| Error Rate | < 0.1% | 30 days |

---

## 5. Dependencies

### Upstream (We depend on)
| Service | Criticality | Contact |
|---------|-------------|---------|
| [Service] | [Critical/Medium/Low] | [Email] |

### Downstream (Depends on us)
| Service | Owner | Impact if down |
|---------|-------|----------------|
| [Service] | [Team] | [Description] |

---

## 6. References

- [Deployment Guide](./02-deployment-runbook.md)
- [Service Ownership](./03-service-ownership.md)
- [Monitoring Setup](./04-observability.md)
EOF

# 2. Deployment Runbook
cat > "$SERVICE_DIR/02-deployment-runbook.md" << EOF
# $SERVICE_NAME - Deployment Runbook

> **Service Owner:** @$OWNER_TEAM  
> **Version:** 1.0

---

## Prerequisites

- [ ] Access to production environment
- [ ] kubectl access to cluster
- [ ] Credentials configured

---

## Deployment Procedure

### 1. Pre-deployment Checks

\`\`\`bash
# Verify environment
kubectl get pods -n production

# Check current version
kubectl describe deployment $SERVICE_NAME
\`\`\`

### 2. Deployment Steps

\`\`\`bash
# Deploy new version
kubectl apply -f kubernetes/deployment.yaml

# Monitor rollout
kubectl rollout status deployment/$SERVICE_NAME

# Verify
kubectl get pods -l app=$SERVICE_NAME
\`\`\`

### 3. Rollback Procedure

\`\`\`bash
# Rollback to previous version
kubectl rollout undo deployment/$SERVICE_NAME

# Verify rollback
kubectl rollout status deployment/$SERVICE_NAME
\`\`\`

---

## Runbooks

### Issue: Service Not Starting

**Symptoms:** Pods in CrashLoopBackOff

**Diagnosis:**
\`\`\`bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
\`\`\`

**Resolution:**
[Steps to resolve]

---

## Contacts

- On-call: Via PagerDuty
- Team: @$OWNER_TEAM
- Escalation: [Manager email]
EOF

# 3. Service Ownership
cat > "$SERVICE_DIR/03-service-ownership.md" << EOF
# $SERVICE_NAME - Service Ownership

> **Primary Owner:** @$OWNER_TEAM

---

## Team

| Role | Name | Contact |
|------|------|---------|
| Service Owner | [Name] | [Email] |
| Tech Lead | [Name] | [Email] |
| On-call Primary | [Rotation] | PagerDuty |

---

## RACI Matrix

| Activity | $OWNER_TEAM | SRE | Product |
|----------|-------------|-----|---------|
| Feature Development | R | C | A |
| Deployment | A | R | I |
| Incident Response | R | R | I |

**Legend:** R=Responsible, A=Accountable, C=Consulted, I=Informed

---

## On-Call Rotation

[PagerDuty schedule link]

## Escalation

1. Primary on-call (15 min)
2. Tech Lead (30 min)
3. Service Owner (1 hour)
EOF

# 4. Observability
cat > "$SERVICE_DIR/04-observability.md" << EOF
# $SERVICE_NAME - Observability

> **SRE Lead:** [Name]

---

## SLIs

| Metric | Target | Dashboard |
|--------|--------|-----------|
| Availability | 99.9% | [Link] |
| Latency p95 | < 300ms | [Link] |
| Error Rate | < 0.1% | [Link] |

---

## Dashboards

- **Main Dashboard:** [Grafana link]
- **Alerts:** [PagerDuty link]

---

## Alerts

### Critical Alerts

| Alert | Threshold | Runbook |
|-------|-----------|---------|
| High Error Rate | > 1% for 5min | [Link] |
| Service Down | 0 healthy pods | [Link] |

---

## Logging

**Query Examples:**

\`\`\`
# Find errors in last hour
index=production service=$SERVICE_NAME level=ERROR
| timechart count

# Slow requests
index=production service=$SERVICE_NAME duration_ms>1000
| stats avg(duration_ms) by endpoint
\`\`\`
EOF

# 5. README
cat > "$SERVICE_DIR/README.md" << EOF
# $SERVICE_NAME

> **Owner:** @$OWNER_TEAM  
> **Status:** 🚧 In Development

## Quick Links

- [Architecture Design](./01-architecture-design.md)
- [Deployment Guide](./02-deployment-runbook.md)
- [Service Ownership](./03-service-ownership.md)
- [Monitoring](./04-observability.md)

## Overview

[Brief description of the service]

## Getting Started

[How to run locally, contribute, etc.]

## Tech Stack

- Language: [e.g., Python 3.11]
- Framework: [e.g., FastAPI]
- Database: [e.g., PostgreSQL]
- Infrastructure: [e.g., Kubernetes]
EOF

# Crear directorio para diagramas
touch "$SERVICE_DIR/diagrams/.gitkeep"

# Actualizar CODEOWNERS
echo "" >> CODEOWNERS
echo "# $SERVICE_NAME" >> CODEOWNERS
echo "/$SERVICE_DIR/ @$OWNER_TEAM" >> CODEOWNERS

echo ""
echo "✅ Service structure created successfully!"
echo ""
echo "Next steps:"
echo "1. cd $SERVICE_DIR"
echo "2. Fill in the documentation templates"
echo "3. Add architecture diagrams to diagrams/"
echo "4. git add $SERVICE_DIR CODEOWNERS"
echo "5. git commit -m 'docs: Add $SERVICE_NAME service'"
echo "6. git push origin main"
echo ""
echo "📋 Files created:"
find "$SERVICE_DIR" -type f
```

**Uso:**

```bash
# Crear nuevo servicio
./scripts/new-service.sh payment-gateway payments-team

# Estructura creada:
# 03-services/payment-gateway/
# ├── 01-architecture-design.md
# ├── 02-deployment-runbook.md
# ├── 03-service-ownership.md
# ├── 04-observability.md
# ├── README.md
# └── diagrams/
```

---

## 3. generate-metrics.py

**Propósito:** Generar dashboard de métricas de documentación

```python
#!/usr/bin/env python3
"""
generate-metrics.py - Generate documentation metrics dashboard
Usage: ./scripts/generate-metrics.py [--output html|json|console]
"""

import os
import sys
import json
from datetime import datetime, timedelta
from pathlib import Path
from typing import Dict, List
import subprocess

def count_markdown_files(directory: str = ".") -> int:
    """Count total markdown files"""
    return len(list(Path(directory).rglob("*.md")))

def get_file_age(filepath: str) -> int:
    """Get file age in days"""
    mtime = os.path.getmtime(filepath)
    age = datetime.now() - datetime.fromtimestamp(mtime)
    return age.days

def check_staleness(max_age: int = 90) -> Dict:
    """Check for stale documentation"""
    stale_files = []
    total_files = 0
    
    for md_file in Path(".").rglob("*.md"):
        if ".git" in str(md_file) or "node_modules" in str(md_file):
            continue
            
        total_files += 1
        age = get_file_age(str(md_file))
        
        if age > max_age:
            stale_files.append({
                "file": str(md_file),
                "age_days": age
            })
    
    return {
        "total": total_files,
        "stale": len(stale_files),
        "stale_percentage": round((len(stale_files) / total_files * 100), 2) if total_files > 0 else 0,
        "files": stale_files
    }

def check_service_completeness() -> Dict:
    """Check if services have all required docs"""
    services_dir = Path("03-services")
    if not services_dir.exists():
        return {"error": "Services directory not found"}
    
    required_files = [
        "01-architecture-design.md",
        "02-deployment-runbook.md",
        "03-service-ownership.md",
        "04-observability.md"
    ]
    
    results = []
    for service_dir in services_dir.iterdir():
        if service_dir.is_dir() and not service_dir.name.startswith("."):
            missing = []
            for req_file in required_files:
                if not (service_dir / req_file).exists():
                    missing.append(req_file)
            
            completeness = (len(required_files) - len(missing)) / len(required_files) * 100
            
            results.append({
                "service": service_dir.name,
                "completeness": round(completeness, 2),
                "missing": missing
            })
    
    avg_completeness = sum(r["completeness"] for r in results) / len(results) if results else 0
    
    return {
        "services": len(results),
        "average_completeness": round(avg_completeness, 2),
        "details": results
    }

def count_by_department() -> Dict:
    """Count docs by department"""
    infra_dir = Path("02-infrastructure")
    if not infra_dir.exists():
        return {"error": "Infrastructure directory not found"}
    
    departments = {}
    for dept_dir in infra_dir.iterdir():
        if dept_dir.is_dir():
            count = len(list(dept_dir.rglob("*.md")))
            departments[dept_dir.name] = count
    
    return departments

def run_linting_stats() -> Dict:
    """Get linting statistics"""
    try:
        # Run markdownlint and capture output
        result = subprocess.run(
            ["markdownlint", "**/*.md", "--json"],
            capture_output=True,
            text=True
        )
        
        if result.returncode == 0:
            return {"errors": 0, "warnings": 0}
        else:
            # Parse errors (simplified)
            return {"errors": result.stdout.count("error"), "warnings": result.stdout.count("warning")}
    except Exception as e:
        return {"error": str(e)}

def generate_metrics() -> Dict:
    """Generate all metrics"""
    print("📊 Generating documentation metrics...")
    
    metrics = {
        "generated_at": datetime.now().isoformat(),
        "total_files": count_markdown_files(),
        "staleness": check_staleness(),
        "services": check_service_completeness(),
        "departments": count_by_department(),
        "linting": run_linting_stats()
    }
    
    return metrics

def output_console(metrics: Dict):
    """Output to console"""
    print("\n" + "="*50)
    print("📚 DOCUMENTATION METRICS")
    print("="*50)
    print(f"\n📅 Generated: {metrics['generated_at']}")
    print(f"\n📄 Total Files: {metrics['total_files']}")
    
    print("\n📆 Staleness:")
    print(f"  - Total: {metrics['staleness']['total']}")
    print(f"  - Stale (>90 days): {metrics['staleness']['stale']} ({metrics['staleness']['stale_percentage']}%)")
    
    print("\n🚀 Services:")
    print(f"  - Total Services: {metrics['services']['services']}")
    print(f"  - Avg Completeness: {metrics['services']['average_completeness']}%")
    
    print("\n🏢 By Department:")
    for dept, count in metrics['departments'].items():
        print(f"  - {dept}: {count} files")
    
    print("\n✓ Linting:")
    print(f"  - Errors: {metrics['linting'].get('errors', 'N/A')}")
    print(f"  - Warnings: {metrics['linting'].get('warnings', 'N/A')}")
    print("\n" + "="*50)

def output_html(metrics: Dict):
    """Generate HTML dashboard"""
    html = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Documentation Metrics</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }}
        .container {{ max-width: 1200px; margin: 0 auto; background: white; padding: 20px; border-radius: 8px; }}
        h1 {{ color: #333; border-bottom: 3px solid #007bff; padding-bottom: 10px; }}
        .metric-card {{ background: #f8f9fa; padding: 15px; margin: 10px 0; border-radius: 5px; border-left: 4px solid #007bff; }}
        .metric-title {{ font-weight: bold; font-size: 1.2em; margin-bottom: 10px; }}
        .metric-value {{ font-size: 2em; color: #007bff; }}
        .good {{ color: #28a745; }}
        .warning {{ color: #ffc107; }}
        .bad {{ color: #dc3545; }}
        table {{ width: 100%; border-collapse: collapse; margin: 10px 0; }}
        th, td {{ padding: 10px; text-align: left; border-bottom: 1px solid #ddd; }}
        th {{ background: #007bff; color: white; }}
        .progress-bar {{ width: 100%; height: 20px; background: #e9ecef; border-radius: 10px; overflow: hidden; }}
        .progress-fill {{ height: 100%; background: #28a745; transition: width 0.3s; }}
    </style>
</head>
<body>
    <div class="container">
        <h1>📚 Documentation Metrics Dashboard</h1>
        <p>Generated: {metrics['generated_at']}</p>
        
        <div class="metric-card">
            <div class="metric-title">📄 Total Documentation Files</div>
            <div class="metric-value">{metrics['total_files']}</div>
        </div>
        
        <div class="metric-card">
            <div class="metric-title">📆 Staleness Check (>90 days)</div>
            <div class="metric-value {'good' if metrics['staleness']['stale_percentage'] < 10 else 'warning' if metrics['staleness']['stale_percentage'] < 30 else 'bad'}">
                {metrics['staleness']['stale']} / {metrics['staleness']['total']} ({metrics['staleness']['stale_percentage']}%)
            </div>
            <div class="progress-bar">
                <div class="progress-fill" style="width: {100 - metrics['staleness']['stale_percentage']}%"></div>
            </div>
        </div>
        
        <div class="metric-card">
            <div class="metric-title">🚀 Services Coverage</div>
            <div class="metric-value {'good' if metrics['services']['average_completeness'] > 80 else 'warning' if metrics['services']['average_completeness'] > 60 else 'bad'}">
                {metrics['services']['average_completeness']}%
            </div>
            <p>{metrics['services']['services']} services tracked</p>
            
            <table>
                <tr>
                    <th>Service</th>
                    <th>Completeness</th>
                    <th>Missing</th>
                </tr>
"""
    
    for service in metrics['services']['details']:
        html += f"""
                <tr>
                    <td>{service['service']}</td>
                    <td>{service['completeness']}%</td>
                    <td>{', '.join(service['missing']) if service['missing'] else '✓ Complete'}</td>
                </tr>
"""
    
    html += """
            </table>
        </div>
        
        <div class="metric-card">
            <div class="metric-title">🏢 Documentation by Department</div>
            <table>
                <tr>
                    <th>Department</th>
                    <th>Files</th>
                </tr>
"""
    
    for dept, count in metrics['departments'].items():
        html += f"""
                <tr>
                    <td>{dept}</td>
                    <td>{count}</td>
                </tr>
"""
    
    html += """
            </table>
        </div>
    </div>
</body>
</html>
"""
    
    with open("metrics-dashboard.html", "w") as f:
        f.write(html)
    
    print("✅ HTML dashboard generated: metrics-dashboard.html")

def output_json(metrics: Dict):
    """Output as JSON"""
    with open("metrics.json", "w") as f:
        json.dump(metrics, f, indent=2)
    print("✅ JSON metrics generated: metrics.json")

if __name__ == "__main__":
    output_format = sys.argv[1] if len(sys.argv) > 1 else "console"
    
    metrics = generate_metrics()
    
    if output_format == "html":
        output_html(metrics)
    elif output_format == "json":
        output_json(metrics)
    else:
        output_console(metrics)
```

**Uso:**

```bash
# Ver en consola
./scripts/generate-metrics.py

# Generar HTML dashboard
./scripts/generate-metrics.py html
# Abre metrics-dashboard.html en browser

# Generar JSON (para integrar con otros sistemas)
./scripts/generate-metrics.py json
# Crea metrics.json
```

---

## 4. check-staleness.sh

**Propósito:** Detectar y reportar documentación obsoleta

```bash
#!/bin/bash
# check-staleness.sh - Find stale documentation
# Usage: ./scripts/check-staleness.sh [days] [--notify]

DAYS_THRESHOLD=${1:-90}
NOTIFY=${2}

echo "🔍 Checking for documentation older than $DAYS_THRESHOLD days..."
echo ""

STALE_COUNT=0

find . -name "*.md" -type f \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  -not -path "*/vendor/*" \
  | while IFS= read -r file; do
    # Get last modification time
    MTIME=$(stat -c %Y "$file" 2>/dev/null || stat -f %m "$file" 2>/dev/null)
    NOW=$(date +%s)
    AGE_DAYS=$(( (NOW - MTIME) / 86400 ))
    
    if [ $AGE_DAYS -gt $DAYS_THRESHOLD ]; then
      echo "⚠️  $file"
      echo "    Age: $AGE_DAYS days"
      echo "    Last modified: $(date -r "$file" '+%Y-%m-%d')"
      echo ""
      ((STALE_COUNT++))
    fi
  done

echo "=================================="
echo "Found $STALE_COUNT stale files"

# Notify via Teams if requested
if [ "$NOTIFY" == "--notify" ] && [ -n "$TEAMS_WEBHOOK_URL" ] && [ $STALE_COUNT -gt 0 ]; then
  curl -H "Content-Type: application/json" -d "{
    \"@type\": \"MessageCard\",
    \"@context\": \"https://schema.org/extensions\",
    \"summary\": \"Stale Documentation Alert\",
    \"themeColor\": \"ffc107\",
    \"title\": \"⚠️ Stale Documentation Found\",
    \"sections\": [{
      \"facts\": [
        {\"name\": \"Files older than\", \"value\": \"$DAYS_THRESHOLD days\"},
        {\"name\": \"Count\", \"value\": \"$STALE_COUNT\"}
      ]
    }]
  }" "$TEAMS_WEBHOOK_URL"
  
  echo "📢 Notification sent to Teams"
fi
```

**Uso:**

```bash
# Check docs older than 90 days (default)
./scripts/check-staleness.sh

# Check docs older than 180 days
./scripts/check-staleness.sh 180

# Check and notify via Teams
./scripts/check-staleness.sh 90 --notify
```

---

## 5. migrate-docs.sh

**Propósito:** Migrar documentación existente al nuevo formato

```bash
#!/bin/bash
# migrate-docs.sh - Migrate existing docs to new structure
# Usage: ./scripts/migrate-docs.sh <source-dir>

SOURCE_DIR="${1:-.}"

echo "📦 Migrating documentation from: $SOURCE_DIR"
echo ""

# Create structure if doesn't exist
mkdir -p 00-templates
mkdir -p 01-global
mkdir -p 02-infrastructure
mkdir -p 03-services
mkdir -p 04-cross-cutting

# Function to detect doc type and move
migrate_file() {
  local file=$1
  local filename=$(basename "$file")
  local content=$(head -20 "$file")
  
  # Try to guess where it should go
  if [[ $content == *"Architecture"* ]] || [[ $content == *"Design"* ]]; then
    echo "📐 Architecture doc: $file"
    # Prompt for service name or create under services
    
  elif [[ $content == *"Deployment"* ]] || [[ $content == *"Runbook"* ]]; then
    echo "🚀 Deployment doc: $file"
    
  elif [[ $content == *"Monitoring"* ]] || [[ $content == *"Observability"* ]]; then
    echo "📊 Monitoring doc: $file"
    
  else
    echo "📄 General doc: $file"
  fi
}

# Find all markdown files
find "$SOURCE_DIR" -name "*.md" -type f | while read file; do
  migrate_file "$file"
done

echo ""
echo "✅ Migration analysis complete"
echo "   Review the suggestions above and move files accordingly"
```

---

## 6. search-docs.sh

**Propósito:** Búsqueda rápida en documentación

```bash
#!/bin/bash
# search-docs.sh - Quick search in documentation
# Usage: ./scripts/search-docs.sh "search term"

if [ $# -eq 0 ]; then
  echo "Usage: $0 \"search term\""
  exit 1
fi

SEARCH_TERM="$1"

echo "🔍 Searching for: '$SEARCH_TERM'"
echo ""

# Search in markdown files
grep -rn --color=always "$SEARCH_TERM" \
  --include="*.md" \
  --exclude-dir=node_modules \
  --exclude-dir=.git \
  --exclude-dir=vendor \
  . | while IFS=: read -r file line content; do
    echo "📄 $file:$line"
    echo "   $content"
    echo ""
  done
```

**Uso:**

```bash
# Buscar término
./scripts/search-docs.sh "kubernetes"

# Buscar frase
./scripts/search-docs.sh "SLO target"
```

---

## 7. backup-to-s3.sh

**Propósito:** Backup del repositorio a S3

```bash
#!/bin/bash
# backup-to-s3.sh - Backup repository to S3
# Requires: aws-cli configured

REPO_DIR=$(git rev-parse --show-toplevel)
BACKUP_NAME="docs-backup-$(date +%Y%m%d-%H%M%S).tar.gz"
S3_BUCKET="${DOC_BACKUP_BUCKET:-company-docs-backups}"

echo "📦 Creating backup..."

# Create tarball
tar -czf "/tmp/$BACKUP_NAME" \
  --exclude=".git" \
  --exclude="node_modules" \
  --exclude="vendor" \
  -C "$REPO_DIR" .

echo "☁️  Uploading to S3..."

# Upload to S3
aws s3 cp "/tmp/$BACKUP_NAME" "s3://$S3_BUCKET/"

# Cleanup
rm "/tmp/$BACKUP_NAME"

echo "✅ Backup complete: s3://$S3_BUCKET/$BACKUP_NAME"
```

**Configurar en cron:**

```bash
# Daily backup a las 3 AM
0 3 * * * /path/to/scripts/backup-to-s3.sh
```

---

## Instalación de Scripts

**Setup rápido:**

```bash
# En el repositorio
mkdir -p scripts

# Copiar todos los scripts
# (copiar contenido de cada script arriba)

# Hacer ejecutables
chmod +x scripts/*.sh scripts/*.py

# Añadir al PATH (opcional)
echo 'export PATH="$PATH:$(git rev-parse --show-toplevel)/scripts"' >> ~/.bashrc

# Test
./scripts/validate-docs.sh
./scripts/generate-metrics.py
```

---

## Integración con CI/CD

**Añadir a `.gitlab-ci.yml`:**

```yaml
# Job para métricas semanales
metrics:weekly:
  stage: report
  script:
    - python3 scripts/generate-metrics.py html
    - python3 scripts/generate-metrics.py json
  artifacts:
    paths:
      - metrics-dashboard.html
      - metrics.json
    expire_in: 30 days
  only:
    - schedules  # Run on schedule (weekly)

# Job para detectar docs obsoletos
check:staleness:
  stage: validate
  script:
    - bash scripts/check-staleness.sh 90 --notify
  only:
    - schedules  # Run on schedule (monthly)
  allow_failure: true
```

---

**Versión:** 1.0  
**Última actualización:** 15-02-2026
