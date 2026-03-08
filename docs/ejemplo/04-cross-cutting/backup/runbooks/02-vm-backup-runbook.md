# Runbook 02: Backup de Virtual Machines

> **Tiempo:** 30 min (configuración) | **Propietario:** Virtualización Team

## Hyper-V

**Configurar backup job:**
```powershell
# Verificar Integration Services
Get-VM | Get-VMIntegrationService -Name "VSS" | Enable-VMIntegrationService

# En backup console:
# 1. New Job > Hyper-V Backup
# 2. Select hosts o VMs individuales
# 3. Schedule: Daily 22:00
# 4. Options:
#    - Application-aware: YES
#    - Quiesce: YES (via VSS)
#    - Compression: Medium
```

**Restore VM:**
```powershell
# Opción 1: Restore a ubicación original (sobreescribe)
# Opción 2: Restore to new location (VM temporal)
# Opción 3: Instant Recovery (VM arranca desde backup)
```

## VMware vSphere

**Prerequisitos:**
- CBT habilitado en VMs
- VMware Tools instalado

**Config backup job:**
```
Source: vCenter (agentless via VADP API)
Options:
  ✅ Application-aware processing
  ✅ Changed Block Tracking
  ✅ Quiesce filesystem
```

**Troubleshooting:**
```bash
# CBT no funciona
vim-cmd vmsvc/snapshot.get <vmid>
# Si hay snapshots viejos: consolidar
# Disable/re-enable CBT
```
