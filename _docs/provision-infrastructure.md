# Provision infrastructure

## Resource Group

> At the time of writing, AKS is in preview and is available in select regions so please choose **East US** or **West Europe**.

Create a new Resource Group using the Azure Portal or use the Azure CLI

```sh
az group create -n "aks-gitops-rg" -l "eastus"
```

## Azure Kubernetes Service (AKS)

At the time of writing, to make use of the new features such as monitoring integration, HTTP application routing and custom vnets, you need to either use the portal or deploy using an ARM template.

We're not going to rehash existing documentation, so just [follow the walkthrough](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal) to create a new AKS cluster. Make sure to select the same **Resource Group** and **Location** specified earlier.

You don't need to run the application in the walkthrough.

## Azure Container Registry (ACR)

Create the Azure Container Registry using the Azure Portal or use the Azure CLI. Make sure to select the same **Resource Group** and **Location** specified earlier.

```sh
az acr create --resource-group "aks-gitops-rg" --name "aks-gitops-acr" --sku "Standard" --location "eastus"
```

## Visual Studio Team Service (VSTS)

You'll use VSTS as the CI/CD tool [Sign up for a new account](https://docs.microsoft.com/en-us/vsts/accounts/create-account-msa-or-work-student?view=vsts)/[create an account from the Azure Portal](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.TeamProject?tab=Overview) if you don't have one  already.

## Azure Storage account for ChartMuseum

[ChartMuseum](https://github.com/kubernetes-helm/chartmuseum) is a private Helm chart respository that lives in your cluster. You will need it to store the Helm charts generated as part of the build process.

You need to create an Azure Storage account that will be configured in the coming steps to work with ChartMuseum.

## Install required Kubernetes components

### Install ChartMuseum

[ChartMuseum](https://github.com/kubernetes-helm/chartmuseum) is a private Helm chart respository that lives in your cluster. You will need it to store the Helm charts generated as part of the build process.