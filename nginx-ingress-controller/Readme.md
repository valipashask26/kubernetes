## Setup Nginx as ingress controller on k8s cluster (AKS)

This guide provides step-by-step instructions on how to set up Nginx as an Ingress controller in your Kubernetes cluster. Nginx Ingress is a popular choice for managing external access to services within a Kubernetes cluster. It allows you to define and manage routing rules, SSL/TLS termination, load balancing, and more for your applications.

### Prerequisites
Before proceeding with the setup, make sure you have the following prerequisites in place:

1) A running Kubernetes cluster: Ensure you have access to a Kubernetes cluster where you want to install the Nginx Ingress controller.
2) `kubectl` installed: Make sure you have the `kubectl` command-line tool installed on your local machine. You'll use it to interact with the Kubernetes cluster.

Now, let's proceed with the installation and configuration steps.

### Installation Steps

Follow these steps to set up Nginx as an Ingress controller in your Kubernetes cluster:

1) Install Helm

2) Add the Nginx Ingress Helm Repository:
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
3) install the nginx ingress

```sh
helm install ingress-nginx ingress-nginx/ingress-nginx \
        --namespace ingress-deployment \
        --set controller.replicaCount=2 \
        --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
        --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
        --set controller.service.externalTrafficPolicy=Local \
        --set controller.service.loadBalancerIP="<Ipaddress>"
```

**NOTE**: before executing the above step make sure you have already created a Public Ipaddress

Please verify the service with type: LoadBalancer. Wait until it displays the external IP address.

Once the setup is complete, open your web browser and enter the external IP address. If you see a "404" error, it indicates that the configuration is correctly set up.