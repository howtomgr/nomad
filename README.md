# nomad Installation Guide

nomad is a free and open-source flexible workload orchestrator. HashiCorp Nomad enables deployment and management of containers and non-containerized applications, serving as a simpler alternative to Kubernetes

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
  - RAM: 256MB minimum (2GB+ recommended)
  - Storage: 1GB for installation
  - Network: Cluster communication
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4646 (default nomad port)
  - Ports 4647-4648 for RPC
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

# Install nomad
sudo dnf install -y nomad

# Enable and start service
sudo systemctl enable --now nomad

# Configure firewall
sudo firewall-cmd --permanent --add-port=4646/tcp
sudo firewall-cmd --reload

# Verify installation
nomad version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nomad
sudo apt install -y nomad

# Enable and start service
sudo systemctl enable --now nomad

# Configure firewall
sudo ufw allow 4646

# Verify installation
nomad version
```

### Arch Linux

```bash
# Install nomad
sudo pacman -S nomad

# Enable and start service
sudo systemctl enable --now nomad

# Verify installation
nomad version
```

### Alpine Linux

```bash
# Install nomad
apk add --no-cache nomad

# Enable and start service
rc-update add nomad default
rc-service nomad start

# Verify installation
nomad version
```

### openSUSE/SLES

```bash
# Install nomad
sudo zypper install -y nomad

# Enable and start service
sudo systemctl enable --now nomad

# Configure firewall
sudo firewall-cmd --permanent --add-port=4646/tcp
sudo firewall-cmd --reload

# Verify installation
nomad version
```

### macOS

```bash
# Using Homebrew
brew install nomad

# Start service
brew services start nomad

# Verify installation
nomad version
```

### FreeBSD

```bash
# Using pkg
pkg install nomad

# Enable in rc.conf
echo 'nomad_enable="YES"' >> /etc/rc.conf

# Start service
service nomad start

# Verify installation
nomad version
```

### Windows

```bash
# Using Chocolatey
choco install nomad

# Or using Scoop
scoop install nomad

# Verify installation
nomad version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nomad

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nomad version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nomad

# Start service
sudo systemctl start nomad

# Stop service
sudo systemctl stop nomad

# Restart service
sudo systemctl restart nomad

# Check status
sudo systemctl status nomad

# View logs
sudo journalctl -u nomad -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nomad default

# Start service
rc-service nomad start

# Stop service
rc-service nomad stop

# Restart service
rc-service nomad restart

# Check status
rc-service nomad status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nomad_enable="YES"' >> /etc/rc.conf

# Start service
service nomad start

# Stop service
service nomad stop

# Restart service
service nomad restart

# Check status
service nomad status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nomad
brew services stop nomad
brew services restart nomad

# Check status
brew services list | grep nomad
```

### Windows Service Manager

```powershell
# Start service
net start nomad

# Stop service
net stop nomad

# Using PowerShell
Start-Service nomad
Stop-Service nomad
Restart-Service nomad

# Check status
Get-Service nomad
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nomad_backend {
    server 127.0.0.1:4646;
}

server {
    listen 80;
    server_name nomad.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nomad.example.com;

    ssl_certificate /etc/ssl/certs/nomad.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nomad.example.com.key;

    location / {
        proxy_pass http://nomad_backend;
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
    ServerName nomad.example.com
    Redirect permanent / https://nomad.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nomad.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nomad.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nomad.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4646/
    ProxyPassReverse / http://127.0.0.1:4646/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nomad_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nomad.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nomad_backend

backend nomad_backend
    balance roundrobin
    server nomad1 127.0.0.1:4646 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nomad:nomad /etc/nomad
sudo chmod 750 /etc/nomad

# Configure firewall
sudo firewall-cmd --permanent --add-port=4646/tcp
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
sudo systemctl status nomad

# View logs
sudo journalctl -u nomad -f

# Monitor resource usage
top -p $(pgrep nomad)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nomad"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nomad-backup-$DATE.tar.gz" /etc/nomad /var/lib/nomad

echo "Backup completed: $BACKUP_DIR/nomad-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nomad

# Restore from backup
tar -xzf /backup/nomad/nomad-backup-*.tar.gz -C /

# Start service
sudo systemctl start nomad
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nomad -n 100
sudo tail -f /var/log/nomad/nomad.log

# Check configuration
nomad version

# Check permissions
ls -la /etc/nomad
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4646

# Test connectivity
telnet localhost 4646

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nomad)

# Check disk I/O
iotop -p $(pgrep nomad)

# Check connections
ss -an | grep 4646
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nomad:
    image: nomad:latest
    ports:
      - "4646:4646"
    volumes:
      - ./config:/etc/nomad
      - ./data:/var/lib/nomad
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nomad

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nomad

# Arch Linux
sudo pacman -Syu nomad

# Alpine Linux
apk update && apk upgrade nomad

# openSUSE
sudo zypper update nomad

# FreeBSD
pkg update && pkg upgrade nomad

# Always backup before updates
tar -czf /backup/nomad-pre-update-$(date +%Y%m%d).tar.gz /etc/nomad

# Restart after updates
sudo systemctl restart nomad
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nomad

# Clean old logs
find /var/log/nomad -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nomad
```

## Additional Resources

- Official Documentation: https://docs.nomad.org/
- GitHub Repository: https://github.com/nomad/nomad
- Community Forum: https://forum.nomad.org/
- Best Practices Guide: https://docs.nomad.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
