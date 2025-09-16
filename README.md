# PowerDNS Installation Guide

PowerDNS is a free and open-source DNS Server. A modern authoritative DNS server with various backends

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 53 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 53 (default powerdns port)
- **Dependencies**:
  - pdns-backend-mysql, pdns-recursor
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install powerdns
sudo dnf install -y powerdns pdns-backend-mysql, pdns-recursor

# Enable and start service
sudo systemctl enable --now pdns

# Configure firewall
sudo firewall-cmd --permanent --add-service=powerdns
sudo firewall-cmd --reload

# Verify installation
powerdns --version || systemctl status pdns
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install powerdns
sudo apt install -y powerdns pdns-backend-mysql, pdns-recursor

# Enable and start service
sudo systemctl enable --now pdns

# Configure firewall
sudo ufw allow 53

# Verify installation
powerdns --version || systemctl status pdns
```

### Arch Linux

```bash
# Install powerdns
sudo pacman -S powerdns

# Enable and start service
sudo systemctl enable --now pdns

# Verify installation
powerdns --version || systemctl status pdns
```

### Alpine Linux

```bash
# Install powerdns
apk add --no-cache powerdns

# Enable and start service
rc-update add pdns default
rc-service pdns start

# Verify installation
powerdns --version || rc-service pdns status
```

### openSUSE/SLES

```bash
# Install powerdns
sudo zypper install -y powerdns pdns-backend-mysql, pdns-recursor

# Enable and start service
sudo systemctl enable --now pdns

# Configure firewall
sudo firewall-cmd --permanent --add-service=powerdns
sudo firewall-cmd --reload

# Verify installation
powerdns --version || systemctl status pdns
```

### macOS

```bash
# Using Homebrew
brew install powerdns

# Start service
brew services start powerdns

# Verify installation
powerdns --version
```

### FreeBSD

```bash
# Using pkg
pkg install powerdns

# Enable in rc.conf
echo 'pdns_enable="YES"' >> /etc/rc.conf

# Start service
service pdns start

# Verify installation
powerdns --version || service pdns status
```

### Windows

```powershell
# Using Chocolatey
choco install powerdns

# Or using Scoop
scoop install powerdns

# Verify installation
powerdns --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/powerdns

# Set up basic configuration
sudo tee /etc/powerdns/powerdns.conf << 'EOF'
# PowerDNS Configuration
receiver-threads=4, distributor-threads=4
EOF

# Test configuration
sudo powerdns -t || sudo pdns configtest

# Reload service
sudo systemctl reload pdns
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R powerdns:powerdns /etc/powerdns
sudo chmod 750 /etc/powerdns

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable pdns

# Start service
sudo systemctl start pdns

# Stop service
sudo systemctl stop pdns

# Restart service
sudo systemctl restart pdns

# Reload configuration
sudo systemctl reload pdns

# Check status
sudo systemctl status pdns

# View logs
sudo journalctl -u pdns -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add pdns default

# Start service
rc-service pdns start

# Stop service
rc-service pdns stop

# Restart service
rc-service pdns restart

# Check status
rc-service pdns status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'pdns_enable="YES"' >> /etc/rc.conf

# Start service
service pdns start

# Stop service
service pdns stop

# Restart service
service pdns restart

# Check status
service pdns status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start powerdns
brew services stop powerdns
brew services restart powerdns

# Check status
brew services list | grep powerdns
```

### Windows Service Manager

```powershell
# Start service
net start pdns

# Stop service
net stop pdns

# Using PowerShell
Start-Service pdns
Stop-Service pdns
Restart-Service pdns

