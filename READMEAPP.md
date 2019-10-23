Create an ingress controller

To create the ingress controller, use Helm to install nginx-ingress. 

# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Use Helm to deploy an NGINX ingress controller
helm install stable/nginx-ingress \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux


kubectl get service -l app=nginx-ingress --namespace ingress-basic

NAME                                             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
billowing-kitten-nginx-ingress-controller        LoadBalancer   10.0.182.160   51.145.155.210  80:30920/TCP,443:30426/TCP   20m
billowing-kitten-nginx-ingress-default-backend   ClusterIP      10.0.255.77    <none>          80/TCP                       20m
No ingress rules have been created yet. If you browse to the public IP address, the NGINX ingress controller's default 404 page is displayed.
Configure a DNS name

For the HTTPS certificates to work correctly, configure an FQDN for the ingress controller IP address. Update the following script with the IP address of your ingress controller and a unique name that you would like to use for the FQDN:

# Public IP address of your ingress controller
IP="51.145.155.210"

# Name to associate with public IP address
DNSNAME="demo-aks-ingress"

# Get the resource-id of the public ip
PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)

# Update public ip address with DNS name
az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
The ingress controller is now accessible through the FQDN.
Install cert-manager

The NGINX ingress controller supports TLS termination. There are several ways to retrieve and configure certificates for HTTPS. This article demonstrates using cert-manager, which provides automatic Lets Encrypt certificate generation and management functionality.

# Install the CustomResourceDefinition resources separately
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Label the cert-manager namespace to disable resource validation
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.8.0 \
  jetstack/cert-manager
For more information on cert-manager configuration, see the cert-manager project.
Create a CA cluster issuer To create the issuer, use the kubectl apply -f cluster-issuer.yaml command.

kubectl apply -f cluster-issuer.yaml

clusterissuer.certmanager.k8s.io/letsencrypt-staging created
Run demo applications

helm install azure-samples/aks-helloworld --namespace ingress-basic
Now install a second instance of the demo application. For the second instance, you specify a new title so that the two applications are visually distinct. You also specify a unique service name:
console


helm install azure-samples/aks-helloworld \
    --namespace ingress-basic \
    --set title="AKS Ingress Demo" \
    --set serviceName="ingress-demo"
Create an ingress route

Both applications are now running on your Kubernetes cluster, however they're configured with a service of type ClusterIP. As such, the applications aren't accessible from the internet. To make them publicly available, create a Kubernetes ingress resource. The ingress resource configures the rules that route traffic to one of the two applications.In the following example, traffic to the address https://demo-aks-ingress.eastus.cloudapp.azure.com/ is routed to the service named aks-helloworld. Traffic to the address https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two is routed to the ingress-demo service. Update the hosts and host to the DNS name you created in a previous step.

Create the ingress resource using the kubectl apply -f hello-world-ingress.yaml command.

kubectl apply -f hello-world-ingress.yaml

ingress.extensions/hello-world-ingress created
Create a certificate object

Cert-manager has likely automatically created a certificate object for you using ingress-shim, which is automatically deployed with cert-manager since v0.2.2. 


kubectl describe certificate tls-secret --namespace ingress-basic command.

To create the certificate resource, use the kubectl apply -f certificates.yaml command.

kubectl apply -f certificates.yaml

certificate.certmanager.k8s.io/tls-secret-staging created
Test the ingress configuration

Open a web browser to the FQDN of your Kubernetes ingress controller, such as https://demo-aks-ingress.eastus.cloudapp.azure.com.
As these examples use letsencrypt-staging, the issued SSL certificate is not trusted by the browser. Accept the warning prompt to continue to your application. The certificate information shows this Fake LE Intermediate X1 certificate is issued by Let's Encrypt. 
￼
