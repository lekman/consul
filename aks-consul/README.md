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

Deploy AKS cluster using the [Bicep template](./aks.bicep).

Deploy Consul using Helm by following the [instructions](https://developer.hashicorp.com/consul/tutorials/kubernetes/kubernetes-aks-azure).

```bash
az login --use-device-code
az account list
az account set \
    --subscription corp-partner-actionpack
az aks get-credentials \
    --resource-group rg-lekman-web \
    --name aks-lekman-dv-uks
kubectl cluster-info
helm repo add \
    hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install \
    --values values.yaml \
    consul hashicorp/consul \
    --create-namespace --namespace \
    consul --version "1.0.0"
kubectl get pods --namespace consul
export CONSUL_HTTP_TOKEN=$(kubectl get --namespace consul secrets/consul-bootstrap-acl-token --template={{.data.token}} | base64 -d)
echo $CONSUL_HTTP_TOKEN 
export CONSUL_HTTP_ADDR=https://$(kubectl get services/consul-ui --namespace consul -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo $CONSUL_HTTP_ADDR 
export CONSUL_HTTP_SSL_VERIFY=false
consul members
```

Go to the EKS service in Azure. Locate the public IP of the Consul UI service from the "Services and ingress" plane. Open the HTTPS IP address in the browser. Click "Login" top right, use the token from below command.

```bash
kubectl get secrets/bootstrap-token \
    --template={{.data.token}} | base64 --decode
```

Test deploying an application.

```bash
cd ..
kubectl apply -f hashicups/
    kubectl get pods --selector consul.hashicorp.com/connect-inject-status=injected
```
