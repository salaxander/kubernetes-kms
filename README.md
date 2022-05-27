# KMS Plugin for Key Vault

[![Build Status](https://dev.azure.com/AzureContainerUpstream/Kubernetes%20KMS/_apis/build/status/Kubernetes%20KMS%20CI?branchName=master)](https://dev.azure.com/AzureContainerUpstream/Kubernetes%20KMS/_build/latest?definitionId=442&branchName=master)
[![Go Report Card](https://goreportcard.com/badge/Azure/kubernetes-kms)](https://goreportcard.com/report/Azure/kubernetes-kms)
![GitHub go.mod Go version](https://img.shields.io/github/go-mod/go-version/Azure/kubernetes-kms)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/Azure/kubernetes-kms)

Enables encryption at rest of your Kubernetes data in etcd using Azure Key Vault.

From the Kubernetes documentation on [Encrypting Secret Data at Rest]:

> *[KMS Plugin for Key Vault is]* the recommended choice for using a third party tool for key management. Simplifies key rotation, with a new data encryption key (DEK) generated for each encryption, and key encryption key (KEK) rotation controlled by the user.

⚠️ **NOTE**: Currently, KMS plugin for Key Vault does not support key rotation. If you create a new key version in KMS, decryption will fail since it won't match the key used for encryption when the cluster was created.

💡 **NOTE**: To integrate your application secrets from a key management system outside of Kubernetes, use [Azure Key Vault Provider for Secrets Store CSI Driver].

## Features

* Use a key in Key Vault for etcd encryption
* Use a key in Key Vault protected by a Hardware Security Module (HSM)
* Bring your own keys
* Store secrets, keys, and certs in etcd, but manage them as part of Kubernetes

## Getting Started

### Prerequisites

💡 Make sure you have a Kubernetes cluster version 1.10 or later, the minimum version that is supported by KMS Plugin for Key Vault.

### Azure Kubernetes Service (AKS)

Azure Kubernetes Service ([AKS]) creates managed, supported Kubernetes clusters on Azure.

To try out the KMS plugin for Key Vault on AKS, follow this [doc](https://docs.microsoft.com/en-us/azure/aks/use-kms-etcd-encryption).

AKS does encrypt secrets at rest by default, but in that case keys are managed by the service and users cannot bring their own.

### Setting up KMS Plugin manually

Refer to [doc](docs/manual-install.md) for steps to setup the KMS Key Vault plugin on an existing cluster.

## Verifying that Data is Encrypted

Now that Azure KMS provider is running in your cluster and the encryption configuration is setup, it will encrypt the data in etcd. Let's verify that is working:

1. Create a new secret:

    ```bash
    kubectl create secret generic secret1 -n default --from-literal=mykey=mydata
    ```

2. Using `etcdctl`, read the secret from etcd:

    ```bash
    sudo ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/certs/ca.crt --cert=/etc/kubernetes/certs/etcdclient.crt --key=/etc/kubernetes/certs/etcdclient.key get /registry/secrets/default/secret1
    ```

3. Check that the stored secret is prefixed with `k8s:enc:kms:v1:azurekmsprovider`. This indicates the Azure KMS provider has encrypted the data.

4. Verify the secret is decrypted correctly when retrieved via the Kubernetes API:

    ```bash
    kubectl get secrets secret1 -o yaml
    ```

    The output should match `mykey: bXlkYXRh`, which is the encoded data of `mydata`.

## Rotation

Refer to [doc](docs/rotation.md) for steps to rotate the KMS Key on an existing cluster.

## Contributing

The KMS Plugin for Key Vault project welcomes contributions and suggestions. Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Code of conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Support

KMS Plugin for Key Vault is an open source project that is [**not** covered by the Microsoft Azure support policy](https://support.microsoft.com/en-us/help/2941892/support-for-linux-and-open-source-technology-in-azure). [Please search open issues here](https://github.com/Azure/kubernetes-kms/issues), and if your issue isn't already represented please [open a new one](https://github.com/Azure/kubernetes-kms/issues/new/choose). The project maintainers will respond to the best of their abilities.

[AKS]: https://azure.microsoft.com/services/kubernetes-service/
[AKS Engine]: https://github.com/Azure/aks-engine
[Azure Key Vault Data Encryption]: https://github.com/Azure/aks-engine/blob/master/docs/topics/features.md#azure-key-vault-data-encryption
[Encrypting Secret Data at Rest]: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers
[example cluster configuration]: https://github.com/Azure/aks-engine/blob/master/examples/kubernetes-config/kubernetes-keyvault-encryption.json
[Azure Key Vault Provider for Secrets Store CSI Driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure
