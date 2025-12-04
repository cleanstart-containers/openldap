# CleanStart OpenLDAP Container

This container provides a production-ready OpenLDAP directory server that can be deployed in Kubernetes or used standalone. OpenLDAP is an open-source implementation of the Lightweight Directory Access Protocol (LDAP), a standard protocol for accessing and managing directory information services. This container provides a fully functional LDAP directory server using slapd (Standalone LDAP Daemon). The CleanStart OpenLDAP image provides a production-ready, security-hardened container optimized for enterprise environments. Built on a minimal base OS with comprehensive security hardening, this image delivers reliable application execution with advanced security features.

**📌 CleanStart Foundation:** Security-hardened, minimal base OS designed for enterprise containerized environments.

**Image Path:** `ghcr.io/cleanstart-containers/openldap`

**Registry:** CleanStart Registry

---

## Overview

OpenLDAP is an open-source implementation of the Lightweight Directory Access Protocol (LDAP), a standard protocol for accessing and managing directory information services. This container provides a fully functional LDAP directory server using slapd (Standalone LDAP Daemon), designed for easy deployment in Kubernetes clusters or standalone environments. This CleanStart OpenLDAP container is part of the CleanStart application suite, featuring enterprise-grade security hardening, automated vulnerability management, and compliance with industry standards.

---

## About CleanStart

CleanStart is a comprehensive container registry providing security-hardened, enterprise-ready container images. Our images are designed with security-first principles, featuring minimal attack surfaces, regular security updates, and compliance with industry standards.

### About CleanStart Images

CleanStart images are built on secure, minimal base operating systems and optimized for production environments. Each image undergoes rigorous security testing, vulnerability scanning, and compliance validation to ensure enterprise-grade security and reliability.

---

## Container Image

**Image:** `ghcr.io/cleanstart-containers/openldap:latest-dev`

**Entrypoint:** `slapd -u ldap -g ldap -h ldap:// ldaps:// -d 64`

**Ports:**
- `389/tcp` - LDAP (unencrypted)
- `636/tcp` - LDAPS (encrypted/TLS)

**Architecture:** `amd64`  
**OS:** `linux`

For detailed deployment instructions, see `kubernetes/README.md`.

---

## What This Container Provides

- **LDAP Directory Server** - Full-featured LDAP v3 server
- **Dual Protocol Support** - Both LDAP (port 389) and LDAPS (port 636)
- **Production Ready** - Runs as non-root user with minimal required capabilities
- **Debug Logging** - Verbose logging enabled for troubleshooting
- **Kubernetes Optimized** - Designed for easy deployment in Kubernetes clusters

---

## Key Features

- **Security-First Design**: Built with minimal attack surfaces and security hardening
- **Enterprise Compliance**: Meets industry standards including FIPS, STIG, and CIS benchmarks
- **Regular Updates**: Automated security patches and vulnerability management
- **Multi-Architecture Support**: Available for AMD64 and ARM64 architectures
- **Production Ready**: Optimized for enterprise deployment and scaling
- **Comprehensive Documentation**: Detailed guides and best practices for each image

### Protocol Support

- **LDAP** (port 389) - Unencrypted LDAP connections
- **LDAPS** (port 636) - LDAP over SSL/TLS (requires TLS certificates)

### Security

- Runs as `ldap` user (non-root)
- Minimal Linux capabilities (only NET_BIND_SERVICE, SETUID, SETGID)
- Supports TLS/SSL encryption for secure connections
- Read-only root filesystem support

---

## Use Cases

Typical scenarios where this container excels:

- **User Authentication** - Centralized user directory for applications
- **Identity Management** - Store and manage user identities and attributes
- **Service Discovery** - Directory of services and their configurations
- **Application Integration** - LDAP-based authentication for web applications
- **Development/Testing** - Local LDAP server for application development
- Enterprise directory services
- Centralized authentication systems
- Multi-application user management

---

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

---

## Quick Start

### Pull Commands
```bash
docker pull ghcr.io/cleanstart-containers/openldap:latest
docker pull ghcr.io/cleanstart-containers/openldap:latest-dev
```

### Run Commands

Basic test:
```bash
docker run -it --name openldap-test ghcr.io/cleanstart-containers/openldap:latest-dev
```

Production deployment:
```bash
docker run -d --name openldap-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  ghcr.io/cleanstart-containers/openldap:latest
```

---

## Architecture Support

CleanStart images support multiple architectures to ensure compatibility across different deployment environments:

- **AMD64**: Intel and AMD x86-64 processors
- **ARM64**: ARM-based processors including Apple Silicon and ARM servers

### Architecture-based Pull Commands
```bash
docker pull --platform linux/amd64 ghcr.io/cleanstart-containers/openldap:latest
docker pull --platform linux/arm64 ghcr.io/cleanstart-containers/openldap:latest
```

---

## Resources

- **Official Documentation:** https://www.openldap.org/doc/
- **OpenLDAP Documentation:** https://www.openldap.org/
- **Provenance / SBOM / Signature:** https://images.cleanstart.com/images/openldap
- **Docker Hub:** https://hub.docker.com/r/cleanstart/openldap
- **CleanStart All Images:** https://images.cleanstart.com
- **CleanStart Community Images:** https://hub.docker.com/u/cleanstart

---

## Vulnerability Disclaimer

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

**Security remains a shared responsibility:** CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.
