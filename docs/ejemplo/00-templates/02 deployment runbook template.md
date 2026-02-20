# [Nombre del Servicio] - Deployment & Runbooks

> **Version:** 1.0  
> **Fecha:** YYYY-MM-DD  
> **Service Owner:** [Nombre]  
> **Última Actualización:** YYYY-MM-DD

---

## 🎯 Overview

Este documento describe cómo desplegar, actualizar y operar **[Nombre del Servicio]** en producción.

**Principios:**
- ✅ Infrastructure as Code (IaC) es la fuente de verdad
- ✅ Deployments automatizados y repetibles
- ✅ Rollback rápido en caso de problemas
- ✅ Zero-downtime deployments

---

## 1. Prerequisites

### 1.1 Accesos Requeridos

| Recurso | Acceso Necesario | Cómo Solicitarlo |
|---------|------------------|------------------|
| AWS Account (Production) | PowerUser role | Ticket a infra-team@ |
| GitHub Repository | Write access | Solicitar a repo owner |
| Kubernetes Cluster | Namespace admin | kubectl access request |
| Secrets Manager | Read access | IAM policy request |
| PagerDuty | Service responder | PD admin |
| Datadog | Viewer/Editor | Datadog admin |

### 1.2 Herramientas Requeridas

```bash
# Versiones mínimas requeridas
terraform >= 1.5.0
kubectl >= 1.27
aws-cli >= 2.13
docker >= 24.0
helm >= 3.12
```

**Instalación:**
```bash
# macOS
brew install terraform kubectl awscli docker helm

# Linux
# Ver scripts/setup-tools.sh en el repositorio
```

### 1.3 Configuración Local

```bash
# 1. Clonar repositorio
git clone git@github.com:company/service-name.git
cd service-name

# 2. Configurar credenciales AWS
aws configure --profile production
aws sts get-caller-identity --profile production  # Verificar

# 3. Configurar kubectl
aws eks update-kubeconfig --name prod-cluster --region eu-west-1 --profile production

# 4. Inicializar Terraform
cd terraform/environments/production
terraform init
```

---

## 2. Arquitectura de Deployment

### 2.1 Environments

| Environment | AWS Account | Region | Purpose | Approval Required |
|-------------|-------------|--------|---------|-------------------|
| Development | dev-account | eu-west-1 | Testing | No |
| Staging | staging-account | eu-west-1 | Pre-prod validation | Tech Lead |
| Production | prod-account | eu-west-1, us-east-1 | Live traffic | Change Advisory Board |

### 2.2 Deployment Pipeline

```
┌──────────┐     ┌──────────┐     ┌───────────┐     ┌────────────┐
│   Git    │────▶│   CI     │────▶│  Staging  │────▶│ Production │
│  Commit  │     │ (Build)  │     │  Deploy   │     │   Deploy   │
└──────────┘     └──────────┘     └───────────┘     └────────────┘
                      │                  │                  │
                      ▼                  ▼                  ▼
                  Unit Tests        E2E Tests         Smoke Tests
                  Linting          Load Tests         Monitoring
                  Security Scan    Approval Gate      Gradual Rollout
```

### 2.3 Deployment Strategy

**Tipo:** Canary Deployment (Blue-Green como fallback)

**Fases:**
1. **10% traffic** → 10 minutos → Validar métricas
2. **25% traffic** → 15 minutos → Validar métricas
3. **50% traffic** → 20 minutos → Validar métricas
4. **100% traffic** → Monitoreo continuo

**Auto-rollback si:**
- Error rate > 1%
- Latency p99 > 500ms
- Health check failures > 5%

---

## 3. Infrastructure as Code

### 3.1 Estructura del Repositorio

```
service-name/
├── terraform/
│   ├── modules/                    # Módulos reutilizables
│   │   ├── networking/
│   │   ├── compute/
│   │   └── database/
│   ├── environments/               # Configuraciones por entorno
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       ├── outputs.tf
│   │       └── terraform.tfvars
│   └── global/                     # Recursos globales (IAM, S3)
├── kubernetes/
│   ├── base/                       # Kustomize base
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── overlays/                   # Overlays por entorno
│       ├── dev/
│       ├── staging/
│       └── production/
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── scripts/
│   ├── deploy.sh
│   ├── rollback.sh
│   └── health-check.sh
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
└── docs/
    └── runbooks/
```

### 3.2 Terraform Deployment

**Initial Setup (Solo una vez):**

```bash
cd terraform/environments/production

# 1. Review plan
terraform plan -out=tfplan

# 2. Apply (requiere aprobación de 2 personas)
terraform apply tfplan

# 3. Verify outputs
terraform output
```

