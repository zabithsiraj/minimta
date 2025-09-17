# MiniMTA

A lightweight, send-only SMTP MTA (Mail Transfer Agent) written in Go.

## Features

- **SMTP Submission Server**: Accepts emails on port 587 with STARTTLS
- **Authentication**: Supports AUTH PLAIN with plaintext passwords
- **DKIM Signing**: Optional DKIM signature support
- **File-based Queue**: Reliable message storage with JSON metadata
- **Delivery Worker**: Processes queued messages with retry logic
- **CLI Tools**: Queue management and monitoring
- **Configuration**: YAML-based configuration
- **Logging**: Structured logging with Zap
- **Easy Installation**: One-command installation script

## Quick Installation

### Prerequisites

- Linux system (Ubuntu/Debian/CentOS/RHEL/Fedora/Arch)
- Root access (sudo)
- Internet connection

**Note**: All prerequisites (Go, OpenSSL, Git, etc.) are automatically installed by the script.

### One-Command Installation

```bash
# Clone and install MiniMTA
git clone <repository-url>
cd minimta
sudo ./install.sh
```

The installation script will:
- **Install all prerequisites**: Go, OpenSSL, Git, build tools
- **Create system user and directories**: Secure service user setup
- **Build all binaries**: Compile from source
- **Generate SSL certificates**: Self-signed certificates for TLS
- **Create systemd services**: Automatic service management
- **Set up configuration files**: Default configuration with examples

### Start Services

```bash
# Start both services
sudo systemctl start mta-intake mta-deliver

# Enable auto-start on boot
sudo systemctl enable mta-intake mta-deliver

# Check status
sudo systemctl status mta-intake mta-deliver
```

### Default Configuration

After installation:
- **SMTP Server**: Port 587 with STARTTLS
- **Default User**: `admin` / `admin123`
- **Config Directory**: `/etc/minimta/`
- **Install Directory**: `/opt/minimta/`
- **Logs**: `journalctl -u mta-intake -f`

## Manual Installation

If you prefer manual installation:

1. **Clone and build**:
   ```bash
   git clone <repository-url>
   cd minimta
   go mod tidy
   go build -o bin/mta-intake cmd/mta-intake/main.go
   go build -o bin/mta-deliver cmd/mta-deliver/main.go
   go build -o bin/mta-cli cmd/mta-cli/main.go
   ```

2. **Configure**:
   ```bash
   cp config.yaml.example config.yaml
   cp users.json.example users.json
   # Edit configuration files
   ```

3. **Run**:
   ```bash
   ./bin/mta-intake -config config.yaml
   ./bin/mta-deliver -config config.yaml
   ```

## Configuration

### SMTP Server Settings

Edit `/etc/minimta/config.yaml`:

```yaml
server:
  hostname: "mail.yourdomain.com"
  listen_addr: "0.0.0.0:587"
  max_message_size: 10485760  # 10MB

tls:
  cert_file: "/etc/minimta/certs/server.crt"
  key_file: "/etc/minimta/certs/server.key"
```

### User Management

Edit `/etc/minimta/users.json`:

```json
[
  {
    "username": "admin",
    "password": "your-secure-password",
    "email": "admin@yourdomain.com",
    "active": true
  }
]
```

## Usage

### SMTP Client Configuration

Configure your email client with:
- **Server**: `your-server-ip`
- **Port**: `587`
- **Encryption**: `STARTTLS`
- **Authentication**: `Username/Password`
- **Username**: `admin` (or your configured user)
- **Password**: `admin123` (or your configured password)

### CLI Commands

```bash
# List queued messages
mta queue list

# Retry a failed message
mta queue retry <message-id>

# View service logs
journalctl -u mta-intake -f
journalctl -u mta-deliver -f
```

### Service Management

```bash
# Start services
sudo systemctl start mta-intake mta-deliver

# Stop services
sudo systemctl stop mta-intake mta-deliver

# Restart services
sudo systemctl restart mta-intake mta-deliver

# Check status
sudo systemctl status mta-intake mta-deliver
```

## Architecture

MiniMTA consists of two main components:

1. **mta-intake**: Submission server that accepts messages from authenticated clients
2. **mta-deliver**: Delivery worker that processes the queue and sends messages

## Development

### Project Structure

```
minimta/
├── cmd/
│   ├── mta-intake/     # Submission server
│   ├── mta-deliver/    # Delivery worker
│   └── mta-cli/        # CLI tools
├── internal/
│   ├── auth/           # Authentication
│   ├── config/         # Configuration
│   ├── dkim/           # DKIM signing
│   ├── delivery/       # SMTP delivery
│   ├── log/            # Logging
│   ├── queue/          # Message queue
│   └── smtpserver/     # SMTP server
├── systemd/            # Systemd service files
├── config.yaml.example # Example configuration
├── users.json.example  # Example users
├── install.sh          # Installation script
└── README.md           # This file
```

### Building from Source

```bash
# Install dependencies
go mod tidy

# Build all binaries
go build -o bin/mta-intake cmd/mta-intake/main.go
go build -o bin/mta-deliver cmd/mta-deliver/main.go
go build -o bin/mta-cli cmd/mta-cli/main.go

# Run tests
go test ./...
```

## Security Notes

- **Plaintext Passwords**: This implementation uses plaintext passwords for simplicity. For production use, consider implementing proper password hashing.
- **Self-signed Certificates**: The installation script generates self-signed certificates. For production, use proper SSL certificates.
- **Firewall**: Ensure proper firewall configuration to protect the SMTP port.

## Troubleshooting

### Common Issues

1. **Service won't start**:
   ```bash
   journalctl -u mta-intake -f
   ```

2. **Authentication fails**:
   - Check `/etc/minimta/users.json`
   - Verify username/password combination

3. **TLS errors**:
   - Check certificate files in `/etc/minimta/certs/`
   - Verify certificate permissions

4. **Queue not processing**:
   - Check if `mta-deliver` service is running
   - Verify queue directory permissions

### Logs

```bash
# View intake server logs
journalctl -u mta-intake -f

# View delivery worker logs
journalctl -u mta-deliver -f

# View all MiniMTA logs
journalctl -u mta-* -f
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## Support

For issues and questions:
- Create an issue on GitHub
- Check the troubleshooting section
- Review the logs for error messages