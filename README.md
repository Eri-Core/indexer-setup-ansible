# Ansible Server Setup

Simple Ansible setup for Nginx SSL, ClickHouse, Prometheus, and Grafana with full monitoring.

## Quick Start

1. **Configure inventory** in `inventory/hosts`:
```ini
[servers]
server1 ansible_host=YOUR_SERVER_IP ansible_user=ubuntu

[nginx_servers]
server1

[clickhouse_servers]
server1

[prometheus_servers]
server1

[grafana_servers]
server1
```

2. **Edit configuration** in `group_vars/all.yml`:
```yaml
indexer_domain: "yourdomain.com"
admin_email: "admin@yourdomain.com"
clickhouse_password: "secure-password"
grafana_admin_password: "secure-password"
```

3. **Run playbook**:
```bash
ansible-playbook site.yml
```

## Command Line Parameters

You can pass variables directly when running the playbook instead of editing `group_vars/all.yml`:

### Basic Usage
```bash
ansible-playbook site.yml --extra-vars "indexer_domain=mydomain.com admin_email=admin@mydomain.com clickhouse_password=mypass123 grafana_admin_password=grafana123"
```

### JSON Format
```bash
ansible-playbook site.yml --extra-vars '{"indexer_domain":"mydomain.com","admin_email":"admin@mydomain.com","clickhouse_password":"mypass123","grafana_admin_password":"grafana123"}'
```

### From Variables File
```bash
# Create custom-vars.yml with your values
ansible-playbook site.yml --extra-vars "@custom-vars.yml"
```

### Using Environment Variables
```bash
export INDEXER_DOMAIN="mydomain.com"
export ADMIN_EMAIL="admin@mydomain.com"
export CLICKHOUSE_PASSWORD="mypass123"
export GRAFANA_ADMIN_PASSWORD="grafana123"

ansible-playbook site.yml --extra-vars "indexer_domain=${INDEXER_DOMAIN} admin_email=${ADMIN_EMAIL} clickhouse_password=${CLICKHOUSE_PASSWORD} grafana_admin_password=${GRAFANA_ADMIN_PASSWORD}"
```

**Note**: Command line variables override values in `group_vars/all.yml`

## Services

### Nginx (Port 443)
- HTTPS with automatic SSL certificates via Let's Encrypt
- Nginx Prometheus Exporter for metrics collection

### ClickHouse (Port 8123)
- Database name: `ericore`
- Password authentication for default user
- Prometheus metrics endpoint enabled

### Prometheus (Port 9090)
- Collects metrics from:
  - ClickHouse (port 9363)
  - Nginx Exporter (port 9113)
- 15-second scrape interval

### Grafana (Port 3000)
- Pre-configured dashboards:
  - **ClickHouse**: Query rate, memory usage, connections, query duration
  - **Nginx**: Request rate, active connections, connection stats
- Prometheus datasource auto-configured

## Access

- **Grafana**: `https://yourdomain.com/grafana/` (admin/your-password)
- **ClickHouse**: `http://server-ip:8123` (default/your-password)
- **Prometheus**: `http://server-ip:9090`

## Monitoring

Both Nginx and ClickHouse send metrics to Prometheus, which are visualized in Grafana:

- **Nginx Metrics**: Request rates, connection states, response times
- **ClickHouse Metrics**: Query performance, memory usage, active connections

## Security Notes

- Change default passwords in `group_vars/all.yml` before deployment
- SSL certificates auto-renew via certbot
- All services are configured with systemd