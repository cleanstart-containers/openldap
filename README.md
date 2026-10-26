## Container Documentation for Openldap Documentation

The CleanStart Openldap image provides a production-ready, security-hardened container optimized for enterprise environments. Built on a minimal base OS with comprehensive security hardening, this image delivers reliable application execution with advanced security features.

📌 **Base Foundation**: Production-ready container from cleanstart.

**Image Path**: `ghcr.io/cleanstart-containers/openldap`

**Registry**: cleanstart Registry

## Pull Latest Image
Download the container image from the registry

```bash
docker pull ghcr.io/cleanstart-containers/openldap:latest
```
```bash
docker pull ghcr.io/cleanstart-containers/openldap:latest-dev
```

## Basic Run
Run the container with basic configuration

```bash
docker run -it --name openldap ghcr.io/cleanstart-containers/openldap:latest
```

## Production Deployment
Create slapd.conf for bind mount
```bash
create file - nano ~/slapd.conf

- paste below config -
include         /etc/openldap/schema/core.schema
include         /etc/openldap/schema/cosine.schema
include         /etc/openldap/schema/inetorgperson.schema
include         /etc/openldap/schema/nis.schema

modulepath      /usr/lib/openldap
moduleload      back_mdb.so

database        config

database        mdb
maxsize         1073741824
suffix          "dc=my-domain,dc=com"
rootdn          "cn=Manager,dc=my-domain,dc=com"
rootpw          {SSHA}yourhashedpasswordhere
directory       /var/lib/openldap/openldap-data
index           objectClass     eq

database        monitor
```
### Production usage
```bash
docker rm -f openldap-prod 2>/dev/null; \
docker volume create openldap-prod-libdir && \
docker run --rm -v openldap-prod-libdir:/mount alpine sh -c "mkdir -p /mount/openldap-data && chown -R 389:389 /mount && chmod -R 700 /mount" && \
chmod 644 ~/slapd.conf && \
docker run -d --name openldap-prod \
  --security-opt=no-new-privileges \
  --user 389:389 \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  --restart unless-stopped \
  -v openldap-prod-libdir:/var/lib/openldap \
  -v ~/slapd.conf:/etc/openldap/slapd.conf:ro \
  -p 389:389 -p 636:636 \
  --entrypoint slapd \
  ghcr.io/cleanstart-containers/openldap:latest \
  -h "ldap:// ldaps://" -d 0
```


Volume Mount Mount local directory for persistent data

```bash
docker run -it --name openldap \
  -v openldap-data:/var/lib/openldap/openldap-data \
  ghcr.io/cleanstart-containers/openldap:latest
```

Port Forwarding Run with custom port mappings

```bash
docker run -it --name openldap \
  -p 1389:389 -p 1636:636 \
  ghcr.io/cleanstart-containers/openldap:latest
```

## Documentation Resources
Essential links and resources for further information

- **Container Registry**: [https://www.cleanstart.com/](https://www.cleanstart.com/)
- **CleanStart Community Images**: [https://hub.docker.com/u/cleanstart](https://hub.docker.com/u/cleanstart)
- **How-to-Run CleanStart images & sample projects**: [https://github.com/orgs/cleanstart-containers/](https://github.com/orgs/cleanstart-containers/)
  - How to run sample projects using Dockerfile
  - How to deploy via Kubernetes YAML
  - How to migrate from public images to CleanStart images

---

**Vulnerability Disclaimer**

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

Security remains a shared responsibility: CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.
