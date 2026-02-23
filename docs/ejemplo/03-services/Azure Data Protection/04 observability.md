# Industrial Image Backup to Azure — Observability & Monitoring

> **Version:** 1.0  
> **Fecha:** 2025-02-19  
> **Estado:** Approved  
> **Autor:** Cloud Architecture  
> **Service Owner:** Cloud Architecture
> 
> **Última Revisión:** 2025-02-19

---

## 1. Monitoring Strategy

**Objetivo:** Detectar proactivamente fallos en el flujo de backup OT → Azure antes de que impacten al negocio.

**Principios:**

- **Proactive, not reactive:** Alertar ANTES de que se pierdan 24h de imágenes
- **End-to-end coverage:** Monitorizar todos los puntos del flujo (OT → FW → IT → Azure)
- **Actionable alerts:** Cada alerta debe tener un runbook asociado
- **Noise reduction:** Max 2 alertas/semana por planta en operación normal

---

## 2. Monitoring Stack

| Layer                     | Tool                   | Metrics Collected                      | Logs Collected                      |
| ------------------------- | ---------------------- | -------------------------------------- | ----------------------------------- |
| **VM Gateway (OS)**       | Azure Monitor Agent    | CPU, RAM, Disk, Network                | Windows Event Logs, PowerShell logs |
| **Application (Scripts)** | Custom PowerShell logs | Files detected, uploaded, failed       | D:\ImageBackup\Logs\*.log           |
| **Azure Blob Storage**    | Azure Monitor          | Blob count, storage size, transactions | Access logs, lifecycle events       |
| **Network**               | Azure Network Watcher  | Bandwidth, latency                     | Flow logs (NSG)                     |
| **Alerting**              | Azure Monitor Alerts   | —                                      | —                                   |
| **Dashboards**            | Azure Workbooks        | —                                      | —                                   |

---

## 3. Golden Signals

Monitorizamos los **4 Golden Signals** de Google SRE:

### 3.1 Latency

**Definition:** Tiempo desde que una imagen llega a `Incoming/` hasta confirmación en Azure

**Metric:**

```
upload_latency_seconds = timestamp_azure_confirmed - timestamp_file_arrived_incoming
```

**Target:**

- p50 < 15 minutes
- p95 < 30 minutes
- p99 < 60 minutes

**Alert:**

- Warning: p95 > 45 min sustained for 2 hours
- Critical: p95 > 90 min

**Data Source:** Custom metric enviada desde `Upload-ToAzure.ps1`

### 3.2 Traffic

**Definition:** Volumen de archivos procesados por día

**Metric:**

```
files_uploaded_per_day = count(successful_uploads) in last 24h
```

**Target:**

- Promedio: 50-200 files/day (varía por planta)
- Baseline establecido después de 30 días de operación

**Alert:**

- Warning: traffic < 50% of baseline for 24h (posible problema en OT)
- Critical: traffic = 0 for 24h (servicio completamente caído)

**Data Source:** Azure Blob Storage metrics (`BlobCount`)

### 3.3 Errors

**Definition:** Tasa de fallos en uploads

**Metric:**

```
error_rate = failed_uploads / total_upload_attempts
```

**Target:**

- < 0.5% (995 de 1000 archivos suben exitosamente)

**Alert:**

- Warning: error_rate > 2% for 1 hour
- Critical: error_rate > 10% for 30 min

**Data Source:** Logs de `Upload-ToAzure.ps1` (parsing "ERROR" lines)

### 3.4 Saturation

**Definition:** Utilización de recursos críticos

**Metrics:**

- `vm_disk_used_percent` = (used space / total space) * 100 on D:\
- `vm_cpu_percent` = avg CPU over 10 min
- `vm_memory_percent` = used memory / total memory

**Targets:**

- Disk < 80%
- CPU < 70%
- Memory < 85%

**Alerts:**

- Warning: disk > 80% for 2 hours
- Critical: disk > 90%

**Data Source:** Azure Monitor (VM metrics)

---

## 4. Metrics Collection

### 4.1 Infrastructure Metrics (Azure Monitor)

**VM Gateway Metrics (built-in):**

