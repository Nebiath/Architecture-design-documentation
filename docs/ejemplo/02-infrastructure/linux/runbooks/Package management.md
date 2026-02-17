# Runbook: Gestión de Paquetes APT

> **Propietario:** @linux-team | APT, PPAs, updates, kernel upgrades

## 1. Operaciones Básicas APT
```bash
# Actualizar lista de paquetes
sudo apt update

# Actualizar todos los paquetes
sudo apt upgrade -y

# Full-upgrade (puede remover paquetes)
sudo apt full-upgrade -y

# Instalar paquete
sudo apt install nginx -y

# Desinstalar paquete
sudo apt remove nginx
sudo apt purge nginx  # Incluye archivos de configuración

# Autoremove (limpiar dependencias huérfanas)
sudo apt autoremove -y
```

## 2. Búsqueda de Paquetes
```bash
# Buscar paquete
apt search nginx

# Info de paquete
apt show nginx

# Ver qué paquetes están instalados
apt list --installed

# Ver updates disponibles
apt list --upgradable
```

## 3. PPAs (Personal Package Archives)
```bash
# Añadir PPA
sudo add-apt-repository ppa:nginx/stable
sudo apt update

# Remover PPA
sudo add-apt-repository --remove ppa:nginx/stable
```

## 4. Kernel Upgrades
```bash
# Ver kernel actual
uname -r

# Instalar HWE kernel (hardware enablement)
sudo apt install linux-generic-hwe-24.04 -y

# Listar kernels instalados
dpkg --list | grep linux-image

# Remover kernels antiguos
sudo apt autoremove --purge
```

## 5. Unattended Upgrades
```bash
# Configurar actualizaciones automáticas
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades

# Config en /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot "false";
```

## 6. Hold de Paquetes
```bash
# Prevenir que un paquete se actualice
sudo apt-mark hold nginx

# Ver paquetes en hold
apt-mark showhold

# Deshacer hold
sudo apt-mark unhold nginx
```

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
