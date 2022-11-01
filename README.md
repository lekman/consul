# Service Mesh using Consul

This repository is for research and learning purposes only.

## Local install

The following steps were taken to run Consul locally.

1. [Install Minikube](https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe)
1. Start Minikube
   ```
   minikube start --memory 4096
   ```
1. Create [Heml values file](./deploy/helm-consul-values.yaml)
1. Install Consul in K8s
    ```
    helm repo add hashicorp https://helm.releases.hashicorp.com
    
    cd ./deploy
    
    helm install --values helm-consul-values.yaml consul hashicorp/consul --create-namespace --namespace consul --version "0.43.0"
    
    minikube service list

    minikube service consul-ui --namespace consul
    ```
1. Open a new Terminal window (if using Docker driver)
    ```
    kubectl exec --stdin --tty consul-server-0 --namespace consul -- /bin/sh
    ```