```kql
// CPU utilization
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avg_cpu = avg(CounterValue) by Computer
| where avg_cpu > 70

// Disk space
Perf
| where TimeGenerated > ago(10m)
| where ObjectName == "LogicalDisk" and CounterName == "% Free Space" and InstanceName == "D:"
| summarize free_percent = avg(CounterValue) by Computer
| where free_percent < 20
```

### 4.2 Application Metrics (Custom)

**Script enhancement:** Modificar `Upload-ToAzure.ps1` para enviar custom metrics

```powershell
# Al final del script Upload-ToAzure.ps1, añadir:

# Enviar métrica custom a Azure Monitor
$customMetrics = @{
    ResourceId = "/subscriptions/{sub-id}/resourceGroups/RG-IndustrialBackup-WestEurope/providers/Microsoft.Compute/virtualMachines/ITGW-FACTORY-EU01"
    MetricName = "FilesUploadedCount"
    MetricValue = $files.Count
    Timestamp = (Get-Date).ToUniversalTime()
}

# Usar Azure Monitor REST API (con Managed Identity auth)
$token = (Get-AzAccessToken -ResourceUrl "https://monitoring.azure.com").Token
$uri = "https://monitoring.azure.com$($customMetrics.ResourceId)/metrics"
$headers = @{
    "Authorization" = "Bearer $token"
    "Content-Type" = "application/json"
}
$body = @{
    time = $customMetrics.Timestamp
    data = @{
        baseData = @{
            metric = $customMetrics.MetricName
            namespace = "IndustrialBackup"
            dimNames = @()
            series = @(
                @{
                    dimValues = @()
                    min = $customMetrics.MetricValue
                    max = $customMetrics.MetricValue
                    sum = $customMetrics.MetricValue
                    count = 1
                }
            )
        }
    }
} | ConvertTo-Json -Depth 10

Invoke-RestMethod -Uri $uri -Method POST -Headers $headers -Body $body
```

### 4.3 Storage Metrics (Azure Monitor)

```kql
// Blobs uploaded today
StorageBlobLogs
| where TimeGenerated > startofday(now())
| where OperationName == "PutBlob" and StatusText == "Success"
| summarize BlobsUploaded = count() by bin(TimeGenerated, 1h)

// Storage size growth
AzureMetrics
| where ResourceId contains "industrialbackupweu"
| where MetricName == "BlobCapacity"
| summarize UsedCapacity_GB = max(Total) / 1024 / 1024 / 1024 by bin(TimeGenerated, 1d)
```

---

## 5. Logging

### 5.1 Log Sources

| Source                 | Log Path                  | Format                           | Retention     | Shipped to    |
| ---------------------- | ------------------------- | -------------------------------- | ------------- | ------------- |
| **PowerShell Scripts** | D:\ImageBackup\Logs\*.log | Plain text (timestamp + message) | 30 días local | Log Analytics |
| **Windows Event Log**  | Application, System       | Windows Event                    | 30 días local | Log Analytics |
| **Azure Storage**      | Diagnostic logs           | JSON                             | 90 días       | Log Analytics |
| **Task Scheduler**     | Task Scheduler logs       | Windows Event                    | 30 días local | Log Analytics |

### 5.2 Log Shipping Configuration

```powershell
# Configurar Azure Monitor Agent para enviar logs a Log Analytics

# 1. Instalar Azure Monitor Agent en VM
$workspaceId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  # Log Analytics Workspace ID
$workspaceKey = Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName "RG-IndustrialBackup-WestEurope" -Name "law-industrialbackup-weu"

# 2. Configurar data collection rule (DCR)
# Azure Portal → Monitor → Data Collection Rules → Create
# - Name: DCR-IndustrialBackup-Gateway
# - Data source: Custom Text Logs → D:\ImageBackup\Logs\*.log
# - Destination: Log Analytics Workspace → law-industrialbackup-weu
# - Table: IndustrialBackup_CL (custom log table)

# 3. Asociar DCR con VM Gateway
# Azure Portal → VM → Monitoring → Logs → Configure → Select DCR
```

### 5.3 Log Analysis Queries (KQL)

**Query 1: Errores en últimas 24 horas**

```kql
IndustrialBackup_CL
| where TimeGenerated > ago(24h)
| where RawData contains "ERROR"
| project TimeGenerated, Computer, RawData
| order by TimeGenerated desc
```

**Query 2: Archivos subidos por hora (últimos 7 días)**

