# Runbook: Provisioning de Servidores Linux

> **Propietario:** @linux-team | Cloud-init, templates, post-install checklist

## 1. Provisioning en Cloud (Azure/AWS)
```bash
# Ubuntu Cloud Image con cloud-init
# /var/lib/cloud/instance/user-data.txt
#cloud-config
hostname: server01
fqdn: server01.company.com
manage_etc_hosts: true

packages:
  - vim
  - htop
  - curl

users:
  - name: ansible
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAA...

runcmd:
  - apt update && apt upgrade -y
  - systemctl enable ssh
```

## 2. Provisioning en VMware (Template)
```bash
# Crear template base Ubuntu 24.04
# 1. Instalar Ubuntu limpio
# 2. Actualizar: sudo apt update && sudo apt upgrade -y
# 3. Instalar cloud-init: sudo apt install cloud-init -y
# 4. Limpiar: sudo cloud-init clean
# 5. Apagar y convertir a template en vCenter
```

## 3. Post-Install Checklist
```bash
□ Hostname configurado correctamente
□ Timezone configurado (timedatectl set-timezone UTC)
□ NTP sincronizado (timedatectl status)
□ SSH keys configuradas
□ Firewall UFW habilitado
□ Monitoring agent instalado
□ Logs forwarding configurado
□ Servidor unido a AD (si aplica)
□ Ansible inventory actualizado
□ CMDB actualizado
```

## 4. Ansible Playbook de Bootstrap
```yaml
# playbooks/bootstrap.yml
---
- name: Bootstrap new server
  hosts: new_servers
  tasks:
    - name: Set hostname
      hostname:
        name: "{{ inventory_hostname }}"
    
    - name: Join to AD
      command: realm join -U admin company.com
    
    - name: Apply common role
      include_role:
        name: common
```

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
