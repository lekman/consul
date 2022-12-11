#

The tutorial is available on the [Hashicorp Consul learning](https://developer.hashicorp.com/consul/tutorials/kubernetes-features/service-mesh-deploy) section.

## Deployment of Services

1. Start minikube if not already done.
   ```bash
   minikube start --memory 4096
   ```
1. Start Consul
    ```bash
    minikube service consul-ui --namespace consul
    ```
1. Use kubectl in a new terminal tab to interrogate server nodes
    ```bash
    kubectl exec -it consul-server-0 --namespace consul -- consul members
    ```
1. Deploy test application then check the pods
    ```bash
    kubectl apply -f hashicups/
    kubectl get pods --selector consul.hashicorp.com/connect-inject-status=injected
    ```
1. Test the application
    ```bash
    kubectl port-forward service/nginx 18080:80 --address 0.0.0.0
    ```