# Check status
Get-Service pdns
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/powerdns/powerdns.conf << 'EOF'
receiver-threads=4, distributor-threads=4
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart pdns
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream powerdns_backend {
    server 127.0.0.1:53;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name powerdns.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name powerdns.example.com;

    ssl_certificate /etc/ssl/certs/powerdns.example.com.crt;
    ssl_certificate_key /etc/ssl/private/powerdns.example.com.key;

    location / {
        proxy_pass http://powerdns_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName powerdns.example.com
    Redirect permanent / https://powerdns.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName powerdns.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/powerdns.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/powerdns.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:53/
    ProxyPassReverse / http://127.0.0.1:53/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:53/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend powerdns_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/powerdns.pem
    redirect scheme https if !{ ssl_fc }
    default_backend powerdns_backend

backend powerdns_backend
    balance roundrobin
    option httpchk GET /health
    server powerdns1 127.0.0.1:53 check
    server powerdns2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R powerdns:powerdns /etc/powerdns
sudo chmod 750 /etc/powerdns

# Configure firewall
sudo firewall-cmd --permanent --add-service=powerdns
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/powerdns.conf << 'EOF'
[powerdns]
enabled = true
port = 53
filter = powerdns
logpath = /var/log/pdns/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/powerdns.key \
    -out /etc/ssl/certs/powerdns.crt

# Configure SSL in powerdns
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE powerdns_db;
CREATE USER powerdns_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE powerdns_db TO powerdns_user;
EOF

# Configure powerdns to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE powerdns_db;
CREATE USER 'powerdns_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON powerdns_db.* TO 'powerdns_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# PowerDNS specific tuning
receiver-threads=4, distributor-threads=4
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
powerdns soft nofile 65535
powerdns hard nofile 65535
powerdns soft nproc 32768
powerdns hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'powerdns'
    static_configs:
      - targets: ['localhost:53']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet pdns; then
    echo "PowerDNS is running"
    exit 0
else
    echo "PowerDNS is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/powerdns << 'EOF'
/var/log/pdns/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 powerdns powerdns
    postrotate
        systemctl reload pdns > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# PowerDNS backup script
BACKUP_DIR="/backup/powerdns"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop pdns

# Backup configuration
tar -czf "$BACKUP_DIR/powerdns-config-$DATE.tar.gz" /etc/powerdns

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/powerdns-data-$DATE.tar.gz" /var/lib/powerdns

# Start service
systemctl start pdns

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop pdns

# Restore configuration
sudo tar -xzf /backup/powerdns/powerdns-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/powerdns/powerdns-data-*.tar.gz -C /

# Set permissions
sudo chown -R powerdns:powerdns /etc/powerdns
sudo chown -R powerdns:powerdns /var/lib/powerdns

# Start service
sudo systemctl start pdns
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u pdns -n 100
sudo tail -f /var/log/pdns/*.log

# Check configuration
sudo powerdns -t || sudo pdns configtest

# Check permissions
ls -la /etc/powerdns
ls -la /var/lib/powerdns
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 53
sudo netstat -tlnp | grep 53

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 53
nc -zv localhost 53
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep pdns_server)
htop -p $(pgrep pdns_server)

# Check connections
ss -ant | grep :53 | wc -l

# Monitor I/O
iotop -p $(pgrep pdns_server)
```

### Debug Mode

```bash
# Run in debug mode
sudo powerdns -d
# or
sudo pdns debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  powerdns:
    image: powerdns:latest
    container_name: powerdns
    ports:
      - "53:53"
    volumes:
      - ./config:/etc/powerdns
      - ./data:/var/lib/powerdns
    environment:
      - powerdns_CONFIG=/etc/powerdns/powerdns.conf
    restart: unless-stopped
    networks:
      - powerdns_net

networks:
  powerdns_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: powerdns
spec:
  replicas: 3
  selector:
    matchLabels:
      app: powerdns
  template:
    metadata:
      labels:
        app: powerdns
    spec:
      containers:
      - name: powerdns
        image: powerdns:latest
        ports:
        - containerPort: 53
        volumeMounts:
        - name: config
          mountPath: /etc/powerdns
      volumes:
      - name: config
        configMap:
          name: powerdns-config
---
apiVersion: v1
kind: Service
metadata:
  name: powerdns
spec:
  selector:
    app: powerdns
  ports:
  - port: 53
    targetPort: 53
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure PowerDNS
  hosts: all
  become: yes
  tasks:
    - name: Install powerdns
      package:
        name: powerdns
        state: present
    
    - name: Configure powerdns
      template:
        src: powerdns.conf.j2
        dest: /etc/powerdns/powerdns.conf
        owner: powerdns
        group: powerdns
        mode: '0640'
      notify: restart powerdns
    
    - name: Start and enable powerdns
      systemd:
        name: pdns
        state: started
        enabled: yes
  
  handlers:
    - name: restart powerdns
      systemd:
        name: pdns
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update powerdns

# Debian/Ubuntu
sudo apt update && sudo apt upgrade powerdns

# Arch Linux
sudo pacman -Syu powerdns

# Alpine Linux
apk update && apk upgrade powerdns

# openSUSE
sudo zypper update powerdns

# FreeBSD
pkg update && pkg upgrade powerdns

# Always backup before updates
tar -czf /backup/powerdns-pre-update-$(date +%Y%m%d).tar.gz /etc/powerdns

# Restart after updates
sudo systemctl restart pdns
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/pdns -name "*.log" -mtime +30 -delete

# Verify integrity
sudo powerdns --verify || sudo pdns check

# Update databases (if applicable)
sudo powerdns-update-db

# Optimize performance
sudo powerdns-optimize

# Check for security updates
sudo powerdns --security-check
```

## Additional Resources

- Official Documentation: https://docs.powerdns.org/
- GitHub Repository: https://github.com/powerdns/powerdns
- Community Forum: https://forum.powerdns.org/
- Wiki: https://wiki.powerdns.org/
- Comparison vs BIND, Knot DNS, NSD, CoreDNS: https://docs.powerdns.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
