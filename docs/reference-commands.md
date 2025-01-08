# Kubernetes Secrets Management Command Reference

> **Author**: [Md Toriqul Islam](https://linkedin.com/in/thetoriqul)  
> **Focus**: Advanced Kubernetes Secrets Management  
> **Note**: This reference guide documents commands for educational purposes. Always validate commands in a safe environment before production use.

## Table of Contents
- [Core Operations](#core-operations)
- [Advanced Operations](#advanced-operations)
- [Troubleshooting](#troubleshooting)
- [Production Guidelines](#production-guidelines)

## Core Operations

### Creating Secrets

```bash
# Create a secret from literal values
kubectl create secret generic db-credentials \
    --from-literal=username=admin \
    --from-literal=password=secretpassword

# Verify secret creation
kubectl get secret db-credentials

# Create a secret from files
kubectl create secret generic tls-certs \
    --from-file=private.key \
    --from-file=certificate.crt

# Create a secret from a configuration file
kubectl apply -f secret-config.yaml
```

### Viewing and Managing Secrets

```bash
# List all secrets in the current namespace
kubectl get secrets

# Get detailed information about a secret
kubectl describe secret db-credentials

# Decode secret values
kubectl get secret db-credentials -o jsonpath='{.data.username}' | base64 --decode
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 --decode

# Export secret definition to YAML
kubectl get secret db-credentials -o yaml > secret-backup.yaml
```

## Advanced Operations

### Secret Management in Pods

```bash
# Create a pod with secrets as environment variables
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: myapp
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
EOF

# Create a pod with secrets mounted as volumes
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  containers:
  - name: myapp
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secrets"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
EOF
```

### Working with Base64 Encoding

```bash
# Encode values for YAML configuration
echo -n 'admin' | base64
echo -n 'secretpassword' | base64

# Decode secret values
echo 'YWRtaW4=' | base64 --decode
echo 'c2VjcmV0cGFzc3dvcmQ=' | base64 --decode
```

## Troubleshooting

```bash
# Check secret mounting in pods
kubectl exec secret-vol-pod -- ls /etc/secrets

# Verify environment variables
kubectl exec secret-env-pod -- env | grep DB_

# Check pod events for secret-related issues
kubectl describe pod secret-env-pod
kubectl describe pod secret-vol-pod

# Validate secret permissions
kubectl auth can-i get secrets
kubectl auth can-i create secrets
```

## Production Guidelines

### Security Best Practices

```bash
# Create namespace with encrypted secrets
kubectl create namespace secure-ns
kubectl config set-context --current --namespace=secure-ns

# Apply RBAC policies
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
EOF

# Create service account with limited access
kubectl create serviceaccount restricted-sa
kubectl create rolebinding secret-reader-binding \
    --role=secret-reader \
    --serviceaccount=secure-ns:restricted-sa
```

### Cleanup Operations

```bash
# Remove secrets
kubectl delete secret db-credentials
kubectl delete secret tls-certs

# Remove pods using secrets
kubectl delete pod secret-env-pod
kubectl delete pod secret-vol-pod

# Clean up RBAC resources
kubectl delete role secret-reader
kubectl delete rolebinding secret-reader-binding
kubectl delete serviceaccount restricted-sa
```

## Learning Notes

1. Always use `--from-literal` or `--from-file` for direct secret creation
2. Implement RBAC to restrict secret access
3. Use volume mounts for sensitive files
4. Regularly rotate secrets in production
5. Monitor secret usage through audit logs

---

> 💡 **Best Practice**: Store sensitive data in Kubernetes Secrets, never in ConfigMaps or environment variables directly.

> ⚠️ **Warning**: Base64 encoding is not encryption. Always use additional security measures in production.

> 📝 **Note**: Secret values are limited to 1MB in size.