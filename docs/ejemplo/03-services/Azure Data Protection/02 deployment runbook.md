# Industrial Image Backup to Azure — Deployment Runbook

> **Version:** 1.0  
> **Fecha:** 2025-02-19  
> **Estado:** Approved  
> **Autor:** Public Cloud Team  
> **Service Owner:** Arquitectura IT  
> **Última Revisión:** 2025-02-19

---

## 1. Overview

**Propósito de este Runbook:**
Procedimiento completo para desplegar el servicio de Industrial Image Backup en una nueva planta industrial, incluyendo configuración de VM gateway, scripts PowerShell, Azure resources, y firewall Checkpoint.

**Tiempo estimado:** 4-6 horas (primera instalación), 2-3 horas (plantas subsecuentes)

**Prerequisitos:**
- [ ] VM Windows Server 2022 provisionada en red IT de la planta
- [ ] Conectividad ExpressRoute o VPN site-to-site a Azure establecida
- [ ] Acceso de red desde VM IT hacia Azure (puerto 443 outbound)
- [ ] Firewall Checkpoint configurado entre red OT y red IT
- [ ] Service Principal creado en Azure con permisos Storage Blob Data Contributor
- [ ] Azure Storage Account creado en región apropiada

**Responsables:**
- **Despliegue inicial:** Public Cloud Team + IT Operations
- **Soporte:** IT Operations (local) + Public Cloud (remoto)

---

## 2. Pre-Deployment Checklist

### 2.1 Información Necesaria

Completa esta tabla ANTES de comenzar el deployment:

| Item | Valor | Ejemplo | Obtenido de |
|------|-------|---------|-------------|
| **Código de Planta** | [FACTORY-XX] | FACTORY-EU01 | Negocio |
| **Región Azure** | [West Europe / East US] | West Europe | Arquitectura IT |
| **Red OT CIDR** | [10.x.x.x/16] | 10.100.0.0/16 | IT Local |
| **IP(s) servidores OT** | [10.x.x.10-20] | 10.100.10.15 | IT Local |
| **Red IT CIDR** | [192.168.x.x/24] | 192.168.10.0/24 | IT Local |
| **IP VM Gateway** | [192.168.x.x] | 192.168.10.50 | IT Local |
| **Nombre VM Gateway** | [ITGW-FACTORY-XX] | ITGW-FACTORY-EU01 | Naming convention |
| **Storage Account Name** | [industrialbackup{region}] | industrialbackupweu | Arquitectura IT (ya existe) |
| **Container Name** | [plant-{code}-images] | plant-eu01-images | Auto-generado |
| **Volumen mensual estimado** | [100-500 GB] | 350 GB | Negocio |
| **Contact IT Local** | [Nombre] | Juan García | IT Local |
| **Contact Negocio** | [Nombre] | María López | Negocio |

### 2.2 Azure Resources Required (verificar existencia)

- [ ] **Subscription:** `Industrial-Operations`
- [ ] **Resource Group:** `RG-IndustrialBackup-{Region}`
- [ ] **Storage Account:** `industrialbackup{region}` (ej: `industrialbackupweu`)
- [ ] **Key Vault:** `kv-industrialbackup-{region}`
- [ ] **Service Principal:** `sp-industrial-backup-upload`
- [ ] **Log Analytics Workspace:** `law-industrialbackup-{region}`

Si alguno NO existe, ejecutar terraform en `infrastructure/azure/` antes de continuar.

### 2.3 Access Requirements

Permisos necesarios para el deploymen

t:

| Persona/Rol | Acceso Necesario | Duración |
|-------------|------------------|----------|
| Cloud Engineer | Owner en Resource Group Azure | Durante deployment |
| Cloud Engineer | Local Admin en VM Gateway | Durante deployment |
| IT Operations | Local Admin en VM Gateway | Permanente (soporte) |
| Network Engineer | Admin en Checkpoint Firewall | Durante configuración FW |

---

## 3. Deployment Steps

### 3.1 Azure Resources Deployment

#### 3.1.1 Crear Blob Storage Container (si no existe)

```powershell
# Login a Azure
Connect-AzAccount -Tenant company.onmicrosoft.com

# Variables
$subscriptionName = "Industrial-Operations"
$resourceGroup = "RG-IndustrialBackup-WestEurope"
$storageAccountName = "industrialbackupweu"
$containerName = "plant-eu01-images"  # Cambiar según código de planta

# Seleccionar subscription
Set-AzContext -Subscription $subscriptionName

# Crear container
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName
$ctx = $storageAccount.Context
New-AzStorageContainer -Name $containerName -Context $ctx -Permission Off

# Verificar creación
Get-AzStorageContainer -Name $containerName -Context $ctx
```

#### 3.1.2 Configurar Lifecycle Policy

```powershell
# Definir lifecycle policy
$rule1 = New-AzStorageAccountManagementPolicyRule `
    -Name "MoveToArchiveAfter365Days" `
    -BlobType blockBlob `
    -TierToArchiveAfterDaysSinceCreationGreaterThan 365

