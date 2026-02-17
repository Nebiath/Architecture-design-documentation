## 🔄 Workflow de Contribución

### 1. Crear Nueva Documentación

```bash
# Para un nuevo servicio
git checkout -b docs/devops/new-service-auth
cp -r 03-services/_service-template 03-services/auth-service
# Completar templates
git add 03-services/auth-service/
git commit -m "docs: Add authentication service architecture"
git push origin docs/devops/new-service-auth
# Crear PR en GitHub
