# Exim Installation Guide

Exim is a free and open-source Mail Server. A message transfer agent (MTA) developed at Cambridge

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
  - Network: 25/587 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 25/587 (default exim port)
- **Dependencies**:
  - exim4-daemon-heavy, sa-exim
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

# Install exim
sudo dnf install -y exim exim4-daemon-heavy, sa-exim

# Enable and start service
sudo systemctl enable --now exim

# Configure firewall
sudo firewall-cmd --permanent --add-service=exim
sudo firewall-cmd --reload

# Verify installation
exim --version || systemctl status exim
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install exim
sudo apt install -y exim exim4-daemon-heavy, sa-exim

# Enable and start service
sudo systemctl enable --now exim

# Configure firewall
sudo ufw allow 25/587

# Verify installation
exim --version || systemctl status exim
```

### Arch Linux

```bash
# Install exim
sudo pacman -S exim

# Enable and start service
sudo systemctl enable --now exim

# Verify installation
exim --version || systemctl status exim
```

### Alpine Linux

```bash
# Install exim
apk add --no-cache exim

# Enable and start service
rc-update add exim default
rc-service exim start

# Verify installation
exim --version || rc-service exim status
```

### openSUSE/SLES

```bash
# Install exim
sudo zypper install -y exim exim4-daemon-heavy, sa-exim

# Enable and start service
sudo systemctl enable --now exim

# Configure firewall
sudo firewall-cmd --permanent --add-service=exim
sudo firewall-cmd --reload

# Verify installation
exim --version || systemctl status exim
```

### macOS

```bash
# Using Homebrew
brew install exim

# Start service
brew services start exim

# Verify installation
exim --version
```

### FreeBSD

```bash
# Using pkg
pkg install exim

# Enable in rc.conf
echo 'exim_enable="YES"' >> /etc/rc.conf

# Start service
service exim start

# Verify installation
exim --version || service exim status
```

### Windows

```powershell
# Using Chocolatey
choco install exim

# Or using Scoop
scoop install exim

# Verify installation
exim --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/exim

# Set up basic configuration
sudo tee /etc/exim/exim.conf << 'EOF'
# Exim Configuration
smtp_accept_max = 100, smtp_accept_max_per_host = 10
EOF

# Test configuration
sudo exim -t || sudo exim configtest

# Reload service
sudo systemctl reload exim
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R exim:exim /etc/exim
sudo chmod 750 /etc/exim

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable exim

# Start service
sudo systemctl start exim

# Stop service
sudo systemctl stop exim

# Restart service
sudo systemctl restart exim

# Reload configuration
sudo systemctl reload exim

# Check status
sudo systemctl status exim

# View logs
sudo journalctl -u exim -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add exim default

# Start service
rc-service exim start

# Stop service
rc-service exim stop

# Restart service
rc-service exim restart

# Check status
rc-service exim status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'exim_enable="YES"' >> /etc/rc.conf

# Start service
service exim start

# Stop service
service exim stop

# Restart service
service exim restart

# Check status
service exim status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start exim
brew services stop exim
brew services restart exim

# Check status
brew services list | grep exim
```

### Windows Service Manager

```powershell
# Start service
net start exim

# Stop service
net stop exim

# Using PowerShell
Start-Service exim
Stop-Service exim
Restart-Service exim