```kql
IndustrialBackup_CL
| where TimeGenerated > ago(7d)
| where RawData contains "Uploaded and moved"
| extend FileName = extract(@"Uploaded and moved ([^ ]+)", 1, RawData)
| summarize FilesUploaded = count() by bin(TimeGenerated, 1h)
| render timechart
```

**Query 3: Upload failures por planta**

```kql
IndustrialBackup_CL
| where TimeGenerated > ago(7d)
| where RawData contains "AzCopy failed"
| summarize FailureCount = count() by Computer
| order by FailureCount desc
```

---

## 6. Dashboards

### 6.1 Main Dashboard: Industrial Backup Overview

**Dashboard URL:** https://portal.azure.com/#view/AppInsightsExtension/DashboardV2/xxx

**Widgets:**

1. **Upload Success Rate (Last 7d)** — Line chart
   
   ```kql
   IndustrialBackup_CL
   | where TimeGenerated > ago(7d)
   | extend UploadResult = iff(RawData contains "Upload complete", "Success", "Failed")
   | summarize SuccessRate = 100.0 * countif(UploadResult == "Success") / count() by bin(TimeGenerated, 6h)
   | render timechart
   ```

2. **Files Uploaded per Plant (Today)** — Bar chart
   
   ```kql
   IndustrialBackup_CL
   | where TimeGenerated > startofday(now())
   | where RawData contains "Uploaded and moved"
   | summarize FilesUploaded = count() by Computer
   | render barchart
   ```

3. **VM Disk Usage** — Gauge
   
   ```kql
   Perf
   | where TimeGenerated > ago(10m)
   | where ObjectName == "LogicalDisk" and CounterName == "% Free Space" and InstanceName == "D:"
   | summarize DiskUsedPercent = 100 - avg(CounterValue) by Computer
   | render table
   ```

4. **Error Log Count (Last 24h)** — Single stat
   
   ```kql
   IndustrialBackup_CL
   | where TimeGenerated > ago(24h)
   | where RawData contains "ERROR"
   | summarize ErrorCount = count()
   ```

5. **Azure Blob Storage Size (Trend)** — Area chart
   
   ```kql
   AzureMetrics
   | where ResourceId contains "industrialbackupweu"
   | where MetricName == "BlobCapacity"
   | summarize StorageGB = max(Total) / 1024 / 1024 / 1024 by bin(TimeGenerated, 1d)
   | render areachart
   ```

### 6.2 Per-Plant Dashboard

Clonar dashboard principal y filtrar por `Computer == "ITGW-FACTORY-EU01"` para vista específica de cada planta.

---

## 7. Alerts

### 7.1 Alert Rules

| Alert Name              | Condition                       | Severity | Frequency   | Action Group      |
| ----------------------- | ------------------------------- | -------- | ----------- | ----------------- |
| **No uploads in 24h**   | BlobCount increment = 0 for 24h | Critical | Every 6h    | Email + PagerDuty |
| **High error rate**     | Error rate > 5% for 1h          | Warning  | Every 15min | Email IT Ops      |
| **VM disk almost full** | Disk D:\ > 85%                  | Warning  | Every 1h    | Email IT Ops      |
| **VM disk critical**    | Disk D:\ > 95%                  | Critical | Every 15min | Email + PagerDuty |
| **VM down**             | VM heartbeat missing for 10min  | Critical | Every 5min  | PagerDuty         |
| **Upload latency high** | p95 > 60min for 2h              | Warning  | Every 30min | Email Cloud Team  |

### 7.2 Alert Configuration (Example)

```powershell
# Create Alert Rule: No uploads in 24h

$resourceGroup = "RG-IndustrialBackup-WestEurope"
$storageAccountName = "industrialbackupweu"
$actionGroupId = "/subscriptions/{sub-id}/resourceGroups/$resourceGroup/providers/microsoft.insights/actionGroups/AG-IndustrialBackup"

$condition = New-AzMetricAlertRuleV2Criteria `
    -MetricName "BlobCount" `
    -TimeAggregation Total `
    -Operator LessThan `
    -Threshold 1

Add-AzMetricAlertRuleV2 `
    -Name "Alert-NoUploadsLast24h" `
    -ResourceGroupName $resourceGroup `
    -WindowSize (New-TimeSpan -Hours 24) `
    -Frequency (New-TimeSpan -Hours 6) `
    -TargetResourceId "/subscriptions/{sub-id}/resourceGroups/$resourceGroup/providers/Microsoft.Storage/storageAccounts/$storageAccountName" `
    -Condition $condition `
    -ActionGroupId $actionGroupId `
    -Severity 0 `
    -Description "No blobs uploaded in last 24 hours - possible service outage"