$rule2 = New-AzStorageAccountManagementPolicyRule `
    -Name "MoveToCoolImmediately" `
    -BlobType blockBlob `
    -TierToCoolAfterDaysSinceCreationGreaterThan 0

# Aplicar policy
$policy = Set-AzStorageAccountManagementPolicy `
    -ResourceGroupName $resourceGroup `
    -AccountName $storageAccountName `
    -Rule $rule1, $rule2

# Verificar policy
Get-AzStorageAccountManagementPolicy -ResourceGroupName $resourceGroup -AccountName $storageAccountName
```

**Explicación:**
- `TierToCoolAfterDaysSinceCreationGreaterThan 0` → Mueve a Cool tier inmediatamente tras upload
- `TierToArchiveAfterDaysSinceCreationGreaterThan 365` → Mueve a Archive tier después de 1 año

#### 3.1.3 Configurar RBAC para Service Principal

```powershell
# Variables
$spObjectId = (Get-AzADServicePrincipal -DisplayName "sp-industrial-backup-upload").Id
$scope = "/subscriptions/{subscription-id}/resourceGroups/$resourceGroup/providers/Microsoft.Storage/storageAccounts/$storageAccountName/blobServices/default/containers/$containerName"

# Asignar rol Storage Blob Data Contributor
New-AzRoleAssignment `
    -ObjectId $spObjectId `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope $scope

# Verificar asignación
Get-AzRoleAssignment -ObjectId $spObjectId -Scope $scope
```

#### 3.1.4 Habilitar Soft Delete (protección contra borrado accidental)

```powershell
# Habilitar soft delete para blobs (14 días retención)
Enable-AzStorageBlobDeleteRetentionPolicy `
    -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccountName `
    -RetentionDays 14

# Verificar
Get-AzStorageBlobServiceProperty -ResourceGroupName $resourceGroup -StorageAccountName $storageAccountName
```

---

### 3.2 VM Gateway Deployment

#### 3.2.1 Provisionar VM Windows Server 2022

**Opción A: Azure Portal (manual)**
1. Azure Portal → Virtual Machines → Create
2. **Basics:**
   - Subscription: `Industrial-Operations`
   - Resource Group: `RG-IndustrialBackup-{Region}`
   - VM Name: `ITGW-FACTORY-EU01`
   - Region: `West Europe`
   - Image: `Windows Server 2022 Datacenter`
   - Size: `Standard_D4s_v5` (4 vCPU, 16 GB RAM)
3. **Disks:**
   - OS Disk: 127 GB Premium SSD
   - Data Disk: Add 1 TB Premium SSD → Mount as D:\
4. **Networking:**
   - Virtual Network: `vnet-factory-it` (debe existir en planta)
   - Subnet: `subnet-it-servers`
   - Public IP: None (VM solo accesible via ExpressRoute)
   - NIC Network Security Group: `nsg-industrial-backup-gateway`
5. **Management:**
   - Enable Managed Identity: **System-assigned** ✅
   - Enable auto-shutdown: No
6. **Create**

**Opción B: Terraform (recomendado)**

```hcl
# infrastructure/azure/vm-gateway.tf
resource "azurerm_windows_virtual_machine" "gateway" {
  name                = "ITGW-${var.factory_code}"
  resource_group_name = azurerm_resource_group.main.name
  location            = var.azure_region
  size                = "Standard_D4s_v5"
  admin_username      = "azureadmin"
  admin_password      = random_password.vm_password.result

  identity {
    type = "SystemAssigned"
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }

  network_interface_ids = [
    azurerm_network_interface.gateway.id,
  ]

  tags = {
    Service = "IndustrialBackup"
    Factory = var.factory_code
  }
}

# Data disk 1 TB
resource "azurerm_managed_disk" "data" {
  name                 = "${azurerm_windows_virtual_machine.gateway.name}-datadisk"
  location             = var.azure_region
  resource_group_name  = azurerm_resource_group.main.name
  storage_account_type = "Premium_LRS"
  create_option        = "Empty"
  disk_size_gb         = 1024
}

resource "azurerm_virtual_machine_data_disk_attachment" "data" {
  managed_disk_id    = azurerm_managed_disk.data.id
  virtual_machine_id = azurerm_windows_virtual_machine.gateway.id
  lun                = 0
  caching            = "ReadWrite"
}
```

Deploy con Terraform:
```bash
cd infrastructure/azure
terraform init
terraform plan -var="factory_code=EU01" -var="azure_region=westeurope"
terraform apply
```

#### 3.2.2 Configuración Inicial OS

Conectarse a la VM via RDP (a través de Bastion o VPN):

```powershell
# 1. Inicializar data disk D:\
Get-Disk | Where-Object PartitionStyle -eq 'RAW' | 
    Initialize-Disk -PartitionStyle MBR -PassThru | 
    New-Partition -DriveLetter D -UseMaximumSize | 
    Format-Volume -FileSystem NTFS -NewFileSystemLabel "ImageBackup" -Confirm:$false

