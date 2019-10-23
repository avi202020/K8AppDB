# Deploy a SQL Server container in Kubernetes
		
##  Prerequisites

   * Azure CLI to manage the cluster.
		
   * Kubernetes cluster has been created
		
   * kubectl CLI to manage the cluster	      

##  Create DataBase Password

```kubectl create secret generic mssql --from-literal=SA_PASSWORD="MyC0m9l&xP@ssw0rd"```

Replace MyC0m9l&xP@ssw0rd with a complex password.

##  Create storage
  Configure a persistent volume and persistent volume claim in the Kubernetes cluster. Complete the following steps:
  kubectl apply -f pvc.yaml


Verify the persistent volume claim.

  ```kubectl describe pvc <PersistentVolumeClaim> i.e kubectl describe pvc mssql-data```

Verify the persistent volume

  ```kubectl describe pv```
    
kubectl returns metadata about the persistent volume that was automatically created and bound to the persistent volume claim.

## SQL Server deployment

Create a manifest to describe the container based on the SQL Server mssql-server-linux Docker image. The manifest references the mssql-server persistent volume claim, and the mssql secret that you already applied to the Kubernetes cluster. The manifest also describes a service. This service is a load balancer. The load balancer guarantees that the IP address persists after SQL Server instance is recovered.
	   
Create a manifest (a YAML file) to describe the deployment. The following example describes a deployment, including a container based on the SQL Server container image.Copy the preceding code into a new file, named sqldeployment.yaml.  When Kubernetes deploys the container, it refers to the secret named mssql to get the value for the password.

```kubectl apply -f sqldeployment.yaml file```
	
Verify the services are running. Run the following command:kubectl get services This command returns services that are running, as well as the internal and external IP addresses for the services. Note the external IP address for the mssql-deployment service. Use this IP address to connect to SQL Server.For more information about the status of the objects in the Kubernetes cluster, run:

```az aks browse --resource-group <MyResourceGroup> --name <MyKubernetesClustername>```

## Connect to the SQL Server instance

If you configured the container as described, you can connect with an application from outside the Azure virtual network. Use the sa account and the external IP address for the service. Use the password that you configured as the Kubernetes secret.
You can use the following applications to connect to the SQL Server instance.
		
connect with sqlcmd, run the following command:

  sqlcmd -S <External IP Address> -U sa -P "MyC0m9l&xP@ssw0rd"
		Replace the following values:
     <External IP Address> with the IP address for the mssql-deployment service  yC0m9l&xP@ssw0rd with your password
		
## Verify failure and recovery

To verify failure and recovery, you can delete the pod. Do the following steps:
        
  List the pod running SQL Server.
	 
   ```kubectl get pods```

    **Note the name of the pod running SQL Server. Delete the pod.

```kubectl delete pod mssql-deployment-0```

mssql-deployment-0 is the value returned from the previous step for pod name.
Kubernetes automatically re-creates the pod to recover a SQL Server instance, and connect to the persistent storage. Use kubectl get pods to verify that a new pod is deployed. Use kubectl get services to verify that the IP address for the new container is the same.

