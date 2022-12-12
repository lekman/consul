# Consul on Azure Kubernetes Service

Generate an [SSH key](./azakskey.pub).

```bash
ssh-keygen \
    -m PEM \
    -t rsa \
    -b 4096 \
    -C "azureuser@myserver" \
    -f ./azakskey \
    -N azakskeysecret
```

Create target resource group.

```bash
az group create -l uksouth -n rg-lekman-k8s-uks
```

Deploy AKS cluster using the [Bicep template](./aks.bicep).

```bash
az bicep upgrade
az deployment group create \
    --resource-group rg-lekman-k8s-uks \
    --template-file aks.bicep \
    --parameters @aks.parameters.json
```

Deploy Consul using Helm by following the [instructions](https://developer.hashicorp.com/consul/tutorials/kubernetes/kubernetes-aks-azure).

```bash
az login --use-device-code
az account list
az account set \
    --subscription corp-partner-actionpack
az aks get-credentials \
    --resource-group rg-lekman-k8s-uks \
    --name aks-lekman-dv-uks
kubectl cluster-info
helm repo add \
    hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install \
    --values consul.yaml \
    consul hashicorp/consul \
    --create-namespace --namespace \
    consul --version "1.0.0"
kubectl get pods --namespace consul
export CONSUL_HTTP_TOKEN=$(kubectl get --namespace consul secrets/consul-bootstrap-acl-token --template={{.data.token}} | base64 -d)
export CONSUL_HTTP_ADDR=https://$(kubectl get services/consul-ui --namespace consul -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export CONSUL_HTTP_SSL_VERIFY=false
consul members
open $CONSUL_HTTP_ADDR
```

The Consul UI is opened. Click "Login" top right, use the token from below command.

```bash
echo $CONSUL_HTTP_TOKEN
```

Test deploying an application.

```bash
cd ..
kubectl apply -f hashicups/
    kubectl get pods --selector consul.hashicorp.com/connect-inject-status=injected
```

Tear down service.

```bash
az group delete --resource-group rg-lekman-k8s-uks
```