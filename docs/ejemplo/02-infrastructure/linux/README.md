# Linux — Sistemas Operativos Linux

> **Propietario:** @linux-team @linux-lead  
> **Última actualización:** YYYY-MM-DD  
> **Distribución estándar:** Ubuntu Server 24.04 LTS (Noble Numbat)

Documentación completa del departamento Linux: arquitectura, servicios, seguridad, automatización y procedimientos operacionales.

## 📋 Documentos

**Arquitectura:** [Distrib Standards](architecture/distribution-standards.md) | [Config Mgmt](architecture/configuration-management.md) | [User Auth](architecture/user-authentication.md)

**Servicios:** [Apache](services/apache-architecture.md) | [Nginx](services/nginx-architecture.md) | [Databases](services/database-servers.md)

**Seguridad:** [Hardening](security/hardening-guide.md) | [AppArmor/SELinux](security/selinux-configuration.md) | [Vulnerabilities](security/vulnerability-management.md)

**Automatización:** [Ansible](automation/ansible-playbooks.md) | [Puppet](automation/puppet-manifests.md)

**Runbooks:** [Provisioning](runbooks/server-provisioning.md) | [Package Mgmt](runbooks/package-management.md)

## Stack Tecnológico

- **OS:** Ubuntu Server 24.04 LTS | **Kernel:** 6.8.x | **Init:** systemd 255.x
- **Config Mgmt:** Ansible 2.16.x | **Web:** Nginx 1.24.x / Apache2 2.4.x
- **DB:** PostgreSQL 16.x, MySQL 8.x | **Security:** AppArmor, UFW, Fail2ban

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
