# CleanStart OpenLDAP Container

This container provides a production-ready OpenLDAP directory server that can be deployed in Kubernetes or used standalone.

## Overview

OpenLDAP is an open-source implementation of the Lightweight Directory Access Protocol (LDAP), a standard protocol for accessing and managing directory information services. This container provides a fully functional LDAP directory server using slapd (Standalone LDAP Daemon).

## What This Container Provides

- **LDAP Directory Server** - Full-featured LDAP v3 server
- **Dual Protocol Support** - Both LDAP (port 389) and LDAPS (port 636)
- **Production Ready** - Runs as non-root user with minimal required capabilities
- **Debug Logging** - Verbose logging enabled for troubleshooting
- **Kubernetes Optimized** - Designed for easy deployment in Kubernetes clusters

## Key Features

### Protocol Support
- **LDAP** (port 389) - Unencrypted LDAP connections
- **LDAPS** (port 636) - LDAP over SSL/TLS (requires TLS certificates)

### Security
- Runs as `ldap` user (non-root)
- Minimal Linux capabilities (only NET_BIND_SERVICE, SETUID, SETGID)
- Supports TLS/SSL encryption for secure connections
- Read-only root filesystem support

## Container Image

**Image:** `cleanstart/openldap:latest-dev`

**Entrypoint:** `slapd -u ldap -g ldap -h ldap:// ldaps:// -d 64`

**Ports:**
- `389/tcp` - LDAP (unencrypted)
- `636/tcp` - LDAPS (encrypted/TLS)

**Architecture:** `amd64`  
**OS:** `linux`

For detailed deployment instructions, see [kubernetes/README.md](kubernetes/README.md).

## Important Notes

### LDAPS (TLS) Configuration

**LDAPS requires TLS certificates to function.** 
Without certificates:
- Port 636 will be listening, but connections will fail
- You'll get "Can't contact LDAP server (-1)" errors
- Use LDAP on port 389 for testing without TLS

To enable LDAPS:
1. Generate TLS certificates
2. Create a Kubernetes secret with the certificates
3. Mount the certificates in the deployment
4. Configure slapd to use them

See the Kubernetes deployment guide for detailed TLS setup instructions.

### Port Usage

- **Port 389/636** - Only accessible inside Kubernetes cluster
- **Port 10389/10636** - Port-forwarded ports for local testing
- Always use port-forwarded ports (10389/10636) when connecting from your local machine

## Use Cases

- **User Authentication** - Centralized user directory for applications
- **Identity Management** - Store and manage user identities and attributes
- **Service Discovery** - Directory of services and their configurations
- **Application Integration** - LDAP-based authentication for web applications
- **Development/Testing** - Local LDAP server for application development