**Actualizar Infraestructura:**

```bash
# 1. Pull latest changes
git pull origin main

# 2. Plan changes
terraform plan -out=tfplan

# 3. Review en Pull Request
# 4. Apply después de aprobación
terraform apply tfplan

# 5. Commit state (si no usas remote backend)
git add terraform.tfstate
git commit -m "Update terraform state after applying [change]"
```

**Módulos Terraform Clave:**

```hcl
# terraform/environments/production/main.tf
module "vpc" {
  source = "../../modules/networking"
  
  environment = "production"
  cidr_block  = "10.0.0.0/16"
  azs         = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
}

module "eks" {
  source = "../../modules/compute"
  
  cluster_name    = "prod-cluster"
  cluster_version = "1.27"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnet_ids
}

module "rds" {
  source = "../../modules/database"
  
  instance_class    = "db.r5.large"
  engine_version    = "14.9"
  multi_az          = true
  backup_retention  = 30
}
```

### 3.3 Kubernetes Deployment

**Manual Deployment (Emergencias):**

```bash
# 1. Build and push Docker image
docker build -t company/service-name:v1.2.3 .
docker push company/service-name:v1.2.3

# 2. Update Kubernetes manifest
cd kubernetes/overlays/production
kustomize edit set image company/service-name:v1.2.3

# 3. Apply changes
kubectl apply -k kubernetes/overlays/production

# 4. Monitor rollout
kubectl rollout status deployment/service-name -n production

# 5. Verify pods
kubectl get pods -n production -l app=service-name
```

**GitOps Deployment (Recomendado):**

```bash
# 1. Update image tag en Git
# kubernetes/overlays/production/kustomization.yaml

images:
- name: company/service-name
  newTag: v1.2.3

# 2. Commit and push
git add kubernetes/overlays/production/kustomization.yaml
git commit -m "Deploy v1.2.3 to production"
git push origin main

# 3. ArgoCD/Flux detecta cambios y despliega automáticamente
# 4. Monitor en ArgoCD UI
```

### 3.4 Configuración (ConfigMaps & Secrets)

**ConfigMaps (datos no sensibles):**

```yaml
# kubernetes/base/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-config
data:
  LOG_LEVEL: "info"
  API_TIMEOUT: "30s"
  FEATURE_FLAG_X: "true"
```

**Secrets (datos sensibles):**

```bash
# Usar External Secrets Operator o Sealed Secrets

# 1. Crear secret en Vault/AWS Secrets Manager
aws secretsmanager create-secret \
  --name production/service-name/db-password \
  --secret-string "super-secure-password"

# 2. Referenciar en ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: service-db-credentials
spec:
  secretStoreRef:
    name: aws-secrets-manager
  target:
    name: service-db-secret
  data:
  - secretKey: password
    remoteRef:
      key: production/service-name/db-password
```

---

## 4. Deployment Procedures

### 4.1 Standard Deployment (CI/CD)

**Trigger:** Push a `main` branch

**Proceso Automático:**

```yaml
# .github/workflows/cd.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Build Docker Image
        run: docker build -t company/service-name:${{ github.sha }} .
      
      - name: Push to Registry
        run: docker push company/service-name:${{ github.sha }}
      
      - name: Deploy to Staging
        run: ./scripts/deploy.sh staging ${{ github.sha }}
      
      - name: Run E2E Tests
        run: npm run test:e2e
      
      - name: Await Manual Approval
        uses: trstringer/manual-approval@v1
        with:
          approvers: tech-leads,sre-team
      
      - name: Deploy to Production (Canary)
        run: ./scripts/deploy.sh production ${{ github.sha }} --canary
      
      - name: Monitor Deployment
        run: ./scripts/monitor-deployment.sh 30m
      
      - name: Promote to 100%
        if: success()
        run: ./scripts/promote-canary.sh
```

### 4.2 Hotfix Deployment

**Escenario:** Bug crítico en producción que necesita fix urgente

**Proceso:**

```bash
# 1. Crear hotfix branch desde producción
git checkout -b hotfix/critical-bug-fix production

# 2. Hacer el fix
vim src/critical-file.js
git add src/critical-file.js
git commit -m "Fix: [descripción del bug]"

# 3. Build y test localmente
npm test
docker build -t company/service-name:hotfix-$(git rev-parse --short HEAD) .

# 4. Push hotfix branch
git push origin hotfix/critical-bug-fix

# 5. Crear Pull Request con label "hotfix"
# 6. Fast-track review (1 aprobador en lugar de 2)
# 7. Merge → Deploy automático
# 8. Merge back a main
git checkout main
git merge hotfix/critical-bug-fix
git push origin main
```