# 2. Crear estructura de directorios
New-Item -Path "D:\ImageBackup" -ItemType Directory
New-Item -Path "D:\ImageBackup\Incoming" -ItemType Directory
New-Item -Path "D:\ImageBackup\Staging" -ItemType Directory
New-Item -Path "D:\ImageBackup\Processed" -ItemType Directory
New-Item -Path "D:\ImageBackup\Scripts" -ItemType Directory
New-Item -Path "D:\ImageBackup\Logs" -ItemType Directory

# 3. Configurar SMB share para recibir archivos desde OT
New-SmbShare -Name "ImageBackup" `
    -Path "D:\ImageBackup\Incoming" `
    -FullAccess "DOMAIN\OT-Upload-Account" `
    -ReadAccess "Everyone" `
    -Description "Industrial image backup incoming from OT"

# Verificar share
Get-SmbShare -Name "ImageBackup"

# 4. Configurar NTFS permissions
$acl = Get-Acl "D:\ImageBackup\Incoming"
$permission = "DOMAIN\OT-Upload-Account","FullControl","ContainerInherit,ObjectInherit","None","Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($accessRule)
Set-Acl "D:\ImageBackup\Incoming" $acl

# 5. Instalar módulos PowerShell necesarios
Install-Module -Name Az.Storage -Scope AllUsers -Force
Install-Module -Name Az.Monitor -Scope AllUsers -Force
Install-Module -Name Az.Accounts -Scope AllUsers -Force

# 6. Descargar AzCopy
$azcopyUrl = "https://aka.ms/downloadazcopy-v10-windows"
$azcopyZip = "D:\ImageBackup\Scripts\azcopy.zip"
Invoke-WebRequest -Uri $azcopyUrl -OutFile $azcopyZip
Expand-Archive -Path $azcopyZip -DestinationPath "D:\ImageBackup\Scripts\AzCopy"
Remove-Item $azcopyZip

# 7. Agregar AzCopy al PATH
$azcopyPath = (Get-ChildItem -Path "D:\ImageBackup\Scripts\AzCopy" -Recurse -Filter "azcopy.exe").DirectoryName
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$azcopyPath", "Machine")

# Verificar instalación
azcopy --version
```

#### 3.2.3 Configurar Managed Identity Authentication

```powershell
# Login con Managed Identity (ejecutar en la VM)
Connect-AzAccount -Identity

# Verificar que puede acceder al Storage Account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-IndustrialBackup-WestEurope" -Name "industrialbackupweu"
Write-Host "✅ Managed Identity authentication successful" -ForegroundColor Green

# Test upload de archivo dummy
$testFile = "D:\test.txt"
"test" | Out-File $testFile
$ctx = $storageAccount.Context
Set-AzStorageBlobContent -File $testFile -Container "plant-eu01-images" -Blob "test.txt" -Context $ctx

# Limpiar test
Remove-AzStorageBlob -Container "plant-eu01-images" -Blob "test.txt" -Context $ctx
Remove-Item $testFile
```

---

### 3.3 PowerShell Scripts Deployment

#### 3.3.1 Script 1: Monitor-IncomingImages.ps1

```powershell
# Crear archivo D:\ImageBackup\Scripts\Monitor-IncomingImages.ps1

@'
<#
.SYNOPSIS
    Detecta nuevos archivos de imagen en Incoming/ y los prepara para upload.
.DESCRIPTION
    Escanea cada 5 minutos la carpeta Incoming/ y mueve archivos nuevos a Staging/
    para procesamiento. Valida que los archivos sean imágenes válidas (extensiones permitidas).
.NOTES
    Ejecutado por Task Scheduler cada 5 minutos.
#>

# Configuración
$IncomingPath = "D:\ImageBackup\Incoming"
$StagingPath = "D:\ImageBackup\Staging"
$LogPath = "D:\ImageBackup\Logs\Monitor-$(Get-Date -Format 'yyyy-MM-dd').log"

# Extensiones permitidas
$AllowedExtensions = @(".jpg", ".jpeg", ".png", ".tiff", ".tif", ".bmp")

