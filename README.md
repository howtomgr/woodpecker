# woodpecker Installation Guide

woodpecker is a free and open-source simple CI/CD engine with great extensibility. Fork of Drone, Woodpecker provides container-native CI/CD with a focus on simplicity and extensibility, serving as an alternative to Jenkins or GitLab CI

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
  - CPU: 1 core minimum (2+ recommended)
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 1GB for builds
  - Network: Git repository access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8000 (default woodpecker port)
  - gRPC on port 9000
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

# Install woodpecker
sudo dnf install -y woodpecker

# Enable and start service
sudo systemctl enable --now woodpecker

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
woodpecker-server --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install woodpecker
sudo apt install -y woodpecker

# Enable and start service
sudo systemctl enable --now woodpecker

# Configure firewall
sudo ufw allow 8000

# Verify installation
woodpecker-server --version
```

### Arch Linux

```bash
# Install woodpecker
sudo pacman -S woodpecker

# Enable and start service
sudo systemctl enable --now woodpecker

# Verify installation
woodpecker-server --version
```

### Alpine Linux

```bash
# Install woodpecker
apk add --no-cache woodpecker

# Enable and start service
rc-update add woodpecker default
rc-service woodpecker start

# Verify installation
woodpecker-server --version
```

### openSUSE/SLES

```bash
# Install woodpecker
sudo zypper install -y woodpecker

# Enable and start service
sudo systemctl enable --now woodpecker

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
woodpecker-server --version
```

### macOS

```bash
# Using Homebrew
brew install woodpecker

# Start service
brew services start woodpecker

# Verify installation
woodpecker-server --version
```

### FreeBSD

```bash
# Using pkg
pkg install woodpecker

# Enable in rc.conf
echo 'woodpecker_enable="YES"' >> /etc/rc.conf

# Start service
service woodpecker start

# Verify installation
woodpecker-server --version
```

### Windows

```bash
# Using Chocolatey
choco install woodpecker

# Or using Scoop
scoop install woodpecker

# Verify installation
woodpecker-server --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/woodpecker

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
woodpecker-server --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable woodpecker

# Start service
sudo systemctl start woodpecker

# Stop service
sudo systemctl stop woodpecker

# Restart service
sudo systemctl restart woodpecker

# Check status
sudo systemctl status woodpecker

# View logs
sudo journalctl -u woodpecker -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add woodpecker default

# Start service
rc-service woodpecker start

# Stop service
rc-service woodpecker stop

# Restart service
rc-service woodpecker restart

# Check status
rc-service woodpecker status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'woodpecker_enable="YES"' >> /etc/rc.conf

# Start service
service woodpecker start

# Stop service
service woodpecker stop

# Restart service
service woodpecker restart

# Check status
service woodpecker status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start woodpecker
brew services stop woodpecker
brew services restart woodpecker

# Check status
brew services list | grep woodpecker
```

### Windows Service Manager

```powershell
# Start service
net start woodpecker

# Stop service
net stop woodpecker

# Using PowerShell
Start-Service woodpecker
Stop-Service woodpecker
Restart-Service woodpecker

# Check status
Get-Service woodpecker
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream woodpecker_backend {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name woodpecker.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name woodpecker.example.com;

    ssl_certificate /etc/ssl/certs/woodpecker.example.com.crt;
    ssl_certificate_key /etc/ssl/private/woodpecker.example.com.key;

    location / {
        proxy_pass http://woodpecker_backend;
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
    ServerName woodpecker.example.com
    Redirect permanent / https://woodpecker.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName woodpecker.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/woodpecker.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/woodpecker.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend woodpecker_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/woodpecker.pem
    redirect scheme https if !{ ssl_fc }
    default_backend woodpecker_backend

backend woodpecker_backend
    balance roundrobin
    server woodpecker1 127.0.0.1:8000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R woodpecker:woodpecker /etc/woodpecker
sudo chmod 750 /etc/woodpecker

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
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
sudo systemctl status woodpecker

# View logs
sudo journalctl -u woodpecker -f

# Monitor resource usage
top -p $(pgrep woodpecker)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/woodpecker"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/woodpecker-backup-$DATE.tar.gz" /etc/woodpecker /var/lib/woodpecker

echo "Backup completed: $BACKUP_DIR/woodpecker-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop woodpecker

# Restore from backup
tar -xzf /backup/woodpecker/woodpecker-backup-*.tar.gz -C /

# Start service
sudo systemctl start woodpecker
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u woodpecker -n 100
sudo tail -f /var/log/woodpecker/woodpecker.log

# Check configuration
woodpecker-server --version

# Check permissions
ls -la /etc/woodpecker
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8000

# Test connectivity
telnet localhost 8000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep woodpecker)

# Check disk I/O
iotop -p $(pgrep woodpecker)

# Check connections
ss -an | grep 8000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  woodpecker:
    image: woodpecker:latest
    ports:
      - "8000:8000"
    volumes:
      - ./config:/etc/woodpecker
      - ./data:/var/lib/woodpecker
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update woodpecker

# Debian/Ubuntu
sudo apt update && sudo apt upgrade woodpecker

# Arch Linux
sudo pacman -Syu woodpecker

# Alpine Linux
apk update && apk upgrade woodpecker

# openSUSE
sudo zypper update woodpecker

# FreeBSD
pkg update && pkg upgrade woodpecker

# Always backup before updates
tar -czf /backup/woodpecker-pre-update-$(date +%Y%m%d).tar.gz /etc/woodpecker

# Restart after updates
sudo systemctl restart woodpecker
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/woodpecker

# Clean old logs
find /var/log/woodpecker -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/woodpecker
```

## Additional Resources

- Official Documentation: https://docs.woodpecker.org/
- GitHub Repository: https://github.com/woodpecker/woodpecker
- Community Forum: https://forum.woodpecker.org/
- Best Practices Guide: https://docs.woodpecker.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
