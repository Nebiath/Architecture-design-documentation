# Guía de Hardening Ubuntu 24.04 LTS

> **Propietario:** @linux-team @security-team | **CIS Benchmark Ubuntu 24.04**

## 1. Actualizaciones Automáticas
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## 2. Firewall UFW
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

## 3. SSH Hardening
```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
X11Forwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 0
Protocol 2
```

## 4. Fail2ban
```bash
sudo apt install fail2ban -y
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 3
bantime = 3600
```

## 5. AppArmor
```bash
sudo systemctl enable apparmor
sudo systemctl start apparmor
sudo aa-status  # Ver perfiles activos
```

## 6. Auditoría con auditd
```bash
sudo apt install auditd -y
sudo systemctl enable --now auditd
```

## 7. Deshabilitar Servicios Innecesarios
```bash
sudo systemctl disable bluetooth
sudo systemctl disable cups
```

## 8. Kernel Hardening (sysctl)
```bash
# /etc/sysctl.d/99-hardening.conf
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
