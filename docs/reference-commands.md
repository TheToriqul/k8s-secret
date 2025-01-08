# Kubernetes Secrets Management Command Reference

> **Author**: [Md Toriqul Islam](https://linkedin.com/in/thetoriqul)  
> **Focus**: Advanced Kubernetes Secrets Management  
> **Note**: This reference guide documents commands for educational purposes. Always validate commands in a safe environment before production use.

## Table of Contents
- [Core Operations](#core-operations)
  - [Secret Creation](#secret-creation)
  - [Secret Management](#secret-management)
  - [Secret Viewing](#secret-viewing)
- [Advanced Operations](#advanced-operations)
  - [Pod Integration](#pod-integration)
  - [Base64 Operations](#base64-operations)
  - [Secret Updates](#secret-updates)
  - [Secret Rotation](#secret-rotation)
- [Troubleshooting](#troubleshooting)
  - [Common Issues](#common-issues)
  - [Debugging](#debugging)
  - [Validation](#validation)
- [Security](#security)
  - [RBAC Management](#rbac-management)
  - [Best Practices](#best-practices)
  - [Access Control](#access-control)
- [Production Operations](#production-operations)
  - [Maintenance](#maintenance)
  - [Backup](#backup)
  - [Restoration](#restoration)
- [Cleanup](#cleanup)

## Core Operations

### Secret Creation

```bash
# 1. Create from literal values
kubectl create secret generic db-credentials \
    --from-literal=username=admin \
    --from-literal=password=secretpassword \
    --from-literal=host=db.example.com \
    --from-literal=port=3306

# 2. Create from files
kubectl create secret generic tls-certs \
    --from-file=./tls.key \
    --from-file=./tls.crt \
    --from-file=./ca.crt

# 3. Create from configuration file
kubectl apply -f secret-config.yaml

# 4. Create Docker registry secret
kubectl create secret docker-registry regcred \
    --docker-server=<your-registry-server> \
    --docker-username=<your-username> \
    --docker-password=<your-password> \
    --docker-email=<your-email>

# 5. Create TLS secret
kubectl create secret tls tls-secret \
    --cert=path/to/cert/file \
    --key=path/to/key/file
```

### Secret Management

```bash
# Export secret to a file
kubectl get secret db-credentials -o yaml > exported-secret.yaml

# Edit secret directly
kubectl edit secret db-credentials

# Patch a secret
kubectl patch secret db-credentials -p '{"data":{"password":"bmV3cGFzc3dvcmQ="}}'

# Scale deployments using the secret
kubectl scale deployment secure-app --replicas=0
kubectl scale deployment secure-app --replicas=3

# Label secrets
kubectl label secret db-credentials environment=production
```

### Secret Viewing

```bash
# List all secrets
kubectl get secrets --all-namespaces

# Get detailed secret information
kubectl describe secret db-credentials

# Get specific secret values
kubectl get secret db-credentials -o jsonpath='{.data.username}' | base64 --decode
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 --decode

# View secret in YAML format
kubectl get secret db-credentials -o yaml

# List secrets with labels
kubectl get secrets -l environment=production
```

## Advanced Operations

### Pod Integration

```bash
# 1. Deploy pods with secrets
kubectl apply -f pod-with-env.yaml
kubectl apply -f pod-with-volume.yaml
kubectl apply -f deployment.yaml

# 2. Verify secret mounting
kubectl exec secret-vol-pod -- ls /etc/secrets
kubectl exec secret-vol-pod -- cat /etc/secrets/username

# 3. Check environment variables
kubectl exec secret-env-pod -- env | grep DB_
kubectl exec -it secret-env-pod -- sh -c 'echo $DB_PASSWORD'

# 4. Update pod with new secret
kubectl delete pod secret-env-pod
kubectl apply -f pod-with-env.yaml
```

### Base64 Operations

```bash
# Encode values
echo -n 'admin' | base64
echo -n 'secretpassword' | base64
echo -n 'db.example.com' | base64
echo -n '3306' | base64

# Decode values
echo 'YWRtaW4=' | base64 --decode
echo 'c2VjcmV0cGFzc3dvcmQ=' | base64 --decode

# Encode file content
base64 -w 0 < tls.key > tls.key.base64
base64 -w 0 < tls.crt > tls.crt.base64
```

### Secret Updates

```bash
# Update existing secret
kubectl create secret generic db-credentials \
    --from-literal=username=newadmin \
    --from-literal=password=newpass \
    -o yaml --dry-run=client | kubectl replace -f -

# Patch specific fields
kubectl patch secret db-credentials --type='json' -p='[{"op": "replace", "path": "/data/password", "value":"bmV3cGFzc3dvcmQ="}]'
```

## Troubleshooting

### Common Issues

```bash
# Check pod status
kubectl get pods
kubectl describe pod secret-env-pod

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check logs
kubectl logs secret-env-pod
kubectl logs -f deployment/secure-app

# Check mount points
kubectl exec secret-vol-pod -- mount | grep secrets
```

### Debugging

```bash
# Debug secret mounting
kubectl debug secret-vol-pod -it --image=busybox

# Check secret permissions
kubectl auth can-i get secrets
kubectl auth can-i create secrets
kubectl auth can-i update secrets

# Verify service account permissions
kubectl auth can-i get secrets --as=system:serviceaccount:secure-ns:restricted-sa
```

## Security

### RBAC Management

```bash
# Create namespace and configure RBAC
kubectl create namespace secure-ns
kubectl config set-context --current --namespace=secure-ns
kubectl apply -f rbac.yaml

# Verify RBAC permissions
kubectl auth can-i get secrets --as=system:serviceaccount:secure-ns:restricted-sa
kubectl auth can-i update secrets --as=system:serviceaccount:secure-ns:restricted-sa

# List roles and bindings
kubectl get roles
kubectl get rolebindings
```

### Access Control

```bash
# Create service account
kubectl create serviceaccount secret-admin

# Create role
kubectl create role secret-manager \
    --verb=get,list,watch,create,update \
    --resource=secrets

# Bind role to service account
kubectl create rolebinding secret-admin-binding \
    --role=secret-manager \
    --serviceaccount=default:secret-admin
```

## Production Operations

### Maintenance

```bash
# Rotate secrets
kubectl create secret generic db-credentials \
    --from-literal=username=admin \
    --from-literal=password=newpassword \
    -o yaml --dry-run=client | kubectl replace -f -

# Update applications
kubectl rollout restart deployment secure-app
```

### Backup

```bash
# Backup all secrets
kubectl get secrets -A -o yaml > all-secrets-backup.yaml

# Backup specific secret
kubectl get secret db-credentials -o yaml > db-credentials-backup.yaml
```

### Cleanup

```bash
# Delete individual resources
kubectl delete -f deployment.yaml
kubectl delete -f pod-with-env.yaml
kubectl delete -f pod-with-volume.yaml
kubectl delete -f secret-config.yaml
kubectl delete -f rbac.yaml

# Delete all resources in namespace
kubectl delete namespace secure-ns

# Remove specific secrets
kubectl delete secret db-credentials
kubectl delete secret tls-certs
```

## Best Practices

1. **Secret Naming Conventions**
   - Use descriptive names
   - Include environment/purpose
   - Follow consistent patterns

2. **Security Measures**
   - Enable encryption at rest
   - Use RBAC strictly
   - Rotate secrets regularly
   - Limit secret access

3. **Monitoring**
   - Track secret usage
   - Monitor access patterns
   - Audit secret changes
   - Log suspicious activities

4. **Maintenance**
   - Regular secret rotation
   - Backup procedures
   - Access review
   - Documentation updates

---

> 💡 **Best Practice**: Store sensitive data in Kubernetes Secrets, never in ConfigMaps or environment variables directly.

> ⚠️ **Warning**: Base64 encoding is not encryption. Always use additional security measures in production.

> 📝 **Note**: Secret values are limited to 1MB in size.

> 🔒 **Security**: Always follow the principle of least privilege when granting access to secrets.