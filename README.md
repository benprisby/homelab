# Homelab

[![Ansible](https://img.shields.io/badge/Ansible-1A1918?logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Docker](https://img.shields.io/badge/Docker-0db7ed?logo=docker&logoColor=white)](https://www.docker.com/)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://github.com/pre-commit/pre-commit)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Ansible playbooks for deploying and managing Docker services across my home server infrastructure.

## 📋 Overview

My homelab is a self-hosted infrastructure playground for running various services both critical and experimental. It
consists of the following servers:

| Name             | Platform        | Role |
| ---------------- | --------------- | ---- |
| `big-slice`      | Intel NUC       | Main infrastructure server running core services |
| `small-slice-01` | Raspberry Pi 4  | Pi cluster node 1 running DNS and monitoring services |
| `small-slice-02` | Raspberry Pi 4  | Pi cluster node 2 running DNS and monitoring services |
| `small-slice-03` | Raspberry Pi 4  | Pi cluster node 3 running DNS and monitoring services |
| `small-slice-04` | Raspberry Pi 4  | Pi cluster node 4 running DNS and monitoring services |
| `nas`            | Synology DS920+ | Network-attached storage for backups and larger files |

The NAS is not managed by these playbooks currently.

## 🖥️ Services

The following are the essential services I am currently running. The NUC runs most of the core services and the Pi
cluster runs distributed services. The `docker_services` variable for each host configuration under `host_vars` defines
which services are deployed on each server.

- **AdGuard Home**: Network-wide DNS filtering on main and IoT VLANs
- **Graylog**: Centralized logging with MongoDB and OpenSearch
- **Prometheus/Grafana + Agents**: Monitoring stack with node and container exporters on each server
- **Nginx Proxy Manager**: Reverse proxy and SSL management
- **Portainer + Agents**: Docker container management for each server
- **Jellyfin**: Home media server
- **Homebridge**: HomeKit integration for non-native smart home devices
- **Docker Proxy**: Secure Docker API access
- **Homepage**: Service dashboard serving as a landing page

The `docker_services` role deploys each of these Compose stacks onto one or more servers.

## 🚀 Getting Started

1. Install requirements:
   ```shell
   ansible-galaxy install -r requirements.yml
   ```

2. Configure secrets (if needed):
   ```shell
   ansible-vault edit group_vars/vault.yml
   ```

3. Run the desired playbook:
   ```shell
   # Full deployment
   ansible-playbook site.yml --ask-vault-pass

   # System updates only
   ansible-playbook system-update.yml --ask-vault-pass
   ```

## 📁 Project Structure

The project aims to follow Ansible best practices in its layout, aiming to be flexible for future changes to my homelab
while not overcomplicating things today.

### Playbooks

- `site.yml`: Configure managed servers and deploy all services
- `system-update.yml`: Update packages and Docker images on managed servers

### Inventory

- `hosts.yml`: Host group definitions and connection information
- `group_vars/all/common.yml`: Global variables common for all servers
- `group_vars/all/vault.yml`: Encrypted secrets (Ansible Vault)
- `host_vars/*.yml`: Host-specific service assignments

### Docker Services

Services are assigned per-host in `host_vars` files, each of which is a Compose stack:

```yaml
docker_services:
  - graylog
  - monitoring
  - portainer
```

Each Compose stack resides under `roles/docker_services/files/<service>`, with any templates under
`roles/docker_services/templates/<service>`. Any files and subdirectories in these locations are merged on the target.

## 🔧 Development

### Git Hooks

```shell
# Install hooks
pre-commit install

# Run manually
pre-commit run --all-files
```

### Testing Changes

```shell
# Lint playbooks for best practices
ansible-lint

# Check syntax
ansible-playbook --syntax-check site.yml

# Dry run
ansible-playbook site.yml --check --ask-vault-pass
```

### Vault Operations

```shell
# Edit encrypted secrets
ansible-vault edit group_vars/all/vault.yml

# View encrypted files without editing
ansible-vault view group_vars/all/vault.yml

# Encrypt a new file
ansible-vault encrypt group_vars/all/vault.yml
```

### Service Management

```shell
# Target specific hosts
ansible-playbook site.yml --limit small-slice-01 --ask-vault-pass
```

### Troubleshooting

```shell
# Test connectivity to all hosts
ansible all -m ping --ask-vault-pass

# Check sudo access
ansible all -m shell -a "whoami" --become --ask-vault-pass

# Verify inventory and variables
ansible-inventory --list --ask-vault-pass

# Debug specific host variables
ansible big-slice -m debug -a "var=docker_services" --ask-vault-pass
```

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