### 4.3 Database Migrations

**Principios:**
- ✅ Migrations siempre son backwards compatible
- ✅ Deploy código primero, luego migration
- ✅ Nunca borrar columnas en la misma release que dejas de usarlas

**Proceso:**

```bash
# 1. Crear migration
npm run migration:create add-user-email-index

# 2. Editar migration file
# migrations/20240315120000-add-user-email-index.js

exports.up = async (db) => {
  await db.collection('users').createIndex({ email: 1 }, { unique: true });
};

exports.down = async (db) => {
  await db.collection('users').dropIndex('email_1');
};

# 3. Test locally
npm run migration:up

# 4. Commit migration file
git add migrations/
git commit -m "Migration: Add email index to users"

# 5. Deploy código (migration NO se ejecuta aún)
./scripts/deploy.sh production v1.2.3

# 6. Ejecutar migration en producción (manual)
kubectl exec -it deployment/service-name -- npm run migration:up

# 7. Verify
kubectl logs deployment/service-name | grep "Migration completed"
```

**Migration Rollback:**

```bash
# Si algo falla
kubectl exec -it deployment/service-name -- npm run migration:down

# Rollback application
./scripts/rollback.sh
```

---

## 5. Rollback Procedures

### 5.1 Application Rollback

**Opción 1: Kubernetes Rollback (Rápido)**

```bash
# Rollback to previous version
kubectl rollout undo deployment/service-name -n production

# Rollback to specific revision
kubectl rollout history deployment/service-name -n production
kubectl rollout undo deployment/service-name -n production --to-revision=5

# Monitor rollback
kubectl rollout status deployment/service-name -n production
```

**Opción 2: GitOps Rollback**

```bash
# 1. Revert commit en Git
git revert HEAD
git push origin main

# 2. ArgoCD detecta y despliega versión anterior
# Monitor en ArgoCD UI
```

**Opción 3: Script de Rollback**

```bash
# ./scripts/rollback.sh
#!/bin/bash
PREVIOUS_VERSION=$(kubectl get deployment service-name -n production -o jsonpath='{.metadata.annotations.previous-version}')

echo "Rolling back to version: $PREVIOUS_VERSION"

kubectl set image deployment/service-name \
  service-name=company/service-name:$PREVIOUS_VERSION \
  -n production

kubectl rollout status deployment/service-name -n production

# Run smoke tests
./scripts/health-check.sh

if [ $? -eq 0 ]; then
  echo "✅ Rollback successful"
else
  echo "❌ Rollback failed - manual intervention required"
  exit 1
fi
```

### 5.2 Infrastructure Rollback

```bash
# Terraform rollback
cd terraform/environments/production

# Opción 1: Revert Git commit
git revert HEAD
terraform plan
terraform apply

# Opción 2: Specific resource rollback
terraform import aws_instance.app <instance-id>
terraform apply -target=aws_instance.app
```

### 5.3 Database Rollback

**⚠️ CUIDADO:** Rollback de DB puede causar pérdida de datos

```bash
# 1. Stop application traffic
kubectl scale deployment/service-name --replicas=0 -n production

# 2. Restore from backup
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier prod-db \
  --target-db-instance-identifier prod-db-restored \
  --restore-time 2024-03-15T10:00:00Z

# 3. Update DNS/connection string
# 4. Restart application
kubectl scale deployment/service-name --replicas=3 -n production

# 5. Verify data integrity
./scripts/verify-db.sh
```

---

## 6. Operational Runbooks

### 6.1 Health Checks

**Manual Health Check:**

```bash
#!/bin/bash
# scripts/health-check.sh

HEALTH_ENDPOINT="https://api.company.com/health"
MAX_RETRIES=3

for i in $(seq 1 $MAX_RETRIES); do
  HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" $HEALTH_ENDPOINT)
  
  if [ "$HTTP_CODE" = "200" ]; then
    echo "✅ Health check passed"
    exit 0
  fi
  
  echo "⚠️ Attempt $i failed (HTTP $HTTP_CODE). Retrying..."
  sleep 5
done

echo "❌ Health check failed after $MAX_RETRIES attempts"
exit 1
```

**Automated Health Checks (Kubernetes):**

```yaml
# kubernetes/base/deployment.yaml
spec:
  containers:
  - name: service-name
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 2
```

### 6.2 Scaling Operations

**Manual Scaling:**

```bash
# Scale up
kubectl scale deployment/service-name --replicas=10 -n production

# Scale down
kubectl scale deployment/service-name --replicas=3 -n production

# Verify
kubectl get pods -n production -l app=service-name
```