# Función de logging
function Write-Log {
    param($Message, $Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "[$timestamp] [$Level] $Message"
    Add-Content -Path $LogPath -Value $logMessage
    Write-Host $logMessage
}

Write-Log "=== Starting file monitoring ==="

try {
    # Verificar que carpetas existen
    if (-not (Test-Path $IncomingPath)) {
        throw "Incoming path does not exist: $IncomingPath"
    }
    if (-not (Test-Path $StagingPath)) {
        New-Item -Path $StagingPath -ItemType Directory | Out-Null
    }

    # Buscar archivos en Incoming
    $files = Get-ChildItem -Path $IncomingPath -File | 
        Where-Object { $AllowedExtensions -contains $_.Extension.ToLower() }

    Write-Log "Found $($files.Count) image files in Incoming"

    foreach ($file in $files) {
        try {
            # Verificar que el archivo no está siendo escrito (esperar si tamaño cambia)
            $initialSize = $file.Length
            Start-Sleep -Seconds 2
            $file.Refresh()
            if ($file.Length -ne $initialSize) {
                Write-Log "File $($file.Name) is still being written, skipping" "WARN"
                continue
            }

            # Mover a Staging
            $destPath = Join-Path $StagingPath $file.Name
            
            # Si ya existe en Staging, renombrar con timestamp
            if (Test-Path $destPath) {
                $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
                $destPath = Join-Path $StagingPath "$($file.BaseName)_$timestamp$($file.Extension)"
            }

            Move-Item -Path $file.FullName -Destination $destPath -Force
            Write-Log "Moved $($file.Name) to Staging ($([math]::Round($file.Length/1MB, 2)) MB)"

        } catch {
            Write-Log "Error processing file $($file.Name): $_" "ERROR"
        }
    }

    Write-Log "=== Monitoring complete ==="

} catch {
    Write-Log "Fatal error: $_" "ERROR"
    exit 1
}

exit 0
'@ | Out-File -FilePath "D:\ImageBackup\Scripts\Monitor-IncomingImages.ps1" -Encoding UTF8
```

#### 3.3.2 Script 2: Upload-ToAzure.ps1

```powershell
# Crear archivo D:\ImageBackup\Scripts\Upload-ToAzure.ps1

@'
<#
.SYNOPSIS
    Sube archivos desde Staging/ a Azure Blob Storage usando AzCopy.
.DESCRIPTION
    Usa AzCopy para upload eficiente de múltiples archivos con retry automático.
    Verifica integridad con MD5 hash y mueve archivos exitosos a Processed/.
.NOTES
    Ejecutado por Task Scheduler cada 10 minutos.
#>

# Configuración
$StagingPath = "D:\ImageBackup\Staging"
$ProcessedPath = "D:\ImageBackup\Processed"
$LogPath = "D:\ImageBackup\Logs\Upload-$(Get-Date -Format 'yyyy-MM-dd').log"

# Azure Config
$StorageAccountName = "industrialbackupweu"  # Cambiar según región
$ContainerName = "plant-eu01-images"          # Cambiar según planta
$BlobUrl = "https://$StorageAccountName.blob.core.windows.net/$ContainerName"

function Write-Log {
    param($Message, $Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "[$timestamp] [$Level] $Message"
    Add-Content -Path $LogPath -Value $logMessage
    Write-Host $logMessage
}

Write-Log "=== Starting Azure upload ==="

try {
    # Login con Managed Identity
    Connect-AzAccount -Identity | Out-Null
    Write-Log "Authenticated with Managed Identity"

    # Verificar carpetas
    if (-not (Test-Path $StagingPath)) {
        Write-Log "No staging folder found, exiting"
        exit 0
    }
    if (-not (Test-Path $ProcessedPath)) {
        New-Item -Path $ProcessedPath -ItemType Directory | Out-Null
    }

    # Buscar archivos en Staging
    $files = Get-ChildItem -Path $StagingPath -File
    if ($files.Count -eq 0) {
        Write-Log "No files to upload"
        exit 0
    }

    Write-Log "Found $($files.Count) files to upload"

    # Upload con AzCopy (batch upload)
    Write-Log "Starting AzCopy upload..."
    
    $azcopyCmd = "azcopy copy `"$StagingPath\*`" `"$BlobUrl`" --recursive=false --check-md5=FailIfDifferent --log-level=INFO"
    
    $result = Invoke-Expression $azcopyCmd 2>&1
    Write-Log "AzCopy output: $result"

    # Verificar éxito de upload
    $uploadSuccess = $true
    if ($LASTEXITCODE -ne 0) {
        Write-Log "AzCopy failed with exit code $LASTEXITCODE" "ERROR"
        $uploadSuccess = $false
    }

    # Si upload exitoso, mover archivos a Processed/
    if ($uploadSuccess) {
        foreach ($file in $files) {
            try {
                $destPath = Join-Path $ProcessedPath $file.Name
                
                # Si ya existe, renombrar con timestamp
                if (Test-Path $destPath) {
                    $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
                    $destPath = Join-Path $ProcessedPath "$($file.BaseName)_$timestamp$($file.Extension)"
                }

                Move-Item -Path $file.FullName -Destination $destPath -Force
                Write-Log "Uploaded and moved $($file.Name) to Processed"

                # Enviar métrica custom a Azure Monitor
                $metric = @{
                    name = "FilesUploaded"
                    value = 1
                    timestamp = (Get-Date).ToUniversalTime()
                }
                # TODO: Implementar envío de métrica con Az.Monitor

            } catch {
                Write-Log "Error moving file $($file.Name): $_" "ERROR"
            }
        }

        Write-Log "✅ Upload complete: $($files.Count) files uploaded successfully"
    } else {
        Write-Log "❌ Upload failed, files remain in Staging for retry" "ERROR"
    }

} catch {
    Write-Log "Fatal error: $_" "ERROR"
    exit 1
}

Write-Log "=== Upload process complete ==="
exit 0
'@ | Out-File -FilePath "D:\ImageBackup\Scripts\Upload-ToAzure.ps1" -Encoding UTF8
```

#### 3.3.3 Script 3: Cleanup-LocalFiles.ps1

```powershell
# Crear archivo D:\ImageBackup\Scripts\Cleanup-LocalFiles.ps1

@'
<#
.SYNOPSIS
    Limpia archivos antiguos de la carpeta Processed/ para liberar espacio.
.DESCRIPTION
    Elimina archivos en Processed/ que tengan más de 7 días de antigüedad.
    Ventana de 7 días permite troubleshooting en caso de problemas con uploads.
.NOTES
    Ejecutado por Task Scheduler diariamente a las 03:00.
#>

# Configuración
$ProcessedPath = "D:\ImageBackup\Processed"
$RetentionDays = 7
$LogPath = "D:\ImageBackup\Logs\Cleanup-$(Get-Date -Format 'yyyy-MM-dd').log"

function Write-Log {
    param($Message, $Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "[$timestamp] [$Level] $Message"
    Add-Content -Path $LogPath -Value $logMessage
    Write-Host $logMessage
}

Write-Log "=== Starting cleanup process ==="

try {
    if (-not (Test-Path $ProcessedPath)) {
        Write-Log "Processed folder does not exist, nothing to clean"
        exit 0
    }

    $cutoffDate = (Get-Date).AddDays(-$RetentionDays)
    Write-Log "Deleting files older than $cutoffDate"

    $oldFiles = Get-ChildItem -Path $ProcessedPath -File | 
        Where-Object { $_.LastWriteTime -lt $cutoffDate }

    if ($oldFiles.Count -eq 0) {
        Write-Log "No old files to delete"
        exit 0
    }

    $totalSize = ($oldFiles | Measure-Object -Property Length -Sum).Sum
    $totalSizeMB = [math]::Round($totalSize / 1MB, 2)

    Write-Log "Found $($oldFiles.Count) files to delete (total: $totalSizeMB MB)"

    foreach ($file in $oldFiles) {
        try {
            Remove-Item -Path $file.FullName -Force
            Write-Log "Deleted: $($file.Name)"
        } catch {
            Write-Log "Error deleting $($file.Name): $_" "ERROR"
        }
    }

    Write-Log "✅ Cleanup complete: $($oldFiles.Count) files deleted, $totalSizeMB MB freed"

} catch {
    Write-Log "Fatal error: $_" "ERROR"
    exit 1
}

Write-Log "=== Cleanup process complete ==="
exit 0
'@ | Out-File -FilePath "D:\ImageBackup\Scripts\Cleanup-LocalFiles.ps1" -Encoding UTF8
```

#### 3.3.4 Configurar Task Scheduler

```powershell
# Script 1: Monitor (cada 5 minutos)
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File `"D:\ImageBackup\Scripts\Monitor-IncomingImages.ps1`""
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 5) -RepetitionDuration ([TimeSpan]::MaxValue)
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName "IndustrialBackup-Monitor" -Action $action -Trigger $trigger -Principal $principal -Description "Monitor incoming images from OT"

# Script 2: Upload (cada 10 minutos)
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File `"D:\ImageBackup\Scripts\Upload-ToAzure.ps1`""
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 10) -RepetitionDuration ([TimeSpan]::MaxValue)
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName "IndustrialBackup-Upload" -Action $action -Trigger $trigger -Principal $principal -Description "Upload images to Azure Blob Storage"

# Script 3: Cleanup (diario a las 03:00)
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File `"D:\ImageBackup\Scripts\Cleanup-LocalFiles.ps1`""
$trigger = New-ScheduledTaskTrigger -Daily -At "03:00"
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName "IndustrialBackup-Cleanup" -Action $action -Trigger $trigger -Principal $principal -Description "Cleanup old processed files"

# Verificar tasks creadas
Get-ScheduledTask | Where-Object { $_.TaskName -like "IndustrialBackup-*" }
```

---

### 3.4 Checkpoint Firewall Configuration

#### 3.4.1 Firewall Rules (OT → IT)

**Regla 1: Permitir SMB desde OT a VM Gateway**

```
Name: OT-to-IT-ImageBackup-SMB
Source:
  - 10.100.10.15/32    # Servidor OT que copia imágenes
  - 10.100.10.16/32    # Backup OT server (opcional)
Destination:
  - 192.168.10.50/32   # VM Gateway IT
Service:
  - TCP 445 (SMB)
Action: Accept
Track: Log
Time: Any
```

**Regla 2: Denegar todo lo demás OT → IT**

```
Name: OT-to-IT-Default-Deny
Source: 10.100.0.0/16  # Red OT completa
Destination: 192.168.10.0/24  # Red IT
Service: Any
Action: Drop
Track: Log
Time: Any
```

**Validación:**

```bash
# Desde servidor OT, test SMB connectivity
Test-NetConnection -ComputerName 192.168.10.50 -Port 445

# Expected output: TcpTestSucceeded : True
```

#### 3.4.2 NSG Rules (Azure - VM Gateway)

```powershell
# Crear NSG si no existe
$nsg = New-AzNetworkSecurityGroup -Name "nsg-industrial-backup-gateway" `
    -ResourceGroupName "RG-IndustrialBackup-WestEurope" `
    -Location "WestEurope"

# Regla 1: Allow HTTPS outbound to Azure Storage
$nsg | Add-AzNetworkSecurityRuleConfig -Name "Allow-HTTPS-To-Azure" `
    -Access Allow -Protocol Tcp -Direction Outbound `
    -Priority 100 -SourceAddressPrefix * -SourcePortRange * `
    -DestinationAddressPrefix "Storage" -DestinationPortRange 443 | Set-AzNetworkSecurityGroup

# Regla 2: Allow SMB inbound from OT network (via ExpressRoute)
$nsg | Add-AzNetworkSecurityRuleConfig -Name "Allow-SMB-From-OT" `
    -Access Allow -Protocol Tcp -Direction Inbound `
    -Priority 200 -SourceAddressPrefix "10.100.10.0/24" -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 445 | Set-AzNetworkSecurityGroup

# Regla 3: Allow RDP from IT Management subnet
$nsg | Add-AzNetworkSecurityRuleConfig -Name "Allow-RDP-From-Mgmt" `
    -Access Allow -Protocol Tcp -Direction Inbound `
    -Priority 300 -SourceAddressPrefix "192.168.1.0/24" -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389 | Set-AzNetworkSecurityGroup

# Regla 4: Deny all other inbound
$nsg | Add-AzNetworkSecurityRuleConfig -Name "Deny-All-Inbound" `
    -Access Deny -Protocol * -Direction Inbound `
    -Priority 4096 -SourceAddressPrefix * -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange * | Set-AzNetworkSecurityGroup
```

---

### 3.5 OT-Side Configuration

#### 3.5.1 Script en Servidor OT para Copiar Imágenes

**Nota:** Este script debe ser configurado por IT Local en el servidor OT que genera las imágenes.

```powershell
# Crear en servidor OT: C:\Scripts\Copy-ImagesToIT.ps1

@'
<#
.SYNOPSIS
    Copia imágenes desde carpeta local OT a share SMB en VM Gateway IT.
.DESCRIPTION
    Ejecutado cada hora, copia imágenes nuevas desde carpeta de generación OT
    hacia el share SMB \\ITGW-FACTORY-EU01\ImageBackup.
#>

# Configuración
$SourcePath = "E:\ProductionImages"  # Carpeta donde equipos OT guardan imágenes
$DestinationShare = "\\192.168.10.50\ImageBackup"  # VM Gateway IT
$LogPath = "C:\Logs\ImageCopy-$(Get-Date -Format 'yyyy-MM-dd').log"

# Credenciales (usar cuenta de servicio AD)
$username = "DOMAIN\OT-Upload-Account"
$password = ConvertTo-SecureString "Password123!" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($username, $password)

function Write-Log {
    param($Message)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    Add-Content -Path $LogPath -Value "[$timestamp] $Message"
}

Write-Log "=== Starting image copy to IT ==="

try {
    # Conectar al share SMB
    New-PSDrive -Name "Z" -PSProvider FileSystem -Root $DestinationShare -Credential $credential -ErrorAction Stop | Out-Null
    Write-Log "Connected to IT share"

    # Buscar imágenes no copiadas (últimas 24 horas)
    $files = Get-ChildItem -Path $SourcePath -File -Recurse | 
        Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-24) -and $_.Extension -match '\.(jpg|jpeg|png|tiff|tif|bmp)$' }

    Write-Log "Found $($files.Count) images to copy"

    foreach ($file in $files) {
        try {
            $destPath = Join-Path "Z:\" $file.Name
            
            # Verificar si ya existe (skip duplicados)
            if (Test-Path $destPath) {
                Write-Log "File $($file.Name) already exists, skipping"
                continue
            }

            # Copiar archivo
            Copy-Item -Path $file.FullName -Destination $destPath -Force
            Write-Log "Copied $($file.Name) ($([math]::Round($file.Length/1MB, 2)) MB)"

        } catch {
            Write-Log "Error copying $($file.Name): $_"
        }
    }

    # Desconectar share
    Remove-PSDrive -Name "Z" -Force

    Write-Log "=== Copy complete ==="

} catch {
    Write-Log "Fatal error: $_"
}
'@ | Out-File -FilePath "C:\Scripts\Copy-ImagesToIT.ps1" -Encoding UTF8

# Configurar Task Scheduler en servidor OT (cada hora)
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File `"C:\Scripts\Copy-ImagesToIT.ps1`""
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 1) -RepetitionDuration ([TimeSpan]::MaxValue)
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName "ImageBackup-CopyToIT" -Action $action -Trigger $trigger -Principal $principal
```

---

## 4. Post-Deployment Validation

### 4.1 Functional Testing

#### Test 1: Conectividad OT → IT (SMB)
```powershell
# Desde servidor OT
Test-NetConnection -ComputerName 192.168.10.50 -Port 445
# Expected: TcpTestSucceeded = True