# Check status
Get-Service exim
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/exim/exim.conf << 'EOF'
smtp_accept_max = 100, smtp_accept_max_per_host = 10
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart exim
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
upstream exim_backend {
    server 127.0.0.1:25/587;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name exim.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name exim.example.com;

    ssl_certificate /etc/ssl/certs/exim.example.com.crt;
    ssl_certificate_key /etc/ssl/private/exim.example.com.key;

    location / {
        proxy_pass http://exim_backend;
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
    ServerName exim.example.com
    Redirect permanent / https://exim.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName exim.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/exim.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/exim.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:25/587/
    ProxyPassReverse / http://127.0.0.1:25/587/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:25/587/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend exim_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/exim.pem
    redirect scheme https if !{ ssl_fc }
    default_backend exim_backend

backend exim_backend
    balance roundrobin
    option httpchk GET /health
    server exim1 127.0.0.1:25/587 check
    server exim2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R exim:exim /etc/exim
sudo chmod 750 /etc/exim

# Configure firewall
sudo firewall-cmd --permanent --add-service=exim
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/exim.conf << 'EOF'
[exim]
enabled = true
port = 25/587
filter = exim
logpath = /var/log/exim/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/exim.key \
    -out /etc/ssl/certs/exim.crt

# Configure SSL in exim
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE exim_db;
CREATE USER exim_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE exim_db TO exim_user;
EOF

# Configure exim to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE exim_db;
CREATE USER 'exim_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON exim_db.* TO 'exim_user'@'localhost';
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

# Exim specific tuning
smtp_accept_max = 100, smtp_accept_max_per_host = 10
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
exim soft nofile 65535
exim hard nofile 65535
exim soft nproc 32768
exim hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'exim'
    static_configs:
      - targets: ['localhost:25/587']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet exim; then
    echo "Exim is running"
    exit 0
else
    echo "Exim is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/exim << 'EOF'
/var/log/exim/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 exim exim
    postrotate
        systemctl reload exim > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Exim backup script
BACKUP_DIR="/backup/exim"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop exim

# Backup configuration
tar -czf "$BACKUP_DIR/exim-config-$DATE.tar.gz" /etc/exim

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/exim-data-$DATE.tar.gz" /var/lib/exim

# Start service
systemctl start exim

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop exim

# Restore configuration
sudo tar -xzf /backup/exim/exim-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/exim/exim-data-*.tar.gz -C /

# Set permissions
sudo chown -R exim:exim /etc/exim
sudo chown -R exim:exim /var/lib/exim

# Start service
sudo systemctl start exim
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u exim -n 100
sudo tail -f /var/log/exim/*.log

# Check configuration
sudo exim -t || sudo exim configtest

# Check permissions
ls -la /etc/exim
ls -la /var/lib/exim
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 25/587
sudo netstat -tlnp | grep 25/587

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 25/587
nc -zv localhost 25/587
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep exim4)
htop -p $(pgrep exim4)

# Check connections
ss -ant | grep :25/587 | wc -l

# Monitor I/O
iotop -p $(pgrep exim4)
```

### Debug Mode

```bash
# Run in debug mode
sudo exim -d
# or
sudo exim debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  exim:
    image: exim:latest
    container_name: exim
    ports:
      - "25/587:25/587"
    volumes:
      - ./config:/etc/exim
      - ./data:/var/lib/exim
    environment:
      - exim_CONFIG=/etc/exim/exim.conf
    restart: unless-stopped
    networks:
      - exim_net

networks:
  exim_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exim
spec:
  replicas: 3
  selector:
    matchLabels:
      app: exim
  template:
    metadata:
      labels:
        app: exim
    spec:
      containers:
      - name: exim
        image: exim:latest
        ports:
        - containerPort: 25/587
        volumeMounts:
        - name: config
          mountPath: /etc/exim
      volumes:
      - name: config
        configMap:
          name: exim-config
---
apiVersion: v1
kind: Service
metadata:
  name: exim
spec:
  selector:
    app: exim
  ports:
  - port: 25/587
    targetPort: 25/587
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Exim
  hosts: all
  become: yes
  tasks:
    - name: Install exim
      package:
        name: exim
        state: present
    
    - name: Configure exim
      template:
        src: exim.conf.j2
        dest: /etc/exim/exim.conf
        owner: exim
        group: exim
        mode: '0640'
      notify: restart exim
    
    - name: Start and enable exim
      systemd:
        name: exim
        state: started
        enabled: yes
  
  handlers:
    - name: restart exim
      systemd:
        name: exim
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update exim

# Debian/Ubuntu
sudo apt update && sudo apt upgrade exim

# Arch Linux
sudo pacman -Syu exim

# Alpine Linux
apk update && apk upgrade exim

# openSUSE
sudo zypper update exim

# FreeBSD
pkg update && pkg upgrade exim

# Always backup before updates
tar -czf /backup/exim-pre-update-$(date +%Y%m%d).tar.gz /etc/exim

# Restart after updates
sudo systemctl restart exim
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/exim -name "*.log" -mtime +30 -delete

# Verify integrity
sudo exim --verify || sudo exim check

# Update databases (if applicable)
sudo exim-update-db

# Optimize performance
sudo exim-optimize

# Check for security updates
sudo exim --security-check
```

## Additional Resources

- Official Documentation: https://docs.exim.org/
- GitHub Repository: https://github.com/exim/exim
- Community Forum: https://forum.exim.org/
- Wiki: https://wiki.exim.org/
- Comparison vs Postfix, Sendmail, OpenSMTPD, Qmail: https://docs.exim.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
