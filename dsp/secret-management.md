---
title: DSP Secret Management
description: Secret Management
---

[&larr; back to Overview](/dsp)

# Secret Management

This page provides an overview of the secret management process on the Digital Solutions Platform (DSP).

---

## Secret Storage

As part of the team on-boarding process, DSP provisions an Azure Key Vault for each team to store and manage their application secrets. The Key Vault is a secure and encrypted storage solution that allows teams to store sensitive information such as API keys, passwords, and certificates.
The Key Vault is an environment-specific resource that is accessible only to the team members and applications that have been granted access.
Also, the Haremonizer uses the Key Vault to propagate secrets into the Spoke clusters.

---

## Application Secrets

Teams can store their application secrets in the Key Vault by creating a secret in the Key Vault. The secret can be created using the Azure portal or the Azure CLI.

### Reservered Secrets

As the secret store is also used by the haremonzier to store the secrets, the teams are not allowed to create secrets with the following names:

- Secret keys starting with `dsp-`


### Mounting Secrets in Kubernetes using External Secrets

In order to access the secrets in the Kubernetes cluster, we make use of the [external-secrets](https://github.com/external-secrets/external-secrets) operator. External-secrets syncs the secrets from the Azure Key Vault to the Kubernetes cluster as normal Kubernetes secrets.
In order to access the secrets in the Kubernetes cluster, teams can create [ExternalSecret](https://external-secrets.io/latest/api/externalsecret/) resource pointing to the `azure-keyvault` SecretStore in the cluster using the GitOps repository.
The ExternalSecret resource references the secret in the Key Vault and specifies the Kubernetes secret name and namespace where the secret should be created.

Following is an example of an ExternalSecret resource that moounts a secret called `salesforceApiKey` from the Key Vault to the Kubernetes cluster in a secret resource called `salesforce-api-key`. For more advanced examples on the ExternalSecret resource, refer to the [ExternalSecret API documentation](https://external-secrets.io/latest/api/externalsecret/).

{% raw %}
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: salesforce-api-key
  # labels and annotations are copied over to the
  # secret that will be created if needed
  labels:
    app.kubernetes.io/part-of: "my-app"

spec:
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore 

  # RefreshInterval is the amount of time before the values reading again from the SecretStore provider
  # Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h" (from time.ParseDuration)
  # May be set to zero to fetch and create it once
  refreshInterval: "1h"

  target:
    # The Secret template is exposed for customization
    template:
      mergePolicy: Merge # Template is merged with generated secret
      data:
        # You can use inline templates to construct your desired config file that contains your secret if needed
        config.yml: |
          salesforce:
            apiKey: {{ .salesforceApiKey }}

  # Data defines the connection between the Kubernetes Secret keys and the Provider data
  data:
    - secretKey: salesforceApiKey
      remoteRef:
        key: salesforceApiKey
```
{% endraw %}

---

## Secrets in Git

Teams are not allowed to store secrets in the Git repositories. The secrets should be stored in the Key Vault and accessed by the applications at runtime.
If there is a need to store secrets in the Git repository, the secrets **MUST** be encrypted using a tool like [SOPS](https://github.com/getsops/sops) or [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets).
If you have such a requirement, please reach out to the DSP team for guidance.