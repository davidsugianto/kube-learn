# Advanced - Commands

## Custom Resources

```bash
# List CRDs
kubectl get crds
kubectl get customresourcedefinitions

# Describe CRD
kubectl describe crd <crd-name>

# Get CRD YAML
kubectl get crd <crd-name> -o yaml

# List custom resources
kubectl get <resource-kind>
kubectl get <resource-kind> -A

# Describe custom resource
kubectl describe <resource-kind> <resource-name>

# Delete CRD
kubectl delete crd <crd-name>
```

## Operators

```bash
# List operators (OLM)
kubectl get clusterserviceversions -A

# List operator subscriptions
kubectl get subscriptions -A

# List operator installations
kubectl get installplans -A

# View operator logs
kubectl logs -n <namespace> deployment/<operator-name>

# Check operator status
kubectl get csv -n <namespace>
```

## Istio Commands

```bash
# Install Istioctl
curl -L https://istio.io/downloadIstio | sh -

# Install Istio
istioctl install --set profile=default -y

# Enable sidecar injection
kubectl label namespace default istio-injection=enabled

# Check injection
kubectl get namespace -L istio-injection

# View Istio resources
kubectl get virtualservices
kubectl get destinationrules
kubectl get gateways

# Proxy status
istioctl proxy-status

# Analyze configuration
istioctl analyze

# Dashboard
istioctl dashboard kiali
istioctl dashboard jaeger
```

## Linkerd Commands

```bash
# Install Linkerd CLI
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh

# Check pre-requisites
linkerd check --pre

# Install Linkerd
linkerd install | kubectl apply -f -

# Verify installation
linkerd check

# Inject mesh
kubectl get deploy -o yaml | linkerd inject - | kubectl apply -f -

# View mesh status
linkerd stat deployments

# Dashboard
linkerd dashboard

# Edge
linkerd edges deployment
```

## ArgoCD Commands

```bash
# Install ArgoCD CLI
brew install argocd

# Login
argocd login <argocd-server>

# Get applications
argocd app list

# Get application details
argocd app get <app-name>

# Sync application
argocd app sync <app-name>

# Refresh application
argocd app refresh <app-name>

# Diff application
argocd app diff <app-name>

# Rollback
argocd app rollback <app-name> <revision>

# View history
argocd app history <app-name>
```

## Flux Commands

```bash
# Install Flux CLI
brew install fluxcd/tap/flux

# Check prerequisites
flux check --pre

# Bootstrap
flux bootstrap github \
  --owner=<github-username> \
  --repository=<repo-name> \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal

# Check installation
flux check

# Get sources
flux get sources git
flux get sources helm

# Get Kustomizations
flux get kustomizations

# Get HelmReleases
flux get helmreleases

# Reconcile
flux reconcile kustomization <name>
flux reconcile helmrelease <name>

# Suspend/Resume
flux suspend kustomization <name>
flux resume kustomization <name>
```

## Policy Commands

```bash
# OPA Gatekeeper
kubectl get constrainttemplates
kubectl get constraints

# Kyverno
kubectl get clusterpolicies
kubectl get policies

# Describe policy
kubectl describe clusterpolicy <policy-name>
```

## Admission Webhooks

```bash
# List validating webhooks
kubectl get validatingwebhookconfigurations

# List mutating webhooks
kubectl get mutatingwebhookconfigurations

# Describe webhook
kubectl describe validatingwebhookconfiguration <name>
kubectl describe mutatingwebhookconfiguration <name>
```

## Multi-Cluster

```bash
# Cluster API clusters
kubectl get clusters

# Machine deployments
kubectl get machinedeployments

# Machines
kubectl get machines

# Switch contexts
kubectl config use-context <context-name>

# Get contexts
kubectl config get-contexts
```

## Useful Queries

```bash
# All operators in cluster
kubectl get csv -A

# All CRDs with version
kubectl get crds -o custom-columns=NAME:.metadata.name,VERSION:.spec.versions[0].name

# Webhooks affecting pods
kubectl get validatingwebhookconfigurations -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'

# Namespaces with sidecar injection
kubectl get namespace -l istio-injection=enabled
kubectl get namespace -l linkerd.io/inject=enabled
```
