# Advanced Topics

Advanced Kubernetes concepts and patterns.

## Custom Resource Definitions (CRD)

Extend the Kubernetes API.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronSpec:
                type: string
              image:
                type: string
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames: ["ct"]
```

### Using Custom Resources

```yaml
apiVersion: stable.example.com/v1
kind: CronTab
metadata:
  name: my-cron-object
spec:
  cronSpec: "* * * * */10"
  image: my-cron-image
```

## Controllers and Operators

### Custom Controller

Watches resources and takes action.

```go
// Controller pseudo-code
func (c *Controller) Run() {
    for {
        // Watch for events
        event := c.queue.Get()
        // Reconcile desired state
        c.reconcile(event)
    }
}
```

### Operator Pattern

Operators encode human operational knowledge.

**Components:**
- Custom Resource Definition (CRD)
- Controller
- Reconciliation logic

### Operator SDK

Build operators without starting from scratch.

```bash
# Install Operator SDK
curl -LO https://github.com/operator-framework/operator-sdk/releases/latest/download/operator-sdk_linux_amd64
chmod +x operator-sdk_linux_amd64

# Create new operator
operator-sdk init --domain example.com --repo github.com/example/memcached-operator
operator-sdk create api --group cache --version v1alpha1 --kind Memcached --resource --controller
```

### Kubebuilder

Framework for building Kubernetes APIs.

```bash
# Install kubebuilder
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod +x kubebuilder

# Initialize project
kubebuilder init --domain my.domain --repo my.domain/guestbook
kubebuilder create api --group webapp --version v1 --kind Guestbook
```

## Service Mesh

Advanced networking, security, and observability.

### Istio

```yaml
# Install Istio
istioctl install --set profile=default -y

# Enable sidecar injection
kubectl label namespace default istio-injection=enabled
```

**Traffic Management:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
```

### Linkerd

```bash
# Install Linkerd CLI
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh

# Install Linkerd
linkerd install | kubectl apply -f -

# Inject mesh
kubectl get deploy -o yaml | linkerd inject - | kubectl apply -f -
```

## GitOps

Declarative infrastructure and applications from Git.

### ArgoCD

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get initial password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
```

### Flux

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Bootstrap Flux
flux bootstrap github \
  --owner=<github-username> \
  --repository=<repo-name> \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

**GitRepository:**
```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/stefanprodan/podinfo
  ref:
    branch: master
```

## Multi-Cluster

### Federation

Manage multiple clusters from one place.

```yaml
# KubeFed Core
apiVersion: core.kubefed.io/v1beta1
kind: KubeFedCluster
metadata:
  name: cluster1
  namespace: kube-federation-system
spec:
  apiEndpoint: https://cluster1.example.com
```

### Cluster API

Provision and manage clusters declaratively.

```yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: my-cluster
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"]
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
    kind: AWSCluster
    name: my-cluster
```

### Multi-Cluster Service

```yaml
apiVersion: multicluster.x-k8s.io/v1alpha1
kind: ServiceExport
metadata:
  name: my-service
```

## Admission Controllers

Intercept requests to the API server.

### ValidatingAdmissionWebhook

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: my-webhook
webhooks:
- name: my-webhook.example.com
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
  clientConfig:
    service:
      name: webhook-service
      namespace: default
      path: "/validate"
```

### MutatingAdmissionWebhook

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: my-mutating-webhook
webhooks:
- name: my-mutating-webhook.example.com
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
  clientConfig:
    service:
      name: webhook-service
      namespace: default
      path: "/mutate"
```

## Policy Engines

### OPA Gatekeeper

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }
```

### Kyverno

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: enforce
  rules:
  - name: check-for-labels
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "label 'app.kubernetes.io/name' is required"
      pattern:
        metadata:
          labels:
            app.kubernetes.io/name: "?*"
```

## Extensibility Patterns

### Scheduler Plugins

Customize pod scheduling.

### CSI Drivers

Container Storage Interface for custom storage.

### CNI Plugins

Container Network Interface for custom networking.

### Device Plugins

Expose resources like GPUs to containers.

## Summary

| Topic | Use Case | Tools |
|-------|----------|-------|
| CRDs | Extend Kubernetes API | kubebuilder, operator-sdk |
| Operators | Automate operations | Operator SDK, KUDO |
| Service Mesh | Traffic, security, observability | Istio, Linkerd, Cilium |
| GitOps | Declarative deployments | ArgoCD, Flux |
| Multi-Cluster | Multiple cluster management | Cluster API, KubeFed |
| Policy | Governance & compliance | OPA Gatekeeper, Kyverno |

## Congratulations

You've completed the Kubernetes Learning Path! Continue practicing and exploring these concepts in real scenarios.
