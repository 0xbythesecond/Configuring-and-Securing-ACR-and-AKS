# Configuring-and-Securing-ACR-and-AKS

<img src="https://github.com/0xbythesecond/Configuring-and-Securing-ACR-and-AKS/blob/main/Configuring%20and%20Securing%20ACR%20and%20AKS%20diagram.gif?raw=true" height="100%" width="100%" alt="Configuring ans Securigin Diagram"/>

## Introduction
Welcome to this lesson on configuring and securing Azure Container Registry (ACR) and Azure Kubernetes Service (AKS). In this lesson, you will learn how to effectively set up and secure your container registry and Kubernetes cluster, enabling you to deploy and manage containerized applications with confidence. By following the step-by-step tasks outlined in this lesson, you will gain practical knowledge and hands-on experience in configuring ACR and AKS to meet your organization's requirements.

## Objective
The objective of this lesson is to provide you with the necessary skills and knowledge to configure and secure Azure Container Registry and Azure Kubernetes Service. By the end of this lesson, you will be able to:

1. Create an Azure Container Registry (ACR) to store and manage container images.
2. Build container images and push them to ACR for deployment.
3. Create an Azure Kubernetes Service (AKS) cluster to orchestrate and manage containerized applications.
4. Grant necessary permissions to AKS cluster for accessing ACR and its virtual network.
5. Deploy external and internal services to AKS and verify their accessibility.<br />

By achieving these objectives, you will have a solid foundation in configuring and securing ACR and AKS, enabling you to effectively leverage these services to deploy and manage your containerized applications in a secure and scalable manner.

Configuring and Securing ACR and AKS

Estimated Timing: 45 minutes

In this lesson, you will learn how to configure and secure Azure Container Registry (ACR) and Azure Kubernetes Service (AKS). Follow the step-by-step tasks below:

<details>
  
  <summary>
    
  ### Task 1: Create an Azure Container Registry
    
  </summary>

Sign in to the Azure portal using an account with Owner or Contributor role in the Azure subscription and Global Administrator role in the Azure AD tenant.</br>
Open the Cloud Shell and ensure Bash is selected.
  
Create a new resource group for the lab:
  ```ex
  az group create --name AZ500LAB09 --location southcentralus
  ```
  
  <img src="https://github.com/0xbythesecond/Configuring-and-Securing-ACR-and-AKS/blob/main/Create%20Resource%20Group%20in%20Bash.png?raw=true" height="60%" width="60%" alt="create resource group in bash"/>
  
Verify the resource group creation: 
  ```ex
  az group list --query "[?name=='AZ500LAB09']" -o table
  ```
  
  <img src="https://github.com/0xbythesecond/Configuring-and-Securing-ACR-and-AKS/blob/main/Verify%20Resource%20Group%20has%20bee%20Created%20in%20Bash.png?raw=true" height="60%" width="60%" alt="verify resource group has been created in bash"/>
  
Create a new Azure Container Registry (ACR) instance:
  ```ex
  az acr create --resource-group AZ500LAB09 --name az500$RANDOM$RANDOM --sku Basic
  ```
  
Confirm the ACR creation: 
  ```ex
  az acr list --resource-group AZ500LAB09
  ```
  </details>
  
  #
  
  <details>
  <summary>
    
### Task 2: Create a Dockerfile, build a container, and push it to ACR
    
  </summary>

Create a Dockerfile for an Nginx-based image:
  
```ex
  echo FROM nginx > Dockerfile
```
  
Build the image and push it to ACR:
Set the ACR name:
  
  ```ex
  ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)
 ``` 
  
Build and push the image: 
  
  ```ex
  az acr build --resource-group AZ500LAB09 --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile 
  ```
  
Verify the image in ACR: <br />
Navigate to the ACR instance in the Azure portal, go to Repositories, and check the sample/nginx:v1 image.
 <img src="https://png.pngtree.com/png-vector/20210604/ourmid/pngtree-gray-network-placeholder-png-image_3416659.jpg" height="50%" width="50%" alt="sample repository"/>
  
</details>

#

<details>
  <summary>
    
  ### Task 3: Create an Azure Kubernetes Service cluster

  </summary> 
Create an AKS cluster:
  
- Search for Kubernetes services in the Azure portal. <br />
- Click + Create and select + Create a Kubernetes cluster. <br />
- Configure the cluster settings and deploy it. <br /> 
  
|Setting	| Value|
|----- | ----- |
|Subscription |	the name of the Azure subscription you are using in this lab
|Resource group |	AZ500LAB09|
|Kubernetes cluster name |	MyKubernetesCluster|
|Region |	(US) South Central US|
|Availability zones |	None|
|Scale method |	Manual|
|Node count |	1|
  
- Verify the deployment and review the resources in the created resource group.
  
  </details>
  
  #
  
  <details> 
  
  <summary> 
    
### Task 4: Grant AKS cluster permissions

  </summary> 
