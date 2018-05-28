# Adding a new service/config and general Git flow

This guide assumes you want to host your code on GitHub and build using Team Services. You can also follow that guide to [host your code and build it on Team Services (work in progress)]().

For each new service/application you want to build and configure to run on your Kubernetes cluster, you need to do the workflow below.

Assuming you want to create a new microservice called `color-service`, you will need to create a Git repository to host its code under `code-repos\color-service\code`. For the Kubernetes config, you'll need to create that under `code-repos\color-service\config`.

Finally, you'll setup some git branches to do some proper governance of code loosely based on what is [documented here](https://docs.microsoft.com/en-us/vsts/git/concepts/git-branching-guidance?view=vsts).

## Create service repository to host code and config

1. [Create](https://github.com/new) a new Github repository for the service. Make sure to check the "Initialize with README" option to be able to clone this repo locally immediately

1. Add the new repository as a submodule using the Git clone URL into the `code-repos` folder

    Add the submodule to the `code-repos/color-service` folder

    ```
    git submodule add https://github.com/sabbour/aks-gitops-color-service.git code-repos/color-service
    ```

## Create the `development` branch and push it

```sh
git checkout -b development
git commit --allow-empty -m "Initial commit"
git push -u origin development
```

## Work on a new feature

The flow below is loosely based on [Simplified Git Flow](https://medium.com/goodtogoat/simplified-git-flow-5dc37ba76ea8).

Branch off the `development` branch to add a new feature.

```sh
git checkout development
git pull
git checkout -b feature/coolnewfeature
git push -u origin feature/coolnewfeature
```

Add some code and commits

```sh
git commit -a -m "The best feature ever"
git push origin feature/coolnewfeature
```

When the feature is ready for review, create a Pull Request to merge `feature/coolnewfeature` into `development`.

Once the Pull Request is reviewed and is ready for merging, merge and close the Pull Request.

Finally, delete the feature branch locally and remotely.

```sh
git branch -d feature/coolnewfeature                # locally
git push origin --delete feature/coolnewfeature     # remotely
```