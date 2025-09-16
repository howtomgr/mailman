# Mailman Installation Guide

Mailman is a free and open-source Mailing List. Free software for managing electronic mail discussion lists

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
  - Network: 80/443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default mailman port)
- **Dependencies**:
  - python3, postfix
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

# Install mailman
sudo dnf install -y mailman python3, postfix

# Enable and start service
sudo systemctl enable --now mailman

# Configure firewall
sudo firewall-cmd --permanent --add-service=mailman
sudo firewall-cmd --reload

# Verify installation
mailman --version || systemctl status mailman
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install mailman
sudo apt install -y mailman python3, postfix

# Enable and start service
sudo systemctl enable --now mailman

# Configure firewall
sudo ufw allow 80/443

# Verify installation
mailman --version || systemctl status mailman
```

### Arch Linux

```bash
# Install mailman
sudo pacman -S mailman

# Enable and start service
sudo systemctl enable --now mailman

# Verify installation
mailman --version || systemctl status mailman
```

### Alpine Linux

```bash
# Install mailman
apk add --no-cache mailman

# Enable and start service
rc-update add mailman default
rc-service mailman start

# Verify installation
mailman --version || rc-service mailman status
```

### openSUSE/SLES

```bash
# Install mailman
sudo zypper install -y mailman python3, postfix

# Enable and start service
sudo systemctl enable --now mailman

# Configure firewall
sudo firewall-cmd --permanent --add-service=mailman
sudo firewall-cmd --reload

# Verify installation
mailman --version || systemctl status mailman
```

### macOS

```bash
# Using Homebrew
brew install mailman

# Start service
brew services start mailman

# Verify installation
mailman --version
```

### FreeBSD

```bash
# Using pkg
pkg install mailman

# Enable in rc.conf
echo 'mailman_enable="YES"' >> /etc/rc.conf

# Start service
service mailman start

# Verify installation
mailman --version || service mailman status
```

### Windows

```powershell
# Using Chocolatey
choco install mailman

# Or using Scoop
scoop install mailman

# Verify installation
mailman --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/mailman

# Set up basic configuration
sudo tee /etc/mailman/mailman.conf << 'EOF'
# Mailman Configuration
QRUNNER_SLEEP_TIME = 1
EOF

# Test configuration
sudo mailman -t || sudo mailman configtest

# Reload service
sudo systemctl reload mailman
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R mailman:mailman /etc/mailman
sudo chmod 750 /etc/mailman

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable mailman

# Start service
sudo systemctl start mailman

# Stop service
sudo systemctl stop mailman

# Restart service
sudo systemctl restart mailman

# Reload configuration
sudo systemctl reload mailman

# Check status
sudo systemctl status mailman

# View logs
sudo journalctl -u mailman -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add mailman default

# Start service
rc-service mailman start

# Stop service
rc-service mailman stop

# Restart service
rc-service mailman restart

# Check status
rc-service mailman status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'mailman_enable="YES"' >> /etc/rc.conf

# Start service
service mailman start

# Stop service
service mailman stop

# Restart service
service mailman restart

# Check status
service mailman status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start mailman
brew services stop mailman
brew services restart mailman

# Check status
brew services list | grep mailman
```

### Windows Service Manager

```powershell
# Start service
net start mailman

# Stop service
net stop mailman

# Using PowerShell
Start-Service mailman
Stop-Service mailman
Restart-Service mailman

# Check status
Get-Service mailman
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/mailman/mailman.conf << 'EOF'
QRUNNER_SLEEP_TIME = 1
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart mailman
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
upstream mailman_backend {
    server 127.0.0.1:80/443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name mailman.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name mailman.example.com;

    ssl_certificate /etc/ssl/certs/mailman.example.com.crt;
    ssl_certificate_key /etc/ssl/private/mailman.example.com.key;

    location / {
        proxy_pass http://mailman_backend;
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
    ServerName mailman.example.com
    Redirect permanent / https://mailman.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName mailman.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/mailman.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/mailman.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:80/443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend mailman_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/mailman.pem
    redirect scheme https if !{ ssl_fc }
    default_backend mailman_backend

backend mailman_backend
    balance roundrobin
    option httpchk GET /health
    server mailman1 127.0.0.1:80/443 check
    server mailman2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R mailman:mailman /etc/mailman
sudo chmod 750 /etc/mailman

# Configure firewall
sudo firewall-cmd --permanent --add-service=mailman
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/mailman.conf << 'EOF'
[mailman]
enabled = true
port = 80/443
filter = mailman
logpath = /var/log/mailman/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/mailman.key \
    -out /etc/ssl/certs/mailman.crt

# Configure SSL in mailman
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE mailman_db;
CREATE USER mailman_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE mailman_db TO mailman_user;
EOF

# Configure mailman to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE mailman_db;
CREATE USER 'mailman_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON mailman_db.* TO 'mailman_user'@'localhost';
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

# Mailman specific tuning
QRUNNER_SLEEP_TIME = 1
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
mailman soft nofile 65535
mailman hard nofile 65535
mailman soft nproc 32768
mailman hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'mailman'
    static_configs:
      - targets: ['localhost:80/443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet mailman; then
    echo "Mailman is running"
    exit 0
else
    echo "Mailman is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/mailman << 'EOF'
/var/log/mailman/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 mailman mailman
    postrotate
        systemctl reload mailman > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Mailman backup script
BACKUP_DIR="/backup/mailman"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop mailman

# Backup configuration
tar -czf "$BACKUP_DIR/mailman-config-$DATE.tar.gz" /etc/mailman

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/mailman-data-$DATE.tar.gz" /var/lib/mailman

# Start service
systemctl start mailman

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop mailman

# Restore configuration
sudo tar -xzf /backup/mailman/mailman-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/mailman/mailman-data-*.tar.gz -C /

# Set permissions
sudo chown -R mailman:mailman /etc/mailman
sudo chown -R mailman:mailman /var/lib/mailman

# Start service
sudo systemctl start mailman
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u mailman -n 100
sudo tail -f /var/log/mailman/*.log

# Check configuration
sudo mailman -t || sudo mailman configtest

# Check permissions
ls -la /etc/mailman
ls -la /var/lib/mailman
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443
sudo netstat -tlnp | grep 80/443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 80/443
nc -zv localhost 80/443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep mailmanctl)
htop -p $(pgrep mailmanctl)

# Check connections
ss -ant | grep :80/443 | wc -l

# Monitor I/O
iotop -p $(pgrep mailmanctl)
```

### Debug Mode

```bash
# Run in debug mode
sudo mailman -d
# or
sudo mailman debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  mailman:
    image: mailman:latest
    container_name: mailman
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/mailman
      - ./data:/var/lib/mailman
    environment:
      - mailman_CONFIG=/etc/mailman/mailman.conf
    restart: unless-stopped
    networks:
      - mailman_net

networks:
  mailman_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mailman
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mailman
  template:
    metadata:
      labels:
        app: mailman
    spec:
      containers:
      - name: mailman
        image: mailman:latest
        ports:
        - containerPort: 80/443
        volumeMounts:
        - name: config
          mountPath: /etc/mailman
      volumes:
      - name: config
        configMap:
          name: mailman-config
---
apiVersion: v1
kind: Service
metadata:
  name: mailman
spec:
  selector:
    app: mailman
  ports:
  - port: 80/443
    targetPort: 80/443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Mailman
  hosts: all
  become: yes
  tasks:
    - name: Install mailman
      package:
        name: mailman
        state: present
    
    - name: Configure mailman
      template:
        src: mailman.conf.j2
        dest: /etc/mailman/mailman.conf
        owner: mailman
        group: mailman
        mode: '0640'
      notify: restart mailman
    
    - name: Start and enable mailman
      systemd:
        name: mailman
        state: started
        enabled: yes
  
  handlers:
    - name: restart mailman
      systemd:
        name: mailman
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update mailman

# Debian/Ubuntu
sudo apt update && sudo apt upgrade mailman

# Arch Linux
sudo pacman -Syu mailman

# Alpine Linux
apk update && apk upgrade mailman

# openSUSE
sudo zypper update mailman

# FreeBSD
pkg update && pkg upgrade mailman

# Always backup before updates
tar -czf /backup/mailman-pre-update-$(date +%Y%m%d).tar.gz /etc/mailman

# Restart after updates
sudo systemctl restart mailman
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/mailman -name "*.log" -mtime +30 -delete

# Verify integrity
sudo mailman --verify || sudo mailman check

# Update databases (if applicable)
sudo mailman-update-db

# Optimize performance
sudo mailman-optimize

# Check for security updates
sudo mailman --security-check
```

## Additional Resources

- Official Documentation: https://docs.mailman.org/
- GitHub Repository: https://github.com/mailman/mailman
- Community Forum: https://forum.mailman.org/
- Wiki: https://wiki.mailman.org/
- Comparison vs Sympa, Listmonk, phpList, Majordomo: https://docs.mailman.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
