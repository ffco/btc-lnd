# Bitcoin Core and Lightning Network Node Docker Setup

This repository contains a Docker Compose configuration for running a Bitcoin Core node and Lightning Network Daemon (LND) together.

## Prerequisites

- Docker
- Docker Compose
- At least 500GB of free disk space (for Bitcoin mainnet)
- Sufficient RAM (recommended: 8GB minimum)
- Ubuntu Server (recommended for production)

## Configuration

1. Before starting the containers, modify the following in `docker-compose.yml`:
   - Change `your_secure_password_here` to a strong password for both Bitcoin Core and LND
   - Replace `your-domain.com` with your actual domain
   - Replace `your-server-ip` with your server's IP address
   - Adjust ports if needed (default ports are standard for mainnet)

2. The setup uses the following ports:
   - Bitcoin Core: 8333 (P2P), 8332 (RPC - localhost only)
   - LND: 9735 (Lightning P2P), 10009 (gRPC - localhost only)

## Usage

### Running as Docker Compose (Manual)

1. Start the containers:
   ```bash
   docker-compose up -d
   ```

2. Monitor the logs:
   ```bash
   docker-compose logs -f
   ```

3. Stop the containers:
   ```bash
   docker-compose down
   ```

### Running as a System Service (Recommended)

1. Copy the systemd service file:
   ```bash
   sudo cp btc-lnd.service /etc/systemd/system/
   ```

2. Reload systemd to recognize the new service:
   ```bash
   sudo systemctl daemon-reload
   ```

3. Enable the service to start on boot:
   ```bash
   sudo systemctl enable btc-lnd
   ```

4. Start the service:
   ```bash
   sudo systemctl start btc-lnd
   ```

5. Check the service status:
   ```bash
   sudo systemctl status btc-lnd
   ```

6. View service logs:
   ```bash
   sudo journalctl -u btc-lnd -f
   ```

7. Stop the service:
   ```bash
   sudo systemctl stop btc-lnd
   ```

## Production Security Requirements

### System Security

1. Server Hardening:
   ```bash
   # Update system regularly
   sudo apt update && sudo apt upgrade -y
   
   # Install and configure UFW firewall
   sudo apt install ufw
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow ssh
   sudo ufw allow 8333/tcp  # Bitcoin P2P
   sudo ufw allow 9735/tcp  # Lightning P2P
   sudo ufw enable
   ```

2. SSH Security:
   - Use SSH keys instead of passwords
   - Change default SSH port
   - Disable root login
   - Use fail2ban for SSH protection

3. System Monitoring:
   - Install and configure monitoring tools (e.g., Prometheus, Grafana)
   - Set up log rotation
   - Configure system alerts

### Docker Security

1. Docker Hardening:
   ```bash
   # Create docker group and add user
   sudo groupadd docker
   sudo usermod -aG docker $USER
   
   # Configure Docker daemon security
   sudo mkdir -p /etc/docker
   sudo nano /etc/docker/daemon.json
   ```
   Add to daemon.json:
   ```json
   {
     "icc": false,
     "userns-remap": "default",
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "10m",
       "max-file": "3"
     },
     "dns": ["8.8.8.8", "8.8.4.4"],
     "dns-search": ["."],
     "mtu": 1400,
     "no-new-privileges": true,
     "seccomp-profile": "/etc/docker/seccomp-profiles/default.json"
   }
   ```

2. Container Security:
   - Regular image updates
   - Resource limits (already configured in docker-compose.yml)
   - Read-only root filesystem where possible
   - No privileged mode

### Network Security

1. RPC Access:
   - RPC and gRPC ports are restricted to localhost only
   - Use SSH tunnel for remote access:
     ```bash
     ssh -L 8332:localhost:8332 user@your-server
     ssh -L 10009:localhost:10009 user@your-server
     ```

2. Reverse Proxy (Optional):
   - Set up Nginx with SSL/TLS for any external access
   - Configure rate limiting
   - Enable HTTP/2

### Backup and Recovery

1. Regular Backups:
   ```bash
   # Backup script example
   #!/bin/bash
   BACKUP_DIR="/path/to/backups"
   DATE=$(date +%Y%m%d)
   
   # Backup LND data
   docker cp lnd:/root/.lnd $BACKUP_DIR/lnd_backup_$DATE
   
   # Backup Bitcoin Core data
   docker cp bitcoind:/home/bitcoin/.bitcoin $BACKUP_DIR/bitcoin_backup_$DATE
   
   # Compress backups
   tar -czf $BACKUP_DIR/lnd_backup_$DATE.tar.gz $BACKUP_DIR/lnd_backup_$DATE
   tar -czf $BACKUP_DIR/bitcoin_backup_$DATE.tar.gz $BACKUP_DIR/bitcoin_backup_$DATE
   ```

2. Recovery Procedures:
   - Document recovery steps
   - Test recovery process regularly
   - Keep multiple backup copies

### Monitoring and Maintenance

1. Regular Tasks:
   - Monitor disk space
   - Check system logs
   - Update Docker images
   - Verify backup integrity
   - Monitor network connectivity

2. Alerting:
   - Set up alerts for:
     - High CPU/Memory usage
     - Disk space warnings
     - Service failures
     - Network issues
     - Unauthorized access attempts

## Troubleshooting

1. If containers fail to start, check the logs:
   ```bash
   docker-compose logs bitcoind
   docker-compose logs lnd
   ```

2. Ensure all required ports are available and not blocked by firewall
3. Verify sufficient disk space is available
4. Check Docker logs for any permission issues
5. For service-related issues, check the systemd logs:
   ```bash
   sudo journalctl -u btc-lnd -n 100
   ```

## Additional Resources

- [Bitcoin Core Documentation](https://bitcoin.org/en/developer-documentation)
- [Lightning Network Documentation](https://docs.lightning.engineering/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [Ubuntu Server Security Guide](https://ubuntu.com/server/docs/security) 