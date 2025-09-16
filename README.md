# cassandra Installation Guide

cassandra is a free and open-source distributed NoSQL database. Apache Cassandra provides high availability with no single point of failure, serving as an open-source alternative to Amazon DynamoDB or Azure Cosmos DB

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
  - CPU: 2+ cores minimum (8+ recommended)
  - RAM: 8GB minimum (32GB+ recommended)
  - Storage: 10GB+ for data
  - Network: Cluster communication
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9042 (default cassandra port)
  - Port 7000 for inter-node
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

# Install cassandra
sudo dnf install -y cassandra

# Enable and start service
sudo systemctl enable --now cassandra

# Configure firewall
sudo firewall-cmd --permanent --add-port=9042/tcp
sudo firewall-cmd --reload

# Verify installation
cassandra -v
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install cassandra
sudo apt install -y cassandra

# Enable and start service
sudo systemctl enable --now cassandra

# Configure firewall
sudo ufw allow 9042

# Verify installation
cassandra -v
```

### Arch Linux

```bash
# Install cassandra
sudo pacman -S cassandra

# Enable and start service
sudo systemctl enable --now cassandra

# Verify installation
cassandra -v
```

### Alpine Linux

```bash
# Install cassandra
apk add --no-cache cassandra

# Enable and start service
rc-update add cassandra default
rc-service cassandra start

# Verify installation
cassandra -v
```

### openSUSE/SLES

```bash
# Install cassandra
sudo zypper install -y cassandra

# Enable and start service
sudo systemctl enable --now cassandra

# Configure firewall
sudo firewall-cmd --permanent --add-port=9042/tcp
sudo firewall-cmd --reload

# Verify installation
cassandra -v
```

### macOS

```bash
# Using Homebrew
brew install cassandra

# Start service
brew services start cassandra

# Verify installation
cassandra -v
```

### FreeBSD

```bash
# Using pkg
pkg install cassandra

# Enable in rc.conf
echo 'cassandra_enable="YES"' >> /etc/rc.conf

# Start service
service cassandra start

# Verify installation
cassandra -v
```

### Windows

```bash
# Using Chocolatey
choco install cassandra

# Or using Scoop
scoop install cassandra

# Verify installation
cassandra -v
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/cassandra

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
cassandra -v
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable cassandra

# Start service
sudo systemctl start cassandra

# Stop service
sudo systemctl stop cassandra

# Restart service
sudo systemctl restart cassandra

# Check status
sudo systemctl status cassandra

# View logs
sudo journalctl -u cassandra -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add cassandra default

# Start service
rc-service cassandra start

# Stop service
rc-service cassandra stop

# Restart service
rc-service cassandra restart

# Check status
rc-service cassandra status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'cassandra_enable="YES"' >> /etc/rc.conf

# Start service
service cassandra start

# Stop service
service cassandra stop

# Restart service
service cassandra restart

# Check status
service cassandra status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start cassandra
brew services stop cassandra
brew services restart cassandra

# Check status
brew services list | grep cassandra
```

### Windows Service Manager

```powershell
# Start service
net start cassandra

# Stop service
net stop cassandra

# Using PowerShell
Start-Service cassandra
Stop-Service cassandra
Restart-Service cassandra

# Check status
Get-Service cassandra
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream cassandra_backend {
    server 127.0.0.1:9042;
}

server {
    listen 80;
    server_name cassandra.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cassandra.example.com;

    ssl_certificate /etc/ssl/certs/cassandra.example.com.crt;
    ssl_certificate_key /etc/ssl/private/cassandra.example.com.key;

    location / {
        proxy_pass http://cassandra_backend;
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
    ServerName cassandra.example.com
    Redirect permanent / https://cassandra.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName cassandra.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cassandra.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/cassandra.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9042/
    ProxyPassReverse / http://127.0.0.1:9042/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend cassandra_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/cassandra.pem
    redirect scheme https if !{ ssl_fc }
    default_backend cassandra_backend

backend cassandra_backend
    balance roundrobin
    server cassandra1 127.0.0.1:9042 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R cassandra:cassandra /etc/cassandra
sudo chmod 750 /etc/cassandra

# Configure firewall
sudo firewall-cmd --permanent --add-port=9042/tcp
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
sudo systemctl status cassandra

# View logs
sudo journalctl -u cassandra -f

# Monitor resource usage
top -p $(pgrep cassandra)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/cassandra"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/cassandra-backup-$DATE.tar.gz" /etc/cassandra /var/lib/cassandra

echo "Backup completed: $BACKUP_DIR/cassandra-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop cassandra

# Restore from backup
tar -xzf /backup/cassandra/cassandra-backup-*.tar.gz -C /

# Start service
sudo systemctl start cassandra
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u cassandra -n 100
sudo tail -f /var/log/cassandra/cassandra.log

# Check configuration
cassandra -v

# Check permissions
ls -la /etc/cassandra
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9042

# Test connectivity
telnet localhost 9042

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep cassandra)

# Check disk I/O
iotop -p $(pgrep cassandra)

# Check connections
ss -an | grep 9042
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  cassandra:
    image: cassandra:latest
    ports:
      - "9042:9042"
    volumes:
      - ./config:/etc/cassandra
      - ./data:/var/lib/cassandra
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update cassandra

# Debian/Ubuntu
sudo apt update && sudo apt upgrade cassandra

# Arch Linux
sudo pacman -Syu cassandra

# Alpine Linux
apk update && apk upgrade cassandra

# openSUSE
sudo zypper update cassandra

# FreeBSD
pkg update && pkg upgrade cassandra

# Always backup before updates
tar -czf /backup/cassandra-pre-update-$(date +%Y%m%d).tar.gz /etc/cassandra

# Restart after updates
sudo systemctl restart cassandra
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/cassandra

# Clean old logs
find /var/log/cassandra -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/cassandra
```

## Additional Resources

- Official Documentation: https://docs.cassandra.org/
- GitHub Repository: https://github.com/cassandra/cassandra
- Community Forum: https://forum.cassandra.org/
- Best Practices Guide: https://docs.cassandra.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
