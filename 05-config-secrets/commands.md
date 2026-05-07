# Config & Secrets - Commands

## ConfigMap Operations

```bash
# Create ConfigMap from literal
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2

# Create ConfigMap from file
kubectl create configmap my-config --from-file=config.properties

# Create ConfigMap from env file
kubectl create configmap my-config --from-env-file=config.env

# Create ConfigMap from directory
kubectl create configmap my-config --from-file=./configs/

# Get ConfigMaps
kubectl get configmaps
kubectl get cm  # shorthand

# Describe ConfigMap
kubectl describe cm my-config

# Get ConfigMap as YAML
kubectl get cm my-config -o yaml

# Edit ConfigMap
kubectl edit cm my-config

# Delete ConfigMap
kubectl delete cm my-config

# Apply from file
kubectl apply -f configmap.yaml
```

## Secret Operations

```bash
# Create generic secret from literal
kubectl create secret generic my-secret --from-literal=password=mypassword

# Create secret from file
kubectl create secret generic my-secret --from-file=ssh-privatekey=~/.ssh/id_rsa

# Create TLS secret
kubectl create secret tls tls-secret --cert=path/to/tls.crt --key=path/to/tls.key

# Create docker registry secret
kubectl create secret docker-registry my-registry \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com

# Get secrets
kubectl get secrets
kubectl get secret  # shorthand

# Describe secret (values are hidden)
kubectl describe secret my-secret

# Get secret YAML
kubectl get secret my-secret -o yaml

# Decode secret value
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode

# Edit secret
kubectl edit secret my-secret

# Delete secret
kubectl delete secret my-secret

# Apply from file
kubectl apply -f secret.yaml
```

## Viewing and Editing

```bash
# View ConfigMap data
kubectl get cm my-config -o jsonpath='{.data}'

# View specific key
kubectl get cm my-config -o jsonpath='{.data.key1}'

# View secret keys (not values)
kubectl get secret my-secret -o jsonpath='{.data}' | jq 'keys'

# Edit ConfigMap in place
kubectl edit cm my-config

# Patch ConfigMap
kubectl patch cm my-config -p '{"data":{"new-key":"new-value"}}'
```

## Using ConfigMaps and Secrets

```bash
# Create pod with ConfigMap as env var
kubectl run nginx --image=nginx --env="KEY1=value1" --dry-run=client -o yaml > pod.yaml

# Get pods using specific ConfigMap
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.containers[*].envFrom[*].configMapRef.name=="my-config")]}{.metadata.name}{"\n"}{end}'

# Get pods using specific Secret
kubectl get pods -A -o jsonpath='{range .items[?(@.spec.containers[*].envFrom[*].secretRef.name=="my-secret")]}{.metadata.name}{"\n"}{end}'
```

## Generate YAML

```bash
# Generate ConfigMap YAML
kubectl create configmap my-config --from-literal=key1=value1 --dry-run=client -o yaml > configmap.yaml

# Generate Secret YAML
kubectl create secret generic my-secret --from-literal=password=mypassword --dry-run=client -o yaml > secret.yaml

# Generate TLS Secret YAML
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key --dry-run=client -o yaml > tls-secret.yaml

# Generate Docker Registry Secret YAML
kubectl create secret docker-registry my-registry \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com \
  --dry-run=client -o yaml > docker-secret.yaml
```

## Quick Commands

```bash
# Quick ConfigMap
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  key1: value1
  key2: value2
EOF

# Quick Secret (base64 encoded)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: bXlwYXNzd29yZA==
EOF

# Quick Secret (stringData - auto encoded)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  password: mypassword
EOF
```

## Useful Queries

```bash
# All ConfigMaps in namespace
kubectl get cm -n <namespace>

# All secrets with type
kubectl get secrets -o custom-columns=NAME:.metadata.name,TYPE:.type

# Count secrets by type
kubectl get secrets -A -o json | jq -r '.items | group_by(.type) | .[] | {type: .[0].type, count: length}'

# Find ConfigMaps without labels
kubectl get cm -A -o jsonpath='{range .items[?(!.metadata.labels)]}{.metadata.name}{"\n"}{end}'

# Decode all secrets in a namespace
kubectl get secrets -n <namespace> -o json | jq -r '.items[].data | to_entries[] | "\(.key)=\(.value|@base64d)"'
```