**Horizontal Pod Autoscaler (HPA):**

```yaml
# kubernetes/base/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: service-name-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: service-name
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
```

### 6.3 Certificate Rotation

```bash
# TLS certificates (cert-manager automático)
kubectl get certificate -n production
kubectl describe certificate service-name-tls -n production

# Manual renewal si falla cert-manager
kubectl delete secret service-name-tls -n production
# cert-manager recreará automáticamente

# Secrets rotation (AWS Secrets Manager)
aws secretsmanager rotate-secret \
  --secret-id production/service-name/db-password \
  --rotation-lambda-arn arn:aws:lambda:eu-west-1:123456789:function:SecretsManagerRotation

# Restart pods to pick up new secret
kubectl rollout restart deployment/service-name -n production
```

### 6.4 Log Access

```bash
# Tail logs de todos los pods
kubectl logs -f deployment/service-name -n production

# Logs de un pod específico
kubectl logs -f service-name-7d8b9c5f-xk2m4 -n production

# Logs anteriores (si pod crasheó)
kubectl logs --previous service-name-7d8b9c5f-xk2m4 -n production

# Buscar errors
kubectl logs deployment/service-name -n production | grep ERROR

# Logs en Splunk/ELK
# https://splunk.company.com/search?q=index=production service=service-name
```

### 6.5 Troubleshooting Common Issues

#### Issue: Pods en CrashLoopBackOff

```bash
# 1. Check pod status
kubectl describe pod <pod-name> -n production

# 2. Check logs
kubectl logs <pod-name> -n production --previous

# 3. Common causes:
# - Fallo en health checks → Check /health endpoint
# - OOMKilled → Increase memory limits
# - ConfigMap/Secret missing → Verify existence
# - Image pull errors → Check registry auth

# 4. Exec into pod (si está running)
kubectl exec -it <pod-name> -n production -- /bin/bash
```

#### Issue: High Latency

```bash
# 1. Check resource utilization
kubectl top pods -n production -l app=service-name

# 2. Check database connections
kubectl exec -it deployment/service-name -n production -- \
  psql -U user -d database -c "SELECT count(*) FROM pg_stat_activity;"

# 3. Check external dependencies
curl -w "@curl-format.txt" -o /dev/null -s https://external-api.com/

# 4. Scale up if needed
kubectl scale deployment/service-name --replicas=10 -n production
```

#### Issue: Deployment Stuck

```bash
# 1. Check deployment status
kubectl rollout status deployment/service-name -n production

# 2. Describe deployment
kubectl describe deployment service-name -n production

# 3. Common issues:
# - Image pull error → Check image exists
# - Resource quota exceeded → Check namespace limits
# - PVC issues → Check storage provisioning

# 4. Force rollback if stuck
kubectl rollout undo deployment/service-name -n production
```

---

## 7. Maintenance Windows

### 7.1 Scheduled Maintenance

**Ventanas permitidas:**
- **Día:** Domingo
- **Hora:** 02:00 - 06:00 UTC
- **Frecuencia:** Mensual (primer domingo del mes)
- **Duración máxima:** 4 horas

**Procedimiento:**

```bash
# 1. Notificar 72h antes (automated via PagerDuty)
# Template: "Scheduled maintenance on [date] [time]"

# 2. Pre-maintenance checklist
- [ ] Backup database
- [ ] Snapshot VMs
- [ ] Export current configs
- [ ] Test rollback procedure
- [ ] Notify stakeholders

# 3. Durante maintenance
./scripts/maintenance.sh start
# ... realizar cambios ...
./scripts/health-check.sh
./scripts/maintenance.sh end

# 4. Post-maintenance
- [ ] Verify all services healthy
- [ ] Check monitoring dashboards
- [ ] Run smoke tests
- [ ] Notify stakeholders (maintenance complete)
```

### 7.2 Emergency Maintenance

**Trigger:** P1 incident que requiere cambios inmediatos

**No requiere pre-notificación, pero:**

```bash
# 1. Notificar en Slack #incidents channel
"🚨 Emergency maintenance in progress - [descripción]"

# 2. Create incident in PagerDuty
./scripts/create-incident.sh "Emergency maintenance: [reason]"

# 3. Realizar cambios siguiendo change log
# 4. Post-mortem después del incidente
```

---

## 8. Disaster Recovery Drills

**Frecuencia:** Trimestral

**Escenarios a probar:**

### Drill 1: Complete Region Failure

