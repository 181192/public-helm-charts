# akv2k8s Helm Chart (Helm 3)

This chart will install:
  * a Controller for syncing AKV secrets to Kubernetes secrets
  * a Env-Injector enabling transparent environment injection of AKV secrets into container programs

For more information and installation instructions see the official documentation at https://akv2k8s.io

## The AzureKeyVaultSecret CRD

We have removed the CRD from the Helm Chart and this must be manually installed/updated prior to installing the chart:

```
kubectl apply -f https://raw.githubusercontent.com/sparebankenvest/azure-key-vault-to-kubernetes/crd-{{ version }}/crds/AzureKeyVaultSecret.yaml
```

To use the latest CRD, run:

```
kubectl apply -f https://raw.githubusercontent.com/sparebankenvest/azure-key-vault-to-kubernetes/crd-1.1.0/crds/AzureKeyVaultSecret.yaml
```

## Configuration

### General

|               Parameter                           |                Description                   |                  Default                 |
| ------------------------------------------------- | -------------------------------------------- | -----------------------------------------|
| crds.enabled                                      | if this chart should manage the azurekeyvaultsecret crd     | true |
| crds.create                                       | if this chart should create the azurekeyvaultsecret crd | true |
| crds.keep                                         | if this chart should keep azurekeyvaultsecret crd after uninstalling the chart - note: if set to false, all azurekeyvaultsecret resources created in the cluster will automatically be removed when the crd gets removed | true | 
| rbac.create                                       | create rbac resources|true|
| rbac.podSecurityPolicies                          | any pod security policies|{}|
| keyVault.customAuth                               | if custom auth enabled | false |
| keyVault.useAuthService                           | if auth service is used - set to false to force env-injector to use local pod credentials for azure key vault | true |
| keyVault.env                                      | auth env vars used for auth with azure key vault | {}                                       |
| runningInsideAzureAks                             | if running inside azure aks - set to false if running outside aks |true |

### Controller

|               Parameter                           |                Description                   |                  Default                 |
| ------------------------------------------------- | -------------------------------------------- | -----------------------------------------|
|controller.env                                     |aditional env vars to send to pod             | {}                                       |
|controller.image.repository                        |image repo that contains the controller image | spvest/azure-keyvault-controller         |
|controller.image.tag                               |image tag                                     | 1.2.0-beta.3|
|controller.image.pullPolicy                        |pull policy                                   | IfNotPresent |
|controller.keyVaultPolling.normalInterval          |interval to wait before polling azure key vault for secret updates | 1m |
|controller.keyVaultPolling.failureInterval         |interval to wait when polling has failed `failureAttempts` before polling azure key vault for secret updates | 5m |
|controller.keyVaultPolling.failureAttempts         |number of times to allow secret updates to fail before applying `failureInterval` | 5 |
|controller.logFormat                               |log format - fmt or json | fmt                   |
|controller.logLevel                                |log level | info |
|controller.labels                                  |any additional labels | {}
|controller.podLabels                               |any additional labels | {}

### Env-Injector

|               Parameter                                     |                Description                  |                  Default                 |
| ----------------------------------------------------------- | ------------------------------------------- | -----------------------------------------|
|env_injector.enabled                                         | if the env-injector will be installed | true |
|env_injector.affinity                                        |affinities to use                            |{}                                        |
|env_injector.caBundleConfigMapName                           |name of configmap containing ca bundle       |akv2k8s-ca                |
|env_injector.certificate.useCertManager                      |use cert manager to handle webhook certificates| false|
|env_injector.certificate.custom.enabled                      |use custom certs for webhook|false|
|env_injector.certificate.custom.server.tls.crt               |custom tls cert|""|
|env_injector.certificate.custom.server.tls.key               |custom tls key|""|
|env_injector.certificate.custom.ca.crt                       |custom ca cert|""|
|env_injector.cloudConfigHostPath                             |path to azure cloud config                   |/etc/kubernetes/azure.json                |
|env_injector.dockerImageInspection.timeout                   |timeout in seconds                           |20                                        |
|env_injector.dockerImageInspection.useAksCredentialsWithACR  |will used aks credentials to pull images in acr |true|
|env_injector.env                                             |aditional env vars to send to pod            |{}                                        |
|env_injector.envImage.repository                             |image repo that contains the env image       |spvest/azure-keyvault-env                 |
|env_injector.envImage.tag                                    |image tag                                    |1.1.0                                    |
|env_injector.failurePolicy                                   |if env-injection/webhook fails, fail pod or ignore? |Fail|
|env_injector.image.pullPolicy                                |image pull policy                            |IfNotPresent                              |
|env_injector.image.repository                                |image repo that contains the controller      |spvest/azure-keyvault-webhook             |
|env_injector.image.tag                                       |image tag                                    |1.1.8                                    |
|env_injector.labels                                          |Labels to add to the webhook deployment    |{} |
|env_injector.logLevel                                        |log level - Trace, Debug, Info, Warning, Error, Fatal or Panic | Info                   |
|env_injector.logFormat                                       |log format - fmt or json | fmt                   |
|env_injector.metrics.enabled                                 |if prometheus metrics is enabled             |false                                     |
|env_injector.name                                            ||env-injector|
|env_injector.namespaceLabelSelector.enabled                  |if namespaceSelector for env-injection is enabled|true|
|env_injector.namespaceLabelSelector.label.name               |namespace selector label name|azure-key-vault-env-injection|
|env_injector.namespaceLabelSelector.label.value              |namespace selector label value|enabled|
|env_injector.nodeSelector                                    |node selector to use                         |{}                                        |
|env_injector.podDisruptionBudget.enabled                     |if pod disruption budget is enabled          |true                                      |
|env_injector.podDisruptionBudget.maxUnavailable              |pod disruption maximum unavailable           |nil                                       |
|env_injector.podDisruptionBudget.minAvailable                |pod disruption minimum available             |1                                         |
|env_injector.podLabels                                       |Labels to add to the webhook pod           |{} |
|env_injector.replicaCount                                    |number of replicas                           |2                                         |
|env_injector.resources                                       |resources to request                         |{}                                        |
|env_injector.securityContext.runAsUser                       |Which user to run processes                  |65534|
|env_injector.service.name                                    |webhook service name                         |azure-keyvault-secrets-webhook            |
|env_injector.service.type                                    |webhook service type                         |ClusterIP                                 |
|env_injector.service.externalTlsPort                         |service tls port                     |443           |
|env_injector.service.internalTlsPort                         |pod tls port                         |443               |
|env_injector.service.externalHttpPort                        |service http port for metrics and healthz|443           |
|env_injector.service.internalHttpPort                        |pod http port for metrics and healthz|443               |
|env_injector.serviceAccount.create                           |create service account?                      |true|
|env_injector.serviceAccount.name                             |name of service account                      |generated|
|env_injector.tolerations                                     |tolerations to add                           |[]                                        |
