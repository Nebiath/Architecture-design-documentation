# Architecture Documentation

> **Single Source of Truth** para toda la documentación de arquitectura IT

## 🗺️ Navegación Rápida

### Por Departamento
- [Infraestructura/Hardware](./02-infrastructure/hardware/)
- [Virtualización](./02-infrastructure/virtualization/)
- [WAN](./02-infrastructure/wan/)
- [LAN](./02-infrastructure/lan/)
- [Windows](./02-infrastructure/windows/)
- [Linux](./02-infrastructure/linux/)
- [DevOps/Kubernetes](./02-infrastructure/devops/)
- [Cloud Azure](./02-infrastructure/cloud-azure/)

### Por Tipo de Documento
- [📋 Templates](./00-templates/)
- [🌍 Global/Company-Wide](./01-global/)
- [🚀 Servicios](./03-services/)
- [🔄 Cross-Cutting](./04-cross-cutting/)
- [📝 Architecture Decisions](./06-decisions/)

### Recursos
- [🆕 Cómo contribuir](./CONTRIBUTING.md)
- [🔍 Buscar documentación](#búsqueda)
- [📊 Dashboard de cobertura](https://dashboard.company.com/docs)
- [💬 Slack: #documentation](https://company.slack.com/archives/documentation)

## 🎯 Para Nuevos Usuarios

### Quiero documentar un nuevo servicio
1. Copia el template: `cp -r 03-services/_service-template 03-services/my-service`
2. Sigue la [guía de servicio](./00-templates/service-guide.md)
3. Crea un PR con etiqueta `docs/new-service`

### Quiero actualizar infraestructura departamental
1. Navega a tu departamento en `02-infrastructure/`
2. Edita el documento relevante
3. Crea un PR (auto-asignado a tu equipo vía CODEOWNERS)

### Quiero proponer un cambio arquitectónico
1. Crea un ADR: `cp 06-decisions/template.md 06-decisions/XXX-my-decision.md`
2. Completa el ADR siguiendo el template
3. PR para revisión por @architecture-team

## 📖 Índice de Servicios

| Servicio | Owner | Tier | Última Actualización |
|----------|-------|------|---------------------|
| [Authentication Service](./03-services/authentication-service/) | @auth-team | 1 | 2024-03-10 |
| [Payment Gateway](./03-services/payment-gateway/) | @payments-team | 1 | 2024-03-12 |
| [User Management](./03-services/user-management/) | @users-team | 2 | 2024-03-01 |

[Ver catálogo completo →](./03-services/README.md)

## 🔍 Búsqueda

**Portal de búsqueda:** https://docs.company.com/search

**CLI:**
```bash
# Buscar en todos los docs
grep -r "kubernetes" ./

# Buscar solo en servicios
grep -r "load balancer" ./03-services/

# Buscar departamento específico
grep -r "vlan" ./02-infrastructure/lan/
