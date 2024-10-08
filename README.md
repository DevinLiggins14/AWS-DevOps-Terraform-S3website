# AWS-DevOps-Terraform-S3website
<h2>Description</h2>
<br/> In this project we will create a static website in AWS using Terraform
<br />
<br/> <br/>
<img src="https://github.com/user-attachments/assets/6cc07b09-9d87-4aba-9c37-b93f70bbd430"/>


<h2>Languages and Utilities Used</h2>

Bash, Az-cli, Kubectl, VSCode, Helm (Optional: cloudshell)

<h2>Environments Used </h2>

- <b>CentOS Linux release 8.5.2111
 10, Azure portal </b>

<h2>Project walk-through:</h2>
<br/>
<p align="center">

 ##  Step 1: Create AKS Cluster

### **Prerequisites**  
- Download and Install [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).  
- Setup and configure Azure CLI using the `az login` command.  
- Install and configure `kubectl` as mentioned [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

```bash
# Create a resource group
az group create --name aks-demo-rg --location eastus

# Create the AKS cluster
az aks create --resource-group aks-demo-rg --name aks-demo --location eastus2 --node-count 2 --enable-managed-identity --generate-ssh-keys
```

<img src="https://github.com/user-attachments/assets/ac86358c-104c-48b6-a506-a0d3658833bf"/>
<img src="https://github.com/user-attachments/assets/2724b032-6e8a-45b1-ad2d-5554ba06f801"/>

### **Connect to AKS Cluster**  
```bash
# Login to your azure account
az login
# Set the cluster subscription
az account set --subscription Enter subscription Here
# Download cluster credentials
az aks get-credentials --resource-group aks-demo-rg --name aks-demo --overwrite-existing

```
<img src="https://github.com/user-attachments/assets/3c3af067-95d3-458f-a90e-e046da1dcdac"/>

<br/>  Make sure clock is syncronized if error encountered and confirm <br/>
