# Autenticación de Usuarios — SSSD + AD

> **Propietario:** @linux-team | **SSSD + Active Directory** como backend de autenticación

## Stack
- **Auth:** SSSD 2.9.x | **Backend:** Active Directory | **Sudo:** /etc/sudoers.d + AD groups
- **SSH:** Keys preferidos | **MFA:** Google Authenticator PAM

## SSSD + AD
```bash
# Unir al dominio
sudo apt install sssd-ad realmd adcli krb5-user -y
sudo realm join -U admin@COMPANY.COM company.com

# Verificar
realm list
id usuario@company.com
```

## /etc/sssd/sssd.conf
```ini
[sssd]
domains = company.com
services = nss, pam, ssh

[domain/company.com]
ad_domain = company.com
id_provider = ad
use_fully_qualified_names = False
fallback_homedir = /home/%u
access_provider = ad
```

## Grupos AD para Linux
| Grupo | Sudo | Descripción |
|-------|------|-------------|
| Linux-Admins | Full | Administradores |
| Linux-Users | No | Usuarios estándar |
| Linux-Developers | Limitado | `systemctl`, `journalctl`, logs |

## sudo Policy
```bash
# /etc/sudoers.d/ad-groups
%Linux-Admins ALL=(ALL:ALL) ALL
%Linux-Developers ALL=(ALL) NOPASSWD: /bin/systemctl restart *, /bin/journalctl
```

## SSH
```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys
```

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
