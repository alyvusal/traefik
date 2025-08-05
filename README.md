# Traefik Reverse Proxy

A production-ready Traefik v3.1 reverse proxy configuration for Docker Compose deployments with comprehensive routing, SSL termination, and service discovery capabilities.

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

## Overview

This Traefik setup serves as a base deployment for other tools in the repository, providing a unified frontend when using Docker Compose for deployment. It includes automatic service discovery, SSL termination, and a comprehensive dashboard for monitoring and management.

## Features

- **Multi-Entry Point Support**: HTTP (80), HTTPS (443), SSH (22), DNS (53), and Registry (5005)
- **Automatic Service Discovery**: Docker provider integration with custom routing rules
- **Dashboard**: Built-in Traefik dashboard for monitoring and configuration
- **Network Segmentation**: Separate frontend and backend networks for security
- **Health Check**: Optional whoami service for testing and validation
- **Production Ready**: Optimized for production deployments with proper restart policies

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Traefik       │    │   Backend       │
│   Network       │◄──►│   Reverse Proxy │◄──►│   Network       │
│   (Public)      │    │   (Router)      │    │   (Private)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Quick Start

1. **Clone and Navigate**:
   ```bash
   cd traefik/compose
   ```

2. **Configure Environment** (Optional):
   ```bash
   # Edit env.ini if needed for custom environment variables
   cp env.ini.example env.ini
   ```

3. **Start Traefik**:
   ```bash
   docker-compose up -d
   ```

4. **Access Dashboard**:
   ```
   http://traefik-192.168.0.100.nip.io/dashboard/
   ```

5. **Test with Whoami** (Optional):
   ```bash
   docker-compose --profile who up -d whoami
   # Access: http://whoami-192.168.0.100.nip.io
   ```

## Configuration

### Entry Points

| Port | Protocol | Purpose | External Access |
|------|----------|---------|-----------------|
| 80   | HTTP     | Web traffic | ✅ |
| 443  | HTTPS    | Secure web traffic | ✅ |
| 22   | SSH      | SSH tunneling | ✅ |
| 53   | DNS      | DNS queries (TCP/UDP) | ✅ |
| 5005 | TCP      | Docker registry | ✅ |

### Network Configuration

- **Frontend Network**: Public-facing services and Traefik
- **Backend Network**: Internal services and applications
- **Bridge Driver**: Standard Docker networking for container communication

### Service Discovery

Traefik automatically discovers services with the following label pattern:
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.{service}.entrypoints=web"
  - "traefik.http.routers.{service}.rule=Host(`{service}-192.168.0.100.nip.io`)"
```

### Default Routing Rule

Services are automatically routed using the pattern:
```
Host(`{{ trimPrefix `/` .Name }}-192.168.0.100.nip.io`)
```

## Usage Examples

### Adding a New Service

1. **Add to docker-compose.yml**:
   ```yaml
   services:
     myapp:
       image: myapp:latest
       networks: [backend]
       labels:
         - "traefik.enable=true"
         - "traefik.http.routers.myapp.entrypoints=web"
         - "traefik.http.routers.myapp.rule=Host(`myapp-192.168.0.100.nip.io`)"
   ```

2. **Deploy**:
   ```bash
   docker-compose up -d myapp
   ```

3. **Access**: `http://myapp-192.168.0.100.nip.io`

### Custom Domain Configuration

For custom domains, modify the routing rule:
```yaml
labels:
  - "traefik.http.routers.myapp.rule=Host(`myapp.example.com`)"
```

## Security Considerations

- **Docker Socket**: Read-only access to `/var/run/docker.sock`
- **Network Isolation**: Separate frontend/backend networks
- **Port Binding**: Limited to specific IP addresses (127.0.0.1, 192.168.0.100)
- **Service Discovery**: Disabled by default, enabled per service

## Monitoring and Logging

- **Dashboard**: Available at `/dashboard/` path
- **Log Level**: DEBUG (configurable in traefik.yml)
- **API**: Internal API for service discovery and configuration

## Troubleshooting

### Common Issues

1. **Service Not Accessible**:
   - Verify `traefik.enable=true` label
   - Check network configuration
   - Ensure proper routing rules

2. **Dashboard Not Loading**:
   - Verify dashboard is enabled in traefik.yml
   - Check routing rules for dashboard access

3. **Port Conflicts**:
   - Ensure ports are not used by other services
   - Check firewall settings

### Debug Mode

Enable debug logging by uncommenting in docker-compose.yml:
```yaml
command:
  - "--log.level=DEBUG"
```

## Dependencies

- Docker Engine 20.10+
- Docker Compose 2.0+
- Network access to 192.168.0.100 (configurable)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Related Projects

This Traefik configuration is designed to work with other tools in the SRE repository, providing a unified reverse proxy solution for microservices and containerized applications.
