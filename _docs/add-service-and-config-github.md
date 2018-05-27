# Adding a new service and config

This guide assumes you want to host your code on GitHub and build using Team Services. You can also follow that guide to [host your code and build it on Team Services (work in progress)]().

For each new service/application you want to build and configure to run on your Kubernetes cluster, you need to do the workflow below.

## Create service repository to host code

1. [Create](https://github.com/new) a new Github repository for the service. Make sure to check the "Initialize with README" option to be able to clone this repo locally immediately
    ![Create new repository](img/create-repo.png)

1. Add the new repository as a submodule using the Git clone URL into the `code-repos` folder
    ![Get Git clone URL](img/gitclone-url.png)

    Add the submodule to the `code-repos/color-service` folder

    `git submodule add https://github.com/sabbour/aks-gitops-color-service.git code-repos/color-service`

1. Initia

## Create the configuration repository to host Kubernetes configuration

1. [Create](https://github.com/new) a new Github repository for the service. Make sure to check the "Initialize with README" option to be able to clone this repo locally immediately

1. Add the new repository as a submodule using the Git clone URL into the `config-repos` folder

    Add the submodule to the `config-repos/color-service-config` folder

    `git submodule add https://github.com/sabbour/aks-gitops-color-service-config.git config-repos/color-service-config`