```bash
# 1. Simulate region failure
aws ec2 stop-instances --instance-ids $(aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=production" \
  --region eu-west-1 --query 'Reservations[].Instances[].InstanceId' --output text)

# 2. Failover to DR region (us-east-1)
./scripts/dr-failover.sh us-east-1

# 3. Update DNS
aws route53 change-resource-record-sets --hosted-zone-id Z123456 \
  --change-batch file://dns-failover.json

# 4. Verify services in DR region
./scripts/health-check.sh us-east-1

# 5. RTO achieved? Document actual time
```

### Drill 2: Database Corruption

```bash
# 1. Simulate corruption (in staging!)
# 2. Restore from backup
./scripts/restore-db.sh --backup-id 2024-03-15-02:00
# 3. Verify data integrity
# 4. RPO achieved? Document data loss window
```

### Drill 3: Full Application Rollback

```bash
# 1. Deploy intentionally broken version (in staging)
# 2. Trigger rollback
./scripts/rollback.sh
# 3. Measure rollback time
# 4. Verify functionality
```

---

## 9. Automation Scripts

### 9.1 Deploy Script

```bash
#!/bin/bash
# scripts/deploy.sh

set -euo pipefail

ENVIRONMENT=$1
VERSION=$2
CANARY=${3:-false}

echo "🚀 Deploying $VERSION to $ENVIRONMENT"

# Pre-deployment checks
./scripts/pre-flight-check.sh $ENVIRONMENT

# Deploy based on environment
case $ENVIRONMENT in
  staging)
    kubectl apply -k kubernetes/overlays/staging
    ;;
  production)
    if [ "$CANARY" = "--canary" ]; then
      kubectl apply -f kubernetes/overlays/production/canary.yaml
    else
      kubectl apply -k kubernetes/overlays/production
    fi
    ;;
  *)
    echo "Unknown environment: $ENVIRONMENT"
    exit 1
    ;;
esac

# Wait for rollout
kubectl rollout status deployment/service-name -n $ENVIRONMENT --timeout=10m

# Post-deployment checks
./scripts/smoke-test.sh $ENVIRONMENT

echo "✅ Deployment successful"
```

### 9.2 Monitoring Script

```bash
#!/bin/bash
# scripts/monitor-deployment.sh

DURATION=$1  # e.g., "30m"
END_TIME=$(($(date +%s) + $(echo $DURATION | sed 's/m/*60/' | bc)))

echo "📊 Monitoring deployment for $DURATION"

while [ $(date +%s) -lt $END_TIME ]; do
  ERROR_RATE=$(curl -s "https://api.datadog.com/api/v1/query?query=avg:service.errors.rate{service:service-name}" | jq '.series[0].pointlist[-1][1]')
  
  if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
    echo "❌ Error rate too high: $ERROR_RATE"
    ./scripts/rollback.sh
    exit 1
  fi
  
  echo "✅ Metrics healthy (error rate: $ERROR_RATE)"
  sleep 60
done

echo "✅ Monitoring period complete - deployment stable"
```

---

## 10. Checklists

### Pre-Deployment Checklist

- [ ] Código reviewed (2+ approvers)
- [ ] Tests passing (unit, integration, E2E)
- [ ] Security scan passed
- [ ] Changelog actualizado
- [ ] Terraform plan reviewed (si hay cambios infra)
- [ ] Database migrations tested
- [ ] Rollback plan documentado
- [ ] Stakeholders notificados (si breaking changes)
- [ ] Monitoring dashboards actualizados
- [ ] Runbooks actualizados si nuevos features

### Post-Deployment Checklist

- [ ] Health checks passing
- [ ] No alerts en Datadog/PagerDuty
- [ ] Error rate < 0.1%
- [ ] Latency dentro de SLO
- [ ] Logs no muestran errors inesperados
- [ ] Smoke tests passed
- [ ] Key user journeys funcionando
- [ ] Database connections healthy
- [ ] External integrations working

---

## 11. Contacts & Escalation

| Issue Type | Contact | Response Time |
|------------|---------|---------------|
| Deployment bloqueado | SRE on-call | 15 min |
| Infrastructure issues | Cloud team | 30 min |
| Database issues | DBA team | 15 min |
| Security concerns | Security team | Inmediato |
| Application bugs | Development team | 1 hora |

**PagerDuty Escalation Policy:**
1. On-call engineer (5 min)
2. Team lead (15 min)
3. Engineering manager (30 min)
4. VP Engineering (1 hour)

---

## 12. Referencias

- [Architecture Design Doc](./01-architecture-design.md)
- [Observability & Monitoring](./04-observability.md)
- [Incident Runbooks](./06-incident-management.md)
- [Terraform Documentation](https://terraform.io/docs)
- [Kubernetes Documentation](https://kubernetes.io/docs)

---

**Última actualización:** Revisar este documento después de cada deployment significativo
