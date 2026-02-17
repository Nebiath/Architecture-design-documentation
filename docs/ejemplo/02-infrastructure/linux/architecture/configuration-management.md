# Gestión de Configuración — Ansible

> **Propietario:** @linux-team | **Ansible Core 2.16.x** como herramienta estándar

## Arquitectura
- **Ansible Core:** 2.16.x | **Inventario:** NetBox (dinámico) | **Secrets:** Vault + HashiCorp
- **Control plane:** AWX/Ansible Tower | **Testing:** Molecule + testinfra

## Repositorio Ansible
```
ansible-infra/
├── ansible.cfg
├── inventories/{production,staging,dev}/
├── playbooks/
├── roles/{common,nginx,postgresql,monitoring}/
└── vault/
```

## ansible.cfg Estándar
```ini
[defaults]
inventory = inventories/production/netbox.yml
roles_path = ./roles
forks = 20
host_key_checking = False
callbacks_enabled = profile_tasks, timer

[privilege_escalation]
become = True
become_method = sudo
```

## Rol `common` — Base para Todos los Servidores
```yaml
tasks:
- Set hostname, timezone
- Install base packages (vim, htop, curl, net-tools)
- Configure NTP, monitoring agent, log forwarding
- Apply security hardening
```

## Playbook Execution
```bash
ansible-playbook playbooks/site.yml --check --diff  # Dry-run
ansible-playbook playbooks/site.yml --limit webservers
```

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @linux-lead | Creación inicial |