# Test escribir archivo
New-Item -Path "\\192.168.10.50\ImageBackup\test.txt" -ItemType File
# Expected: Archivo creado sin errores
```

#### Test 2: Upload a Azure
```powershell
# Desde VM Gateway IT
# Crear archivo de prueba
$testFile = "D:\ImageBackup\Staging\test-image.jpg"
"dummy content" | Out-File $testFile

# Ejecutar script de upload manualmente
& "D:\ImageBackup\Scripts\Upload-ToAzure.ps1"

# Verificar en Azure Portal que el archivo aparece en container
# O usar PowerShell:
Connect-AzAccount -Identity
$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-IndustrialBackup-WestEurope" -Name "industrialbackupweu"
Get-AzStorageBlob -Container "plant-eu01-images" -Context $storageAccount.Context | Where-Object { $_.Name -eq "test-image.jpg" }

# Expected: Blob encontrado con LastModified reciente
```

#### Test 3: Lifecycle Policy
```powershell
# Verificar que el blob está en Cool tier (no Archive inmediatamente)
$blob = Get-AzStorageBlob -Container "plant-eu01-images" -Blob "test-image.jpg" -Context $storageAccount.Context
$blob.ICloudBlob.Properties.StandardBlobTier

# Expected: "Cool" (no "Archive")
```

#### Test 4: Cleanup
```powershell
# Crear archivo antiguo en Processed/
$oldFile = "D:\ImageBackup\Processed\old-test.jpg"
"old" | Out-File $oldFile
(Get-Item $oldFile).LastWriteTime = (Get-Date).AddDays(-10)

