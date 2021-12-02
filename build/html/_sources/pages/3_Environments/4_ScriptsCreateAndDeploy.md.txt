<img align="right" width="100px" src="../../_images/Foundation29.png">

## 3.4. Scripts for create and deploy environments

To create and configure the different environments, a list of scripts has been implemented in the Dx29.Environments project.
In it you can find the different steps necessary to deploy from scratch each of the environments, and also how to add the file with the values or secret keys required by the different components of the application in a secure way.

In general, these scripts contain the kubectl commands necessary for the deployment of the environment and the association of the components included in it. So, for each of the environments, these are the steps that will be followed:

- Create new [resource group](https://docs.microsoft.com/en-GB/azure/azure-resource-manager/management/manage-resource-groups-portal) for the environment
- Create an AKS using [Azure CLI](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough)
>- Resource Group with resource_group_name
>- Name cluster: cluster_name
>- Node size: Choose from this [list](https://docs.microsoft.com/en-GB/azure/virtual-machines/sizes), taking into account costs:
>>- [Windows](https://azure.microsoft.com/en-gb/pricing/details/virtual-machines/windows/#Windows)
>>- [Linux](https://azure.microsoft.com/en-gb/pricing/details/virtual-machines/linux/#Linux)
- Create new Azure container registry using [Azure CLI](https://docs.microsoft.com/en-GB/azure/container-registry/container-registry-get-started-azure-cli)
>- Resource Group with resource_group_name
>- Name container registry: acr_name
>- SKU: Basic
- Using Script for configure the environment:

```
# Get credentials 
az aks get-credentials --resource-group <resource_group_name> --name <azure_aks_name>

# Only first time: Attach Azure Container Registry
#az aks update --resource-group <resource_group_name> --name <azure_aks_name> --attach-acr <azure_acr_name>


# Create a namespace for your ingress resources
#kubectl create namespace app-ingress

# Add the official stable repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace app-ingress --set controller.replicaCount=2


# Install the CustomResourceDefinition resources separately
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Label the cert-manager namespace to disable resource validation
kubectl label namespace cert-manager cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager --namespace cert-manager --version v0.11.0 jetstack/cert-manager


# Apply cluster-issuer staging
kubectl apply -f cluster-issuer.yaml


# Apply app-ingress staging
kubectl apply -f app-ingress.yaml --namespace app-ingress

# Check status
kubectl describe certificate tls-secret-pro --namespace app-ingress
```

It is important to point out that for the execution of this script it will be necessary to use the configuration files of: 
>- **app-ingress.yaml** 
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-pro
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, PATCH, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
    nginx.ingress.kubernetes.io/cors-allow-headers: "authorization, content-type, Ocp-Apim-Subscription-Key"
    nginx.ingress.kubernetes.io/proxy-body-size: "0"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
spec:
  tls:
  - hosts:
    - <Environment_URL_Address>
    secretName: tls-secret-pro
  rules:
  - host: <Environment_URL_Address>
    http:
      paths: 
      - path: /
        backend:
          serviceName: <service_that_contains_frontend_app>
          servicePort: 80

```

>- **cluster-issuer.yaml**
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-pro
  namespace: app-ingress
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: <user_letsencrypt_email>
    privateKeySecretRef:
      name: letsencrypt-pro
    solvers:
    - http01:
        ingress:
          class: nginx

```

Finally, for the creation and update of the **secrets files** in an environment, these are the steps to follow:

```
# Get credentials
az aks get-credentials --resource-group <resource_group_name> --name <azure_aks_name>

# Access to folder where you have save the file to upload in local.

# Create secret
kubectl create secret generic secret-appsettings --from-file=./appsettings.secrets.json --namespace app-ingress

# Update secret
kubectl delete secret secret-appsettings --namespace app-ingress
kubectl create secret generic secret-appsettings --from-file=./appsettings.secrets.json --namespace app-ingress
# And then deploy the components/microservices that consumes it.

```
