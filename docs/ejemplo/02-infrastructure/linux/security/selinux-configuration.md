# AppArmor / SELinux en Ubuntu 24.04

> **Propietario:** @linux-team @security-team | AppArmor por defecto, SELinux opcional

## AppArmor (Por Defecto en Ubuntu)
```bash
# Verificar estado
sudo aa-status

# Ver perfiles
ls /etc/apparmor.d/

# Enforcing vs Complain mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx  # Enforce
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx  # Complain (audit only)

# Ver logs
sudo grep apparmor /var/log/syslog
```

## SELinux (Alternativa, si se requiere)
```bash
# Instalar SELinux en Ubuntu (no recomendado por defecto)
sudo apt install selinux-basics selinux-policy-default auditd -y
sudo selinux-activate
sudo reboot

# Verificar modo
getenforce

# Cambiar a permissive (testing)
sudo setenforce 0

# Cambiar a enforcing
sudo setenforce 1
```

## Política: AppArmor Preferido
- Ubuntu viene con AppArmor por defecto
- SELinux solo si requisito específico de compliance/aplicación
- No mezclar AppArmor y SELinux en el mismo servidor

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