# Ejecutar cleanup
& "D:\ImageBackup\Scripts\Cleanup-LocalFiles.ps1"

# Verificar que el archivo fue eliminado
Test-Path $oldFile
# Expected: False
```

### 4.2 Performance Testing

```powershell
# Test con 100 archivos de 10 MB cada uno
$testFolder = "D:\ImageBackup\Staging"
1..100 | ForEach-Object {
    $file = Join-Path $testFolder "test-$_.dat"
    $stream = [System.IO.File]::Create($file)
    $stream.SetLength(10MB)
    $stream.Close()
}

# Medir tiempo de upload
$startTime = Get-Date
& "D:\ImageBackup\Scripts\Upload-ToAzure.ps1"
$endTime = Get-Date
$duration = ($endTime - $startTime).TotalMinutes

Write-Host "Uploaded 1 GB (100 files) in $duration minutes"
Write-Host "Average: $([math]::Round(1000/$duration, 2)) MB/min"

# Expected: > 50 MB/min (suficiente para 17 GB/día)
```

### 4.3 Monitoring Validation

```powershell
# Verificar logs se están generando
Get-ChildItem "D:\ImageBackup\Logs" | Sort-Object LastWriteTime -Descending | Select-Object -First 5

# Expected: Logs de hoy para Monitor, Upload, Cleanup

