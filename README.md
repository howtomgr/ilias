# ilias Installation Guide

ilias is a free and open-source learning management. ILIAS provides comprehensive learning management system

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
  - RAM: 4GB minimum
  - Storage: 20GB for data
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default ilias port)
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

# Install ilias
sudo dnf install -y ilias

# Enable and start service
sudo systemctl enable --now ilias

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
ilias --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ilias
sudo apt install -y ilias

# Enable and start service
sudo systemctl enable --now ilias

# Configure firewall
sudo ufw allow 80

# Verify installation
ilias --version
```

### Arch Linux

```bash
# Install ilias
sudo pacman -S ilias

# Enable and start service
sudo systemctl enable --now ilias

# Verify installation
ilias --version
```

### Alpine Linux

```bash
# Install ilias
apk add --no-cache ilias

# Enable and start service
rc-update add ilias default
rc-service ilias start

# Verify installation
ilias --version
```

### openSUSE/SLES

```bash
# Install ilias
sudo zypper install -y ilias

# Enable and start service
sudo systemctl enable --now ilias

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
ilias --version
```

### macOS

```bash
# Using Homebrew
brew install ilias

# Start service
brew services start ilias

# Verify installation
ilias --version
```

### FreeBSD

```bash
# Using pkg
pkg install ilias

# Enable in rc.conf
echo 'ilias_enable="YES"' >> /etc/rc.conf

# Start service
service ilias start

# Verify installation
ilias --version
```

### Windows

```bash
# Using Chocolatey
choco install ilias

# Or using Scoop
scoop install ilias

# Verify installation
ilias --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ilias

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ilias --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ilias

# Start service
sudo systemctl start ilias

# Stop service
sudo systemctl stop ilias

# Restart service
sudo systemctl restart ilias

# Check status
sudo systemctl status ilias

# View logs
sudo journalctl -u ilias -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ilias default

# Start service
rc-service ilias start

# Stop service
rc-service ilias stop

# Restart service
rc-service ilias restart

# Check status
rc-service ilias status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ilias_enable="YES"' >> /etc/rc.conf

# Start service
service ilias start

# Stop service
service ilias stop

# Restart service
service ilias restart

# Check status
service ilias status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ilias
brew services stop ilias
brew services restart ilias

# Check status
brew services list | grep ilias
```

### Windows Service Manager

```powershell
# Start service
net start ilias

# Stop service
net stop ilias

# Using PowerShell
Start-Service ilias
Stop-Service ilias
Restart-Service ilias

# Check status
Get-Service ilias
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ilias_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name ilias.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ilias.example.com;

    ssl_certificate /etc/ssl/certs/ilias.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ilias.example.com.key;

    location / {
        proxy_pass http://ilias_backend;
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
    ServerName ilias.example.com
    Redirect permanent / https://ilias.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ilias.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ilias.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ilias.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ilias_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ilias.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ilias_backend

backend ilias_backend
    balance roundrobin
    server ilias1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ilias:ilias /etc/ilias
sudo chmod 750 /etc/ilias

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status ilias

# View logs
sudo journalctl -u ilias -f

# Monitor resource usage
top -p $(pgrep ilias)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ilias"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ilias-backup-$DATE.tar.gz" /etc/ilias /var/lib/ilias

echo "Backup completed: $BACKUP_DIR/ilias-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ilias

# Restore from backup
tar -xzf /backup/ilias/ilias-backup-*.tar.gz -C /

# Start service
sudo systemctl start ilias
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ilias -n 100
sudo tail -f /var/log/ilias/ilias.log

# Check configuration
ilias --version

# Check permissions
ls -la /etc/ilias
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ilias)

# Check disk I/O
iotop -p $(pgrep ilias)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ilias:
    image: ilias:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/ilias
      - ./data:/var/lib/ilias
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ilias

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ilias

# Arch Linux
sudo pacman -Syu ilias

# Alpine Linux
apk update && apk upgrade ilias

# openSUSE
sudo zypper update ilias

# FreeBSD
pkg update && pkg upgrade ilias

# Always backup before updates
tar -czf /backup/ilias-pre-update-$(date +%Y%m%d).tar.gz /etc/ilias

# Restart after updates
sudo systemctl restart ilias
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ilias

# Clean old logs
find /var/log/ilias -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ilias
```

## Additional Resources

- Official Documentation: https://docs.ilias.org/
- GitHub Repository: https://github.com/ilias/ilias
- Community Forum: https://forum.ilias.org/
- Best Practices Guide: https://docs.ilias.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
