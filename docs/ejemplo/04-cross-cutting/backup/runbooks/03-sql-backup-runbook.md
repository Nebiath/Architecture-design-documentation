# Runbook 03: Backup de SQL Server

> **Tiempo:** 45 min | **Propietario:** BBDD Team

## Configuración SQL

```sql
-- 1. Verificar recovery model
SELECT name, recovery_model_desc FROM sys.databases;
-- Platino/Oro: FULL
-- Plata: SIMPLE

-- 2. Habilitar compression
EXEC sp_configure 'backup compression default', 1;
RECONFIGURE;

-- 3. Verificar permisos backup user
USE master;
CREATE LOGIN [DOMAIN\BackupSQL] FROM WINDOWS;
ALTER SERVER ROLE sysadmin ADD MEMBER [DOMAIN\BackupSQL];
-- O menos privilegiado:
-- GRANT BACKUP DATABASE TO [DOMAIN\BackupSQL];
```

## Backup Jobs

**Full (semanal):**
```
Schedule: Sunday 01:00
Databases: ALL user databases
Options:
  - Compression: YES
  - Checksum: YES
  - Copy-only: NO (trunca logs)
Target: /mnt/backup/SQL-Full/
```

**Transaction Log (cada 4h para Platino):**
```
Schedule: 00:00, 04:00, 08:00, 12:00, 16:00, 20:00
Databases: Solo FULL recovery model
Target: /mnt/backup/SQL-Logs/
```

## Restore

**Point-in-Time Recovery:**
```sql
-- 1. Restore último full
RESTORE DATABASE OrderDB
FROM DISK = 'C:\Backups\OrderDB-Full-20250219.bak'
WITH NORECOVERY;

-- 2. Restore último differential
RESTORE DATABASE OrderDB
FROM DISK = 'C:\Backups\OrderDB-Diff-20250220.bak'
WITH NORECOVERY;

-- 3. Restore transaction logs hasta punto deseado
RESTORE LOG OrderDB
FROM DISK = 'C:\Backups\OrderDB-Log-202502201400.trn'
WITH STOPAT = '2025-02-20 14:30:00', RECOVERY;
```

## Monitoring

```sql
-- Jobs que están corriendo
SELECT * FROM msdb.dbo.backupset
WHERE backup_finish_date IS NULL;

-- Último backup por database
SELECT 
    database_name,
    MAX(backup_finish_date) AS last_backup,
    type AS backup_type
FROM msdb.dbo.backupset
GROUP BY database_name, type
ORDER BY database_name;
```