# Verificar Task Scheduler
Get-ScheduledTask | Where-Object { $_.TaskName -like "IndustrialBackup-*" } | Format-Table TaskName, State, LastRunTime, NextRunTime

# Expected: Todos en estado "Ready", LastRunTime reciente
```

---

## 5. Rollback Plan

**Scenario:** Deployment falla o causa problemas en planta.

### 5.1 Rollback Steps

```powershell
# 1. Deshabilitar Task Scheduler tasks
Get-ScheduledTask | Where-Object { $_.TaskName -like "IndustrialBackup-*" } | Disable-ScheduledTask

# 2. Detener cualquier proceso de upload en progreso
Get-Process -Name "azcopy" -ErrorAction SilentlyContinue | Stop-Process -Force

# 3. Cerrar share SMB (impedir copias desde OT)
Remove-SmbShare -Name "ImageBackup" -Force

# 4. Notificar a IT Local y Negocio
Send-MailMessage -To "it-local@plant.com,business@plant.com" `
    -From "cloud-team@company.com" `
    -Subject "[ROLLBACK] Industrial Backup Service - Plant EU01" `
    -Body "Service has been rolled back due to issues. Images will not be backed up until further notice."

# 5. Documentar problema en incident ticket
# ServiceNow: Create incident con detalles del problema
```

### 5.2 Rollback Validation