```

### 7.3 Action Groups

**AG-IndustrialBackup-Critical:**

- Email: it-ops@company.com, cloud-team@company.com
- PagerDuty: Integration key `xxx`
- SMS: +XX XXX XXX XXX (on-call phone)

**AG-IndustrialBackup-Warning:**

- Email: it-ops@company.com
- Slack: #industrial-backup channel webhook

---

## 8. Synthetic Monitoring

### 8.1 Health Check Script

```powershell
# Script: D:\ImageBackup\Scripts\HealthCheck.ps1
# Ejecutado cada hora por Task Scheduler, envía heartbeat a Azure Monitor

@'
# Health check script
$healthStatus = @{
    IncomingFolderAccessible = Test-Path "D:\ImageBackup\Incoming"
    StagingFolderAccessible = Test-Path "D:\ImageBackup\Staging"
    AzureConnectivity = Test-NetConnection -ComputerName industrialbackupweu.blob.core.windows.net -Port 443 -InformationLevel Quiet
    DiskSpaceOK = (Get-Volume -DriveLetter D).SizeRemaining -gt 100GB
    TaskSchedulerRunning = (Get-ScheduledTask -TaskName "IndustrialBackup-*" | Where-Object State -eq "Ready").Count -eq 3
}

$allHealthy = ($healthStatus.Values | Where-Object { $_ -eq $false }).Count -eq 0

# Enviar métrica heartbeat a Azure Monitor
$customMetrics = @{
    ResourceId = "/subscriptions/{sub-id}/resourceGroups/RG-IndustrialBackup-WestEurope/providers/Microsoft.Compute/virtualMachines/ITGW-FACTORY-EU01"
    MetricName = "ServiceHealthy"
    MetricValue = $(if ($allHealthy) { 1 } else { 0 })
}

# TODO: Implement metric submission to Azure Monitor

if (-not $allHealthy) {
    Write-EventLog -LogName Application -Source "IndustrialBackup" -EventId 1001 -EntryType Error -Message "Health check failed: $(ConvertTo-Json $healthStatus)"
}
'@ | Out-File -FilePath "D:\ImageBackup\Scripts\HealthCheck.ps1" -Encoding UTF8
```

---

## 9. Capacity Monitoring

### 9.1 Azure Storage Growth

```kql
// Proyección de crecimiento de storage (próximos 12 meses)
AzureMetrics
| where ResourceId contains "industrialbackupweu"
| where MetricName == "BlobCapacity"
| where TimeGenerated > ago(90d)
| summarize StorageGB = max(Total) / 1024 / 1024 / 1024 by bin(TimeGenerated, 1d)
| extend DailyGrowth = StorageGB - prev(StorageGB, 1)
| summarize AvgDailyGrowth_GB = avg(DailyGrowth)
| extend ProjectedIn12Months_TB = (AvgDailyGrowth_GB * 365) / 1024
```

### 9.2 VM Disk Capacity Planning

```kql
// Proyección de cuándo se llenará el disco D:\ (basado en trend de últimos 30 días)
Perf
| where TimeGenerated > ago(30d)
| where ObjectName == "LogicalDisk" and CounterName == "Free Megabytes" and InstanceName == "D:"
| summarize AvgFreeMB = avg(CounterValue) by bin(TimeGenerated, 1d)
| extend DailyDecrease = prev(AvgFreeMB, 1) - AvgFreeMB
| summarize AvgDailyDecrease_MB = avg(DailyDecrease)
| extend CurrentFree_GB = toscalar(Perf | where TimeGenerated > ago(1h) | where InstanceName == "D:" | summarize avg(CounterValue) / 1024)
| extend DaysUntilFull = CurrentFree_GB * 1024 / AvgDailyDecrease_MB
```

---

## 10. Troubleshooting Playbooks

### 10.1 Alert: No uploads in 24h

**Symptoms:** Alert "No uploads in 24h" fires

**Investigation:**

```powershell
# 1. Check if VM is running
Get-AzVM -ResourceGroupName "RG-IndustrialBackup-WestEurope" -Name "ITGW-FACTORY-EU01" -Status

