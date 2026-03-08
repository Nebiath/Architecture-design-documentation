# Runbook 05: Procedimientos de Restore

> **Tiempo:** Variable | **Propietario:** IT Ops + Data Owner

## 1. Pre-Restore

**Validar request:**
```
✅ Ticket ServiceNow creado
✅ Justificación documentada
✅ Aprobación del data owner
✅ Para Platino: Aprobación IT Manager + Security (MFA)
✅ Identificar backup point correcto (fecha/hora)
```

## 2. Restore VM

**Opción A: Restore to original location**
```
CUIDADO: Sobreescribe VM actual
1. Power off VM actual (si aún existe)
2. En backup console: Select backup > Restore
3. Options:
   - Target: Original location
   - Power on after restore: NO (manual después de validar)
4. Restore duration: ~30 min por 100 GB
5. Power on y validar con app owner
```

**Opción B: Restore to new location (recomendado)**
```
Más seguro: No afecta producción
1. En backup console: Restore to new VM
2. Asignar nombre temporal: VM-NAME-RESTORE
3. Network: Isolated VLAN (no conflictos IP)
4. Validar datos
5. Si correcto: Swap IPs y promote a producción
```

**Opción C: Instant Recovery**
```
Para RTO urgente (<1h):
1. VM arranca directamente desde backup (read from backup storage)
2. En paralelo: Storage vMotion a production datastore (background)
3. Usuario ya puede trabajar mientras migra
```

## 3. Restore SQL Database

**Full database restore:**
```sql
-- Simple (solo último full)
RESTORE DATABASE OrderDB
FROM DISK = 'C:\Backups\OrderDB-Full-20250219.bak'
WITH REPLACE, RECOVERY;

-- Con point-in-time (ver runbook 03)
```

**Individual table restore:**
```sql
-- Opción 1: Restore DB completa a nombre temporal
RESTORE DATABASE OrderDB_Temp FROM DISK = '...';

-- Copiar tabla específica
INSERT INTO OrderDB.dbo.Customers
SELECT * FROM OrderDB_Temp.dbo.Customers
WHERE LastModified > '2025-02-15';

-- Drop DB temp
DROP DATABASE OrderDB_Temp;
```

## 4. Restore Files

**File-level restore:**
```
1. En backup console: Browse backup
2. Navegar a carpeta/archivo deseado
3. Restore to:
   - Original location (sobreescribe)
   - Alternate location (C:\Temp\RestoreTemp\)
4. Notify user dónde están los archivos
```

## 5. Post-Restore

**Validación:**
```
✅ Application owner confirma datos correctos
✅ Funcionalidad probada (login, transacciones, etc.)
✅ No hay errores en logs
✅ Performance normal
```

**Documentación:**
```
En ticket ServiceNow:
- Qué se restauró (VM/DB/Files)
- De qué backup (fecha/hora)
- A dónde (location)
- Duración total
- Resultado: Exitoso / Fallido (razón)
- Sign-off del data owner
```

## 6. Troubleshooting

**Restore muy lento:**
```
- Verificar bandwidth: sar -n DEV 1
- Verificar I/O storage: iostat -x 1
- Si disco: mover restore a horas no-pico
- Si red: aumentar prioridad QoS
```

**Restore falla:**
```
- Verificar log error específico
- Errores comunes:
  * Not enough space: Liberar espacio destino
  * File locked: Cerrar aplicaciones usando archivo
  * Permission denied: Verificar permisos user restore
  * Backup corrupted: Intentar backup anterior
```
