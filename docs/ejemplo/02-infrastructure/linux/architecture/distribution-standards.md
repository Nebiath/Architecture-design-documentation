# Estándares de Distribución Linux — Ubuntu 24.04 LTS

> **Propietario:** @linux-team | **Ubuntu Server 24.04 LTS** como estándar corporativo

## Decisión: Ubuntu 24.04 LTS (Noble Numbat)
- **Release:** Abril 2024 | **Soporte:** Hasta Abril 2029 | **ESM:** Hasta Abril 2034
- **Kernel:** 6.8.x | **Python:** 3.12 | **systemd:** 255.x

## Versiones Permitidas
| Versión | Estado | Uso | EOL |
|---------|--------|-----|-----|
| 24.04 LTS | ✅ Estándar | Nuevos despliegues | 2029 |
| 22.04 LTS | ⚠️ Fase-out | Solo existente | 2027 |
| 20.04 LTS | 🔴 Migrar | ASAP | 2025 |

## Política de Actualización
- **Point releases** (24.04.x): Aplicar mensualmente vía `apt upgrade`
- **LTS upgrades** (22.04→24.04): Solo tras 6 meses del primer point release
- **Ventana:** Sábados 02:00-06:00 UTC

## Kernel
- **GA (6.8.x):** Por defecto, máxima estabilidad
- **HWE:** Solo si hardware muy reciente lo requiere

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
