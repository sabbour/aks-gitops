# Provision infrastructure

## Resource Group

> Please check the [quotas and region availability for Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/container-service-quotas) page to select a region where AKS is supported.

Create a new Resource Group using the Azure Portal or use the Azure CLI

```sh
az group create -n "aks-gitops-rg" -l "eastus"
```

## Azure Kubernetes Service (AKS)

Please [follow the walkthrough](https://docs.microsoft.com/en-us/azure/aks/http-application-routing) to create a new AKS cluster with HTTP Application Routing enabled. Make sure to select the same **Resource Group** and **Location** specified earlier.

## Azure Container Registry (ACR)

Create the Azure Container Registry using the Azure Portal or use the Azure CLI. Make sure to select the same **Resource Group** and **Location** specified earlier.

```sh
az acr create --resource-group "aks-gitops-rg" --name "aks-gitops-acr" --sku "Standard" --location "eastus"
```

## Create trust between the AKS cluster and ACR

To establish trust between an AKS cluster and an ACR registry, you modify the Azure Active Directory Service Prinicipal used with AKS by adding the `Contributor` role to it with the scope of the ACR repository. To do so, run the following commands, replacing `<aks-rg-name>` and `<aks-cluster-name>` with the resource group and name of your AKS cluster, and `<acr-rg-nam>` and `<acr-repo-name>` with the resource group and repository name of your ACR repository with which you want to create trust.

```sh
export AKS_SP_ID=$(az aks show -g <aks-rg-name> -n <aks-cluster-name> --query "servicePrincipalProfile.clientId" -o tsv)
export ACR_RESOURCE_ID=$(az acr show -g <acr-rg-name> -n <acr-repo-name> --query "id" -o tsv)
az role assignment create --assignee $AKS_SP_ID --scope $ACR_RESOURCE_ID --role contributor
```

## Visual Studio Team Service (VSTS)

You'll use VSTS as the CI/CD tool. [Sign up for a new account](https://docs.microsoft.com/en-us/vsts/accounts/create-account-msa-or-work-student?view=vsts)/[create an account from the Azure Portal](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.TeamProject?tab=Overview) if you don't have one  already.

## Azure Storage account for ChartMuseum

[ChartMuseum](https://github.com/kubernetes-helm/chartmuseum) is a private Helm chart respository that lives in your cluster. You will need it to store the Helm charts generated as part of the build process.

You need to create an Azure Blob Storage account that will be configured in the coming steps to work with ChartMuseum.

Make sure to use a unique name.

```sh
az storage account create --name "aks-gitops-chartmuseum" --resource-group "aks-gitops-rg"--sku "Standard_LRS" --location "eastus"
```

Set default Azure storage account environment variables. You can have multiple storage accounts in your Azure subscription. To select one of them to use for all subsequent storage commands, you can set these environment variables:

```sh
export AZURE_STORAGE_ACCOUNT=<account_name>
export AZURE_STORAGE_ACCESS_KEY=<key>
```

Every blob in Azure storage must be in a container. You can create a container by using the command below.

```sh
az storage container create --name "charts"
```

## Install required Kubernetes components

### Install Helm

Helm is a Kubernetes package manager. It facilitates the installation of common applications as well as your own applications.

Follow the [Helm installation instructions](https://github.com/kubernetes/helm/blob/master/docs/install.md) to install the Helm CLI on your machine.

Initialize Helm on the cluster.

```sh
helm init --upgrade --service-account default
```

### Install ChartMuseum

[ChartMuseum](https://github.com/kubernetes-helm/chartmuseum) is a private Helm chart respository that lives in your cluster. You will need it to store the Helm charts generated as part of the build process.

Create a file named `custom.yaml` with the content below, replacing the `AZURE_STORAGE_ACCOUNT` and `AZURE_STORAGE_ACCESS_KEY` with the correct values.

```yaml
env:
  open:
    STORAGE: microsoft
    STORAGE_MICROSOFT_CONTAINER: charts
    # prefix to store charts for microsoft storage backend
    STORAGE_MICROSOFT_PREFIX:
  secret:
    AZURE_STORAGE_ACCOUNT: "********" ## azure storage account
    AZURE_STORAGE_ACCESS_KEY: "********" ## azure storage account access key
```

Then run the installation, passing in `custom.yaml`.

```sh
helm install --name chartmuseum -f custom.yaml stable/chartmuseum
```