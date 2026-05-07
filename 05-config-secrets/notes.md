# Configuration and Secrets

Managing application configuration and sensitive data in Kubernetes.

## ConfigMap

Store non-confidential configuration data.

### Create from Literal

```bash
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
```

### Create from File

```bash
kubectl create configmap my-config --from-file=config.txt
kubectl create configmap my-config --from-file=path/to/config/
```

### Create from YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db:5432"
  api_key: "12345"
  config.json: |
    {
      "key": "value",
      "nested": {
        "item": "data"
      }
    }
```

## Using ConfigMaps

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    envFrom:
    - configMapRef:
        name: app-config
```

### As Volume Mount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: config.json
        path: config.json
```

### Immutable ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
immutable: true
data:
  key: value
```

## Secret

Store sensitive data (base64 encoded).

### Create from Literal

```bash
kubectl create secret generic my-secret --from-literal=password=mypassword
```

### Create from File

```bash
kubectl create secret generic my-secret --from-file=ssh-privatekey=/path/to/key
```

### Create from YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=      # base64 encoded
  password: MWYyZDFlMmU2N2Rm
# Or use stringData for plain text (will be encoded)
stringData:
  username: admin
  password: mypassword
```

### Secret Types

| Type | Usage |
|------|-------|
| Opaque | Arbitrary user data (default) |
| kubernetes.io/service-account-token | Service account token |
| kubernetes.io/dockercfg | Docker config |
| kubernetes.io/dockerconfigjson | Docker config JSON |
| kubernetes.io/basic-auth | Basic auth credentials |
| kubernetes.io/ssh-auth | SSH auth |
| kubernetes.io/tls | TLS certificate |

### TLS Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi...
  tls.key: LS0tLS1CRUdJTi...
```

Or using kubectl:

```bash
kubectl create secret tls tls-secret --cert=path/to/tls.crt --key=path/to/tls.key
```

## Using Secrets

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    envFrom:
    - secretRef:
        name: db-secret
```

### As Volume Mount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

### Secret Volume Permissions

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
    defaultMode: 0400  # Read-only for owner
```

## Immutable Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
immutable: true
data:
  key: dmFsdWU=
```

## Security Best Practices

1. **Use RBAC** - Limit who can read secrets
2. **Encrypt at rest** - Enable encryption in etcd
3. **Use external secret managers** - Vault, AWS Secrets Manager, etc.
4. **Minimize secret access** - Least privilege principle
5. **Don't log secrets** - Be careful with log output
6. **Use immutable secrets** - Prevent accidental modification
7. **Rotate secrets** - Regularly update credentials

## External Secret Operators

### External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-external-secret
spec:
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: my-secret
  data:
  - secretKey: password
    remoteRef:
      key: /myapp/password
```

### Popular Options

- [External Secrets Operator](https://external-secrets.io/)
- [HashiCorp Vault](https://www.vaultproject.io/)
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/)
- [GCP Secret Manager](https://cloud.google.com/secret-manager)

## ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|---------|-----------|--------|
| Purpose | Non-sensitive config | Sensitive data |
| Encoding | Plain text | Base64 |
| Size limit | 1 MiB | 1 MiB |
| Encryption | No | Can be encrypted at rest |

## Environment Variables vs Volumes

| Method | Pros | Cons |
|--------|------|------|
| Env vars | Easy to use, simple updates | Limited size, visible in process list |
| Volume mount | File-based config, larger data | Requires pod restart for updates (if immutable) |

## Next Steps

Proceed to [06-security](../06-security) to learn about Kubernetes security.