- [ ] Task Scheduler tasks deshabilitadas → verificar `Get-ScheduledTask`
- [ ] Procesos AzCopy detenidos → verificar `Get-Process -Name azcopy`
- [ ] SMB share eliminada → verificar `Get-SmbShare -Name ImageBackup` (debe fallar)
- [ ] Stakeholders notificados → verificar email enviado
- [ ] Incident ticket creado → verificar en ServiceNow

---

## 6. Troubleshooting

### 6.1 Common Issues

| Problema | Causa Probable | Solución |
|----------|---------------|----------|
| **Files not appearing in Staging/** | Script Monitor no se ejecuta | Verificar Task Scheduler: `Get-ScheduledTask IndustrialBackup-Monitor` |
| **Upload fails with "Forbidden"** | Managed Identity sin permisos | Verificar RBAC: `Get-AzRoleAssignment -ObjectId <MI-ObjectId>` |
| **Upload very slow (< 5 MB/min)** | Problema de red/bandwidth | Verificar ExpressRoute: `Test-NetConnection blob.core.windows.net -Port 443` |
| **Files stuck in Staging/** | Script Upload falla silenciosamente | Revisar logs: `D:\ImageBackup\Logs\Upload-*.log` |
| **Local disk full (D:\)** | Cleanup no funciona | Verificar Task Cleanup ejecutándose: `Get-ScheduledTask IndustrialBackup-Cleanup` |
| **OT cannot write to share** | Permisos SMB incorrectos | Verificar `Get-SmbShareAccess -Name ImageBackup` |

### 6.2 Diagnostic Commands

```powershell
# Test 1: Managed Identity authentication
Connect-AzAccount -Identity
Get-AzContext  # Should show VM's Managed Identity

# Test 2: Connectivity to Azure Storage
Test-NetConnection -ComputerName industrialbackupweu.blob.core.windows.net -Port 443

# Test 3: AzCopy version and path
azcopy --version
$env:Path -split ';' | Where-Object { $_ -like "*azcopy*" }

# Test 4: Manual upload test
azcopy copy "D:\ImageBackup\Staging\*" "https://industrialbackupweu.blob.core.windows.net/plant-eu01-images" --check-md5=FailIfDifferent

# Test 5: Check logs for errors
Get-Content "D:\ImageBackup\Logs\Upload-$(Get-Date -Format 'yyyy-MM-dd').log" | Select-String "ERROR"
```

---

## 7. Maintenance

### 7.1 Regular Maintenance Tasks

| Task | Frequency | Owner | Procedure |
|------|-----------|-------|-----------|
| **Review logs for errors** | Weekly | IT Operations | Check `D:\ImageBackup\Logs\` for ERROR entries |
| **Check local disk space** | Weekly | IT Operations | `Get-Volume D` — alert if < 20% free |
| **Verify uploads succeeded** | Daily | IT Operations | Check Azure Portal — files timestamp debe ser < 24h |
| **Test restore from Archive** | Quarterly | Public Cloud | Initiate rehydration and verify integrity |
| **Update PowerShell modules** | Monthly | Public Cloud | `Update-Module Az.Storage, Az.Monitor` |
| **Update AzCopy** | Quarterly | Public Cloud | Download latest from https://aka.ms/downloadazcopy-v10-windows |
| **Review Azure costs** | Monthly | Arquitectura IT | Check cost analysis — alert if > 10% variance |
| **Test DR failover** | Annually | Public Cloud + SRE | Simulate VM failure and rebuild |

### 7.2 Scheduled Maintenance Windows

**Preferred window:** Lunes 02:00-04:00 hora local de cada planta

**Procedure:**
1. Notificar Negocio con 7 días de antelación
2. Deshabilitar Task Scheduler tasks
3. Perform maintenance (update scripts, reboot VM, etc.)
4. Re-enable tasks
5. Validar upload funciona post-mantenimiento
6. Notify Negocio que mantenimiento completó

---

## 8. Contacts

| Role | Name | Email | Phone |
|------|------|-------|-------|
| Service Owner | Architecture Lead | architecture-team@company.com | +XX XXX XXX XXX |
| Tech Lead | Cloud Architect | cloud-team@company.com | +XX XXX XXX XXX |
| On-call (24x7) | IT Operations | it-ops@company.com | +XX XXX XXX XXX |
| Escalation | Cloud Director | cloud-director@company.com | +XX XXX XXX XXX |

**PagerDuty:** https://company.pagerduty.com/services/INDBACK01  
**Slack Channel:** #industrial-backup-support

---

## 9. Change History

| Version | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 1.0 | 2025-02-19 | Public Cloud Team | Versión inicial — procedimiento deployment plantas EU/América |

---

**Notas:**
- Este runbook debe actualizarse cuando se añadan nuevas plantas o cambien procedimientos
- Todas las credenciales (passwords, SAS tokens) deben almacenarse en Azure Key Vault, NO en scripts
- Validar runbook completo al menos 1 vez al año en planta piloto
