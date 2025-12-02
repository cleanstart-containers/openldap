# OpenLDAP - Kubernetes Deployment Guide

Complete Kubernetes deployment guide for the CleanStart OpenLDAP container. OpenLDAP is an open-source implementation of the Lightweight Directory Access Protocol (LDAP).

**What OpenLDAP Does:**
- Provides LDAP directory services on port 389 (LDAP) and port 636 (LDAPS)
- Runs slapd (Standalone LDAP Daemon) as the LDAP server
- Supports both unencrypted (ldap://) and encrypted (ldaps://) connections
- Operates with debug level 64 for detailed logging

## Files

- `deployment.yaml` - Complete deployment manifest (Namespace, Deployment, Service)
- `README.md` - This documentation

## Image Details

**Image:** `cleanstart/openldap:latest-dev`

**Key Features:**
- **Entrypoint:** `slapd -u ldap -g ldap -h ldap:// ldaps:// -d 64`
- **Ports:** 
  - `389/tcp` - LDAP (unencrypted)
  - `636/tcp` - LDAPS (encrypted/TLS)
- **User:** `ldap` (non-root)
- **Architecture:** `amd64`
- **OS:** `linux`
- **SSL Certificates:** Pre-configured at `/etc/ssl/certs/ca-certificates.crt`
- **Debug Level:** 64 (detailed logging)

## Complete Deployment Steps

### Prerequisites

1. **Kubernetes cluster** (Kind, minikube, k3s, GKE, EKS, AKS, or any other)
2. **kubectl** installed and configured to access your cluster
3. **LDAP client tools** (for local testing):
   - **Ubuntu/Debian:** `sudo apt install ldap-utils`
   - **RHEL/CentOS/Fedora:** `sudo yum install openldap-clients` or `sudo dnf install openldap-clients`
   - **Alternative:** Use `ldapsearch` from another container (see Option C below)

### Step 1: Verify Kubernetes Cluster

```bash
# Check cluster connectivity
kubectl cluster-info

# Verify nodes are ready
kubectl get nodes
```

**Expected Output:**
```
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   5m    v1.27.3
```

### Step 2: Deploy OpenLDAP

```bash
# Navigate to the deployment directory
cd containers/openldap/kubernetes

# Apply the deployment
kubectl apply -f deployment.yaml
```

**Expected Output:**
```
namespace/openldap created
deployment.apps/openldap created
service/openldap created
```

### Step 3: Verify Deployment

```bash
# Check if the namespace was created
kubectl get namespace openldap

# Watch the pod status (press Ctrl+C to exit)
kubectl get pods -n openldap -w
```

**Expected Output:**
```
NAME                        READY   STATUS    RESTARTS   AGE
openldap-xxxxxxxxxx-xxxxx   1/1     Running   0          30s
```

### Step 4: Check Pod Logs

```bash
# Get pod name
POD_NAME=$(kubectl get pods -n openldap -l app=openldap -o jsonpath='{.items[0].metadata.name}')

# Follow logs in real-time
kubectl logs -n openldap -f $POD_NAME
```

**Expected Output:**
You should see slapd startup messages and debug output.

### Step 5: Verify Service

```bash
# Check service
kubectl get svc -n openldap

# Get service details
kubectl describe svc openldap -n openldap
```

**Expected Output:**
```
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
openldap   ClusterIP   10.96.xxx.xxx   <none>        389/TCP,636/TCP  1m
```

### Step 6: Test Functionality

#### Option A: Port Forward (Recommended for Testing)

**Note:** Ports 389 and 636 are privileged ports (< 1024) and require root privileges on most systems. Using non-privileged ports for port-forwarding.

**Install LDAP client tools first (if not already installed):**
```bash
# Ubuntu/Debian
sudo apt install ldap-utils

# RHEL/CentOS/Fedora
sudo yum install openldap-clients
# or
sudo dnf install openldap-clients

# macOS
brew install openldap
```

**Forward ports:**
```bash
# Forward local ports to pod ports (using non-privileged ports)
# Terminal 1: Forward LDAP port (local:10389 -> pod:389)
kubectl port-forward -n openldap svc/openldap 10389:389

# Terminal 2: Forward LDAPS port (local:10636 -> pod:636)
kubectl port-forward -n openldap svc/openldap 10636:636
```

**Test LDAP Connection:**
```bash
# In another terminal, test LDAP connection using the forwarded port
# Basic connection test (no authentication)
ldapsearch -H ldap://localhost:10389 -x -s base -b "" namingContexts

# Or using ldapwhoami to test anonymous bind
ldapwhoami -H ldap://localhost:10389 -x
```

**Test LDAPS Connection:**
**Note:** The current deployment configuration listens on `ldaps://` but does not include TLS certificate configuration. For LDAPS to work, you need to:
- Create TLS certificates (certificate and private key)
- Mount them as Kubernetes secrets
- Configure slapd to use them (typically via `TLS_CERT` and `TLS_KEY` environment variables or slapd configuration)
```bash
# Test LDAPS connection using the forwarded port
# Note: LDAPS requires TLS certificates to be configured in the container
ldapsearch -H ldaps://localhost:10636 -x -s base -b "" namingContexts

# If certificate verification fails, you can try:
# 1. Skip certificate verification (not recommended for production)
ldapsearch -H ldaps://localhost:10636 -x -s base -b "" namingContexts -o ldif-wrap=no

# 2. Use StartTLS instead (upgrades plain LDAP to TLS)
ldapsearch -H ldap://localhost:10389 -ZZ -x -s base -b "" namingContexts
```

#### Option B: Exec into Pod and Test from Inside

```bash
# Get pod name
POD_NAME=$(kubectl get pods -n openldap -l app=openldap -o jsonpath='{.items[0].metadata.name}')

# Exec into the pod
kubectl exec -it -n openldap $POD_NAME -- /bin/sh

# Inside the pod, test slapd is running
# Check if slapd process is running
ps aux | grep slapd

# Test local connection
ldapsearch -H ldap://localhost:389 -x -s base -b "" namingContexts

# Test LDAPS locally
ldapsearch -H ldaps://localhost:636 -x -s base -b "" namingContexts
```

#### Option C: Test Service DNS Resolution

```bash
# Get pod name
POD_NAME=$(kubectl get pods -n openldap -l app=openldap -o jsonpath='{.items[0].metadata.name}')

# Test DNS resolution from within the cluster
kubectl exec -n openldap $POD_NAME -- nslookup openldap.openldap.svc.cluster.local

# Test connectivity to service
kubectl exec -n openldap $POD_NAME -- nc -zv openldap.openldap.svc.cluster.local 389
kubectl exec -n openldap $POD_NAME -- nc -zv openldap.openldap.svc.cluster.local 636
```

## Common LDAP Operations

### Basic Search Operations

Use port **10389** (port-forwarded) instead of port 389. Port 389 is only accessible inside the Kubernetes cluster.

```bash
# Search for all entries (base search)
ldapsearch -H ldap://localhost:10389 -x -s base -b "" "(objectclass=*)"

# Search with scope onelevel
ldapsearch -H ldap://localhost:10389 -x -s one -b "dc=example,dc=com" "(objectclass=*)"

# Search with scope subtree
ldapsearch -H ldap://localhost:10389 -x -s sub -b "dc=example,dc=com" "(objectclass=*)"
```

### Check Server Status

```bash
# Get server information
ldapsearch -H ldap://localhost:10389 -x -s base -b "" "+"

# Get supported SASL mechanisms
ldapsearch -H ldap://localhost:10389 -x -s base -b "" supportedSASLMechanisms

# Get supported LDAP versions
ldapsearch -H ldap://localhost:10389 -x -s base -b "" supportedLDAPVersion
```

**Use cases:**

1. **Use LDAP instead of LDAPS (for testing):**
   - LDAP on port 389 works without TLS certificates
   - Use `ldap://localhost:10389` instead of `ldaps://localhost:10636`

2. **Configure TLS certificates (for production):**
   - Generate TLS certificates (self-signed or from a CA)
   - Mount certificates as Kubernetes secrets
   - Configure slapd to use the certificates via environment variables or configuration files
   - Update the deployment to include certificate volumes

3. **Use StartTLS instead of LDAPS:**
   - StartTLS allows upgrading a plain LDAP connection to TLS
   - Connect via `ldap://` and use the `-ZZ` flag to request StartTLS:
   ```bash
   ldapsearch -H ldap://localhost:10389 -ZZ -x -s base -b "" namingContexts
   ```

### Check Container Resources

```bash
# Check resource usage
kubectl top pod -n openldap

# Check resource limits
kubectl describe pod -n openldap -l app=openldap | grep -A 5 "Limits\|Requests"
```

### Debug Logs

The container runs with debug level 64, which provides detailed logging. Check logs for:
- LDAP connection attempts
- Bind operations
- Search operations
- Configuration issues

```bash
# Follow logs in real-time
kubectl logs -n openldap -f -l app=openldap

# Get last 100 lines
kubectl logs -n openldap -l app=openldap --tail=100
```

## Cleanup

```bash
# Delete the deployment
kubectl delete -f deployment.yaml

# Or delete namespace (this will delete everything in the namespace)
kubectl delete namespace openldap

# Verify deletion
kubectl get namespace openldap
```