Grant AKS cluster permissions to access ACR: az aks update -n MyKubernetesCluster -g AZ500LAB09 --attach-acr $ACRNAME
Grant AKS cluster Contributor role to its virtual network:
Define variables:
  
  ```ex
  RG_AKS=AZ500LAB09, AKS_VNET_NAME=AZ500LAB09-vnet, AKS_CLUSTER_NAME=MyKubernetesCluster
  ```
Grant Contributor role: 
  
```ex
AKS_VNET_ID=$(az network vnet show --name $AKS_VNET_NAME --resource-group $RG_AKS --query id -o tsv) and az role assignment create --assignee $AKS_MANAGED_ID --role "Contributor" --scope $AKS_VNET_ID
```  
  
</details>

#

<details>
  <summary> 
  
### Task 5: Deploy an external service to AKS
    
  </summary>

Upload the manifest files for the external service.
  In the Bash session within the Cloud Shell pane, run the following to identify the name of the Azure Container Registry instance:

```elm
 echo $ACRNAME
```  
  
Note: Record the Azure Container Registry instance name. You will need it later in this task.

In the Bash session within the Cloud Shell pane, run the following to open the nginxexternal.yaml file, so you can edit its content.

 ```elm
 code ./nginxexternal.yaml
```
  
  >**Note**: This is the external yaml file.

In the editor pane, scroll down to line 24 and replace the <ACRUniquename> placeholder with the ACR name.

In the editor pane, in the upper right corner, click the ellipses icon, click Save and then click Close editor.

Replace the ACR name in the nginxexternal.yaml file. Apply the changes to the cluster:

```ex
 kubectl apply -f nginxexternal.yaml
``` 
  
In the Bash session within the Cloud Shell pane, review the output of the command you run in the previous task to verify that the deployment and the corresponding service have been created.

```ex
 deployment.apps/nginxexternal created
 service/nginxexternal created
```

Replace the ACR name in the nginxexternal.yaml file.
Apply the changes to the cluster: kubectl apply -f nginxexternal.yaml
  </details>
  
  #
  
  <details>
  <summary>
    
### Task 6: Verify access to the external AKS-hosted service
    
  </summary> 

Retrieve information about the nginxexternal service:
    
```elm
kubectl get service nginxexternal
```
    
Access the external service using the External-IP in a web browser.
</details>

 #
 
 <details> 
 <summary>
    
 ### Task 7: Deploy an internal service to AKS

  </summary> 
    
Edit the nginxinternal.yaml file to replace the ACR name.
In the Bash session within the Cloud Shell pane, run the following to open the nginxintenal.yaml file, so you can edit its content.
``` elm
 code ./nginxinternal.yaml
 ```  
 
Apply the changes to the cluster:
   
   ```elm
   kubectl apply -f nginxinternal.yaml
   ```
   
Retrieve information about the nginxinternal service: 
   ```elm
   kubectl get service nginx internal
   ```
  </details> 
 
  #
  <details> 
  
 <summary> 
   
 ### Task 8: Verify that you can access an internal AKS-hosted service
   
  </summary> 
Step 1: List the pods in the default namespace on the AKS cluster

Open the Bash session within the Cloud Shell pane.

Run the following command to list the pods in the default namespace on the AKS cluster:

```elm
kubectl get pods
```   
  
Locate the first entry in the NAME column of the pod listing and make note of its name. This pod will be used in the subsequent steps.

Step 2: Connect interactively to the first pod

In the same Bash session within the Cloud Shell pane, run the following command, replacing <pod_name> with the name of the pod you copied in the previous step:

```elm
kubectl exec -it <pod_name> -- /bin/bash
```
    
This command will connect you interactively to the selected pod, enabling you to run commands within its environment.

Step 3: Verify the availability of the internal service

In the Bash session within the Cloud Shell pane (connected to the pod), run the following command, replacing <internal_IP> with the IP address you recorded in the previous task:

```elm
curl http://<internal_IP>
```
    
This command will verify that the nginx web site is accessible via the private IP address of the internal service.

Step 4: Close the Cloud Shell pane

Close the Cloud Shell pane to conclude this task.

  >**Note**: You have successfully verified access to an internal AKS-hosted service within your AKS cluster. By following the steps in this lesson, you gained hands-on experience in listing pods, connecting to a specific pod, and verifying the availability of an internal service using the recorded IP address.

Clean up resources:
As a best practice, it is essential to remove any unused Azure resources to avoid unexpected costs. In this lab, you created resource groups specifically for this exercise. To clean up these resources:

Open the Azure portal and click the first icon in the top right to access the Cloud Shell.

Select PowerShell from the drop-down menu in the upper-left corner of the Cloud Shell pane and confirm when prompted.

Run the following command in the PowerShell session within the Cloud Shell pane to remove the resource groups created in this lab:

```powershell
Remove-AzResourceGroup -Name "AZ500LAB09" -Force -AsJob
```    
    
Close the Cloud Shell pane to complete the cleanup process.

By removing the unnecessary resources, you ensure a clean and cost-effective environment for future deployments.
    
</details> 