# 2. Check Task Scheduler tasks
Get-ScheduledTask | Where-Object TaskName -like "IndustrialBackup-*" | Select TaskName, State, LastRunTime, LastTaskResult

# 3. Check recent logs for errors
Get-Content "D:\ImageBackup\Logs\Upload-$(Get-Date -Format 'yyyy-MM-dd').log" | Select-String "ERROR"

# 4. Check if files are arriving in Incoming/
Get-ChildItem "D:\ImageBackup\Incoming" | Measure-Object

# 5. Test Azure connectivity
Test-NetConnection -ComputerName industrialbackupweu.blob.core.windows.net -Port 443
```

**Resolution:**

- Si VM stopped → Start VM
- Si Task failed → Revisar logs y restart manualmente
- Si no hay archivos en Incoming → Problema en OT, escalar a IT Local
- Si no hay conectividad Azure → Problema de red, escalar a Network Team

### 10.2 Alert: High error rate

**Symptoms:** Upload error rate > 5%

**Investigation:**

```kql
IndustrialBackup_CL
| where TimeGenerated > ago(1h)
| where RawData contains "AzCopy failed"
| project TimeGenerated, ErrorMessage = extract(@"Error: (.+)", 1, RawData)
| summarize ErrorCount = count() by ErrorMessage
| order by ErrorCount desc
```

**Common errors:**

- "AuthenticationFailed" → Managed Identity issue, recreate role assignment
- "BlobAlreadyExists" → Duplicate filename, check Staging/ cleanup
- "RequestTimeout" → Network congestion, retry automatically

---

## 11. SLI/SLO Reporting

### 11.1 Monthly SLO Report (automated)

```kql
// Query ejecutado el día 1 de cada mes para reportar SLO del mes anterior
let StartOfLastMonth = startofmonth(ago(30d));
let EndOfLastMonth = endofmonth(ago(30d));

IndustrialBackup_CL
| where TimeGenerated between (StartOfLastMonth .. EndOfLastMonth)
| extend UploadSuccess = iff(RawData contains "Upload complete", 1, 0)
| summarize 
    TotalUploads = count(),
    SuccessfulUploads = countif(UploadSuccess == 1),
    FailedUploads = countif(UploadSuccess == 0)
| extend SuccessRate = 100.0 * SuccessfulUploads / TotalUploads
| extend SLO_Target = 99.5
| extend SLO_Met = iff(SuccessRate >= SLO_Target, "✅ YES", "❌ NO")
| project 
    Month = format_datetime(StartOfLastMonth, "yyyy-MM"),
    TotalUploads,
    SuccessfulUploads,
    FailedUploads,
    SuccessRate = round(SuccessRate, 2),
    SLO_Target,
    SLO_Met
```

Output enviado automáticamente a Service Owner el día 1 de cada mes.

---

## 12. Monitoring Maintenance

### 12.1 Quarterly Review Tasks

- [ ] Review alert rules — are they still appropriate? Too noisy?
- [ ] Review dashboard — add/remove widgets based on usage
- [ ] Review log retention policies — adjust if needed
- [ ] Test alert routing — verify emails/PagerDuty still work
- [ ] Update runbooks — incorporate new learnings from incidents

### 12.2 Annual Review

- [ ] Re-baseline traffic patterns (files/day, size/day)
- [ ] Update capacity projections
- [ ] Review SLO targets — adjust if needed
- [ ] Audit monitoring costs — optimize if spending > $100/mes

---

## 13. Change History

| Version | Fecha      | Autor                 | Cambios                               |
| ------- | ---------- | --------------------- | ------------------------------------- |
| 1.0     | 2025-02-19 | Public Cloud + IT Ops | Versión inicial — Monitoring strategy |

---

**Notes:**

- Dashboards deben ser accesibles a todos los stakeholders (incluido Negocio para transparency)
- Alertas PagerDuty solo para severidad Critical, no Warning (evitar alert fatigue)
- Logs PowerShell deben incluir SIEMPRE timestamp y nivel (INFO/WARN/ERROR)
