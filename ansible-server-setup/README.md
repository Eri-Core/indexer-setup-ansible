# Ansible Server Setup

This Ansible project automates the deployment and configuration of Debian/Ubuntu servers with nginx, ClickHouse, Prometheus, Grafana, SSL certificates, and custom binaries.

## Project Structure

```
ansible-server-setup/
├── ansible.cfg          # Ansible configuration
├── inventory/          
│   └── hosts           # Server inventory
├── group_vars/
│   └── all.yml         # All configuration variables
├── roles/
│   ├── common/         # System updates and base dependencies
│   ├── nginx/          # Nginx load balancer
│   ├── clickhouse/     # ClickHouse database
│   ├── prometheus/     # Prometheus monitoring
│   ├── grafana/        # Grafana dashboards
│   ├── certs/          # SSL certificates with certbot
│   └── binaries/       # Custom binary deployment
└── site.yml            # Main playbook
```

## Quick Start

1. **Configure your inventory** in `inventory/hosts`:
```ini
[servers]
server1 ansible_host=192.168.1.10 ansible_user=ubuntu

[nginx_servers]
server1

[clickhouse_servers]
server1

[prometheus_servers]
server1

[grafana_servers]
server1

[indexer_servers]
server1
```

2. **Edit configuration** in `group_vars/all.yml`:
```yaml
indexer_domain: "yourdomain.com"
admin_email: "admin@yourdomain.com"
grafana_admin_password: "secure-password"

custom_binaries:
  - name: "your-service"
    url: "https://example.com/downloads/your-service"
    # ... other settings
```

3. **Run the playbook**:
```bash
# Full deployment
ansible-playbook site.yml

# Deploy specific roles
ansible-playbook site.yml --tags common
ansible-playbook site.yml --limit nginx_servers
```

## Configuration

All configuration is centralized in `group_vars/all.yml`:

### Key Variables

- **Domain & SSL**:
  - `indexer_domain`: Your domain name
  - `admin_email`: Email for SSL certificates
  - `certbot_*`: SSL certificate settings

- **Service Ports**:
  - `nginx_port`: 443 (HTTPS)
  - `clickhouse_http_port`: 8123
  - `prometheus_port`: 9090
  - `grafana_port`: 3000

- **Custom Binaries**:
  - Configure multiple binaries with different instances
  - Each instance can have its own port and config

## Roles

### common
- Updates system packages
- Installs base dependencies (curl, wget, python3, etc.)
- Gathers and displays server IP addresses

### nginx
- Installs nginx as a load balancer
- Configures SSL termination
- Opens port 443

### clickhouse
- Installs ClickHouse database
- Configures storage and logging
- Exposes metrics for Prometheus

### prometheus
- Installs Prometheus monitoring
- Auto-configures scraping for all services
- Configurable retention and intervals

### grafana
- Installs Grafana
- Pre-configures Prometheus datasource
- Accessible at https://yourdomain.com/grafana/

### certs
- Auto-generates SSL certificates with Let's Encrypt
- Configures automatic renewal
- Supports standalone and nginx methods

### binaries
- Downloads and installs custom binaries
- Creates systemd services
- Supports multiple instances per binary
- Configurable ports and environment variables

## Usage Examples

### Deploy Everything
```bash
ansible-playbook site.yml
```

### Deploy Only Monitoring Stack
```bash
ansible-playbook site.yml --limit prometheus_servers,grafana_servers
```

### Update SSL Certificates
```bash
ansible-playbook site.yml --tags certs
```

### Check Configuration
```bash
ansible-playbook site.yml --check
```

## Security Notes

- Change default passwords in `group_vars/all.yml`
- SSL certificates are automatically renewed
- Services are configured with systemd security features
- Firewall rules are automatically configured

## Troubleshooting

1. **SSL Certificate Issues**:
   - Ensure domain DNS points to your server
   - Check port 80 is accessible for cert validation

2. **Service Connection Issues**:
   - Verify firewall rules with `sudo ufw status`
   - Check service logs with `sudo journalctl -u service-name`

3. **Ansible Connection Issues**:
   - Verify SSH access: `ssh ubuntu@server-ip`
   - Check inventory configuration