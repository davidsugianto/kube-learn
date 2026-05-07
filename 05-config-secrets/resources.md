# Config & Secrets - Resources

## Official Documentation

- [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
- [Configure a Pod to Use a Secret](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-secret/)

## ConfigMap Documentation

- [ConfigMap API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/config-map-v1/)
- [Immutable ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/#configmap-immutable)
- [Mounted ConfigMaps are Updated](https://kubernetes.io/docs/concepts/configuration/configmap/#mounted-configmaps-are-updated-automatically)

## Secret Documentation

- [Secret API](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/secret-v1/)
- [Secret Types](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)
- [Secret Security](https://kubernetes.io/docs/concepts/configuration/secret/#security-properties)
- [Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

## Managing Secrets

### External Secret Managers

- [External Secrets Operator](https://external-secrets.io/) - Sync secrets from AWS, GCP, Azure, Vault
- [HashiCorp Vault](https://www.vaultproject.io/) - Secret management
- [Vault Agent Injector](https://www.vaultproject.io/docs/platform/k8s/injector) - Inject Vault secrets into pods
- [SOPS](https://github.com/mozilla/sops) - Encrypt Kubernetes secrets

### Cloud Provider Secret Managers

- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [AWS Secrets Store CSI Driver](https://github.com/aws/secrets-store-csi-driver-provider-aws)
- [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/)
- [Azure Key Vault Provider](https://github.com/Azure/secrets-store-csi-driver-provider-azure)
- [GCP Secret Manager](https://cloud.google.com/secret-manager)
- [GCP Secret Store CSI Driver](https://github.com/GoogleCloudPlatform/secrets-store-csi-driver-provider-gcp)

## Best Practices

- [Secret Best Practices](https://kubernetes.io/docs/concepts/configuration/secret/#best-practices)
- [Encrypting Secret Data](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/security-best-practices/)

## Tutorials

- [Configure Pod ConfigMap](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-a-configmap/)
- [Inject Secret into Pod](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)
- [Redis with ConfigMap](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-a-configmap/)

## Tools

- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) - Encrypt secrets for GitOps
- [Kamus](https://github.com/Soluto/kamus) - Secrets encryption solution
- [SOPS](https://github.com/mozilla/sops) - Secrets OPerationS
- [helm-secrets](https://github.com/jkroepke/helm-secrets) - Helm plugin for secrets

## GitOps with Secrets

- [GitOps and Secrets](https://argoproj.github.io/argo-cd/user-guide/gitops_and_secrets/)
- [Flux Secrets Management](https://fluxcd.io/docs/guides/mozilla-sops/)
- [ArgoCD Vault Plugin](https://argocd-vault-plugin.readthedocs.io/)

## Security Considerations

- [Kubernetes Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)

## Books

- **Kubernetes Security** - Liz Rice, Michael Hausenblas
- **Hacking Kubernetes** - Andrew Martin, Michael Hausenblas
