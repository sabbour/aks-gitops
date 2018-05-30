# Provision infrastructure

## Resource Group

> At the time of writing, AKS is in preview and is available in select regions so please choose a [supported preview region](https://github.com/Azure/AKS/blob/master/preview_regions.md).

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