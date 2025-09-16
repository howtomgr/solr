# solr Installation Guide

solr is a free and open-source search platform. Solr provides powerful full-text search and near real-time indexing

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
  - CPU: 2+ cores
  - RAM: 2GB minimum (8GB+ recommended)
  - Storage: 20GB+ for indices
  - Network: HTTP/REST API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8983 (default solr port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
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

# Install solr
sudo dnf install -y solr

# Enable and start service
sudo systemctl enable --now solr

# Configure firewall
sudo firewall-cmd --permanent --add-port=8983/tcp
sudo firewall-cmd --reload

# Verify installation
solr --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install solr
sudo apt install -y solr

# Enable and start service
sudo systemctl enable --now solr

# Configure firewall
sudo ufw allow 8983

# Verify installation
solr --version
```

### Arch Linux

```bash
# Install solr
sudo pacman -S solr

# Enable and start service
sudo systemctl enable --now solr

# Verify installation
solr --version
```

### Alpine Linux

```bash
# Install solr
apk add --no-cache solr

# Enable and start service
rc-update add solr default
rc-service solr start

# Verify installation
solr --version
```

### openSUSE/SLES

```bash
# Install solr
sudo zypper install -y solr

# Enable and start service
sudo systemctl enable --now solr

# Configure firewall
sudo firewall-cmd --permanent --add-port=8983/tcp
sudo firewall-cmd --reload

# Verify installation
solr --version
```

### macOS

```bash
# Using Homebrew
brew install solr

# Start service
brew services start solr

# Verify installation
solr --version
```

### FreeBSD

```bash
# Using pkg
pkg install solr

# Enable in rc.conf
echo 'solr_enable="YES"' >> /etc/rc.conf

# Start service
service solr start

# Verify installation
solr --version
```

### Windows

```bash
# Using Chocolatey
choco install solr

# Or using Scoop
scoop install solr

# Verify installation
solr --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/solr

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
solr --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable solr

# Start service
sudo systemctl start solr

# Stop service
sudo systemctl stop solr

# Restart service
sudo systemctl restart solr

# Check status
sudo systemctl status solr

# View logs
sudo journalctl -u solr -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add solr default

# Start service
rc-service solr start

# Stop service
rc-service solr stop

# Restart service
rc-service solr restart

# Check status
rc-service solr status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'solr_enable="YES"' >> /etc/rc.conf

# Start service
service solr start

# Stop service
service solr stop

# Restart service
service solr restart

# Check status
service solr status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start solr
brew services stop solr
brew services restart solr

# Check status
brew services list | grep solr
```

### Windows Service Manager

```powershell
# Start service
net start solr

# Stop service
net stop solr

# Using PowerShell
Start-Service solr
Stop-Service solr
Restart-Service solr

# Check status
Get-Service solr
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream solr_backend {
    server 127.0.0.1:8983;
}

server {
    listen 80;
    server_name solr.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name solr.example.com;

    ssl_certificate /etc/ssl/certs/solr.example.com.crt;
    ssl_certificate_key /etc/ssl/private/solr.example.com.key;

    location / {
        proxy_pass http://solr_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName solr.example.com
    Redirect permanent / https://solr.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName solr.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/solr.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/solr.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8983/
    ProxyPassReverse / http://127.0.0.1:8983/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend solr_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/solr.pem
    redirect scheme https if !{ ssl_fc }
    default_backend solr_backend

backend solr_backend
    balance roundrobin
    server solr1 127.0.0.1:8983 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R solr:solr /etc/solr
sudo chmod 750 /etc/solr

# Configure firewall
sudo firewall-cmd --permanent --add-port=8983/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status solr

# View logs
sudo journalctl -u solr -f

# Monitor resource usage
top -p $(pgrep solr)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/solr"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/solr-backup-$DATE.tar.gz" /etc/solr /var/lib/solr

echo "Backup completed: $BACKUP_DIR/solr-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop solr

# Restore from backup
tar -xzf /backup/solr/solr-backup-*.tar.gz -C /

# Start service
sudo systemctl start solr
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u solr -n 100
sudo tail -f /var/log/solr/solr.log

# Check configuration
solr --version

# Check permissions
ls -la /etc/solr
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8983

# Test connectivity
telnet localhost 8983

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep solr)

# Check disk I/O
iotop -p $(pgrep solr)

# Check connections
ss -an | grep 8983
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  solr:
    image: solr:latest
    ports:
      - "8983:8983"
    volumes:
      - ./config:/etc/solr
      - ./data:/var/lib/solr
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update solr

# Debian/Ubuntu
sudo apt update && sudo apt upgrade solr

# Arch Linux
sudo pacman -Syu solr

# Alpine Linux
apk update && apk upgrade solr

# openSUSE
sudo zypper update solr

# FreeBSD
pkg update && pkg upgrade solr

# Always backup before updates
tar -czf /backup/solr-pre-update-$(date +%Y%m%d).tar.gz /etc/solr

# Restart after updates
sudo systemctl restart solr
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/solr

# Clean old logs
find /var/log/solr -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/solr
```

## Additional Resources

- Official Documentation: https://docs.solr.org/
- GitHub Repository: https://github.com/solr/solr
- Community Forum: https://forum.solr.org/
- Best Practices Guide: https://docs.solr.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
