# Runbook 06: Troubleshooting Backup

> **Tiempo:** Variable | **Propietario:** IT Ops + Backup Team

## Issue: Backup Job Failed

**Diagnóstico:**
```bash
# 1. Ver logs del job
tail -100 /var/log/backup/job-platino-vm.log

# Errores comunes:
# "Snapshot creation failed" → Hypervisor issue
# "Disk full" → Storage target lleno
# "Authentication failed" → Credenciales incorrectas
# "Timeout" → Network o performance issue
```

**Resolución:**
```
Snapshot failed:
→ Verificar snapshots huérfanos: Consolidar manualmente
→ Verificar espacio datastore: Liberar espacio

Disk full:
→ Verificar capacidad: df -h /mnt/backup
→ Limpiar backups antiguos fuera de retention
→ Escalar para expansión storage

Auth failed:
→ Verificar credenciales: Test login manual
→ Password expirado: Resetear en AD
→ Account locked: Unlock en AD
```

## Issue: Agent Offline

**Diagnóstico:**
```bash
# Windows
Get-Service -Name "BackupAgent" | Select Status
Test-NetConnection -ComputerName backup-server -Port 10000

# Linux
systemctl status backup-agent
telnet backup-server 10000
```

**Resolución:**
```powershell
# Windows: Restart service
Restart-Service -Name "BackupAgent"

# Linux
sudo systemctl restart backup-agent

# Si sigue offline:
# - Verificar firewall: sudo iptables -L
# - Verificar logs: journalctl -u backup-agent -n 50
# - Reinstalar agent si corrupto
```

## Issue: Deduplication Ratio Bajo (<5:1)

**Causas:**
```
1. Datos altamente únicos (videos, fotos, archivos comprimidos)
2. Encriptación pre-backup (datos encriptados no dedup bien)
3. Dedup engine no configurado correctamente
4. Primera ejecución (aún no hay datos para dedup)
```

**Acciones:**
```
- Verificar tipo de datos: file /mnt/backup/*
- Si muchos .zip/.7z/.mp4: Expected (no se puede mejorar)
- Si DBs encriptadas (TDE): Deshabilitar TDE o aceptar ratio bajo
- Revisar config dedup engine: dedup block size (64KB recommended)
```

## Issue: Backup Toma Muy Largo

**Diagnóstico:**
```bash
# Medir throughput actual
# Durante backup job activo:
sar -n DEV 1 10  # Network
iostat -x 1 10   # Disk I/O

# Baseline esperado:
# - LAN 10GbE: >1 GB/s
# - Disk NAS: >500 MB/s
# - Dedup overhead: -30% throughput

# Si < 100 MB/s: problema
```

**Optimización:**
```
Network bottleneck:
→ Upgrade NICs a 10GbE
→ Habilitar jumbo frames (MTU 9000)
→ Network bonding/teaming

Disk bottleneck:
→ Usar SSDs para dedup cache
→ RAID 10 mejor que RAID 5 para writes
→ Increase queue depth: blockdev --setra 16384 /dev/sdb

Concurrency:
→ Reducir jobs simultáneos (4 → 2)
→ Schedule jobs en ventanas más largas
```

## Issue: Restore Falla

**Errores comunes:**
```
"Backup chain broken":
→ Falta incremental intermedio
→ Solution: Restore desde backup full anterior

"Media offline":
→ Tape no disponible en library
→ Solution: Cargar tape manualmente o recall desde off-site

"Insufficient space":
→ Destino no tiene espacio
→ Solution: Liberar espacio o restore a ubicación alternativa

"File locked":
→ Aplicación usando archivo
→ Solution: Stop application/service temporalmente
```

## Escalación

```
Level 1 (IT Ops) - No puede resolver en 1h
   ↓
Level 2 (Backup Specialist) - No puede resolver en 4h
   ↓
Level 3 (Vendor Support) - Abrir ticket
   ↓
Level 4 (Arquitectura IT + Management)
```

**Vendor Support:**
```
Preparar antes de llamar:
- Job logs (últimas 24h)
- Screenshot del error
- Versión software: backup-software --version
- Configuración: export config to file
- Caso de uso: "Backing up VMware VMs via vCenter"
```
