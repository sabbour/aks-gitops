# Example guidance on setting up a Git driven AKS deployment pipeline

This is intended as a demo/guide on how to create a Git driven Azure Kubernetes Service (AKS) deployment pipeline.

## What is this

When it comes to setting up a Continuous Integration/Continuous Delivery pipeline for Kubernetes, there are many tutorials and opinions that exist in the community.

This repository is yet another opinionated way of setting this up, that's to say, there are surely better/smarter ways to do this, and you might not agree to this pipeline. Not all of the below might be applicable to your project and feel free to adapt it to your needs.

Many of the ideas presented here are a combination of what others have done, including [Weaveworks](https://www.weave.works/blog/gitops-operations-by-pull-request) and [Jenkins X](https://jenkins.io/blog/2018/03/19/introducing-jenkins-x/) but are adopted to use Microsoft Azure native services such as [Azure Container Service Build](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-build-overview) and [Team Services](https://www.visualstudio.com/team-services/).

 I'm open to discussion and feedback so please open an issue if you need to discuss.

## Project structure

    .
    ├── code-repos                     # Container for Git submodules for standalone services/apps
    │   ├── foo-service                # Git submodule for a foo service
    │   ├── bar-service                # Git submodule for a bar service
    ├── config-repos                   # Container for Git submodules for K8s resources of services/apps
    │   ├── foo-service-config         # Git submodule for a foo service K8s resources
    │   ├── bar-service-config         # Git submodule for a bar service K8s resources
    ├── infrastructure                 # ARM templates to provision required Azure infrastructure
    │   ├── scripts                    # Helper scripts to deploy infrastructure
    ├── LICENSE
    └── README.md

## Some background on design choices

### Why this project structure

There are some distinct features of this folder structure:

1. Using separate Git repositories per microservice/app and linking them all to the main repository (this one) using Git submodules

    > Allows you to have cleaner build pipelines per service without having to fiddle around with path filters in order to build only a specific folder when changes show up there. There are two schools here; mono repository and multiple repository. I chose the latter, but feel free to read and [make](https://medium.com/@somakdas/code-repository-for-micro-services-mono-repository-or-multiple-repositories-d9ad6a8f6e0e) [your](http://blog.shippable.com/our-journey-to-microservices-and-a-mono-repository) [own](http://www.gigamonkeys.com/mono-vs-multi/) choice.

1. For each microservice/app, using separate Git repositories for code and config

    > The latest build of the application and what is deployed on the Kubernetes cluster are not necessarily the same thing. Storing both the source code and config (Helm charts or pure YAML) in the same repository strongly couples them together. If you need to change the configuration, to say, add a load balancer, you will be potentially forced to do a full rebuild of the application's Docker image, tagging it with a new version and pushing it to the registry while the code didn't actually change. For that reason, I chose to host the config in its own repository. Also have a look at [this](https://blog.turbinelabs.io/deploy-not-equal-release-part-one-4724bc1e726b).

1. Use Infrastructure as Code to provision the cluster and related artifacts

    > In the `infrastructure` folder, there are a bunch of Azure Resource Manager (ARM) templates that can be used to stand-up and update the infrastructure required (Kubernetes cluster, Azure Container Registry, Virtual Networks, etc.). In general, that is a good practice because it allows you to treat your infrastructure as cattle. If tomorrow you need to spin up in a new Azure region, you can do so easily. If you go the extra mile and setup a Continuous Delivery pipeline on this folder, you can ensure that no infrastructure configuration drift can happen.

## Pipeline flow

<video width="320" height="240" controls>
  <source src="_docs/img/GitOps.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Guide

This is split into smaller guides, so as to keep the size of this page manageable.

- [Provision the infrastructure](_docs/provision-infrastructure.md)
- [Creating a new service and config](_docs/add-service-and-config-github.md)
- [Workflow to work with Git and pull requests](_docs/branch-pr-github.md)
- [Setting up CI service code pipeline](_docs/ci-service-pipeline.md)
- [Setting up CI config pipeline](_docs/ci-config-pipeline.md)
- [Setting up CD config pipeline](_docs/cd-config-pipeline.md)
- [Demo (testing the pipeline workflow)](_docs/demo.md)


## Areas of improvement

This space is rapidly moving and new tools and best practices keep popping up. The intention is to try and keep this up to date and as relevant as possible.

Below is an unordered checklist of areas I think could be improved/augmented in this pipeline:

- [ ] Document the *inner loop* of the developer's experience and integrate [Azure Dev Spaces](https://docs.microsoft.com/en-us/azure/dev-spaces/azure-dev-spaces)

- [ ] Automatically creating Kubernetes environments (namespaces) for Pull Request

- [ ] Integrating with the AKS HTTP Application Routing to create DNS names per environment (prod, dev, feature).

- [ ] Document how this works with something like [GitFlow](https://datasift.github.io/gitflow/IntroducingGitFlow.html)

- [ ] Add container security scanning solutions into the pipeline ([clair](https://github.com/coreos/clair), [anchore](https://anchore.freshdesk.com/support/solutions/articles/36000060726-installing-anchore-using-helm), ..)

- [ ] [Best practices](https://blogs.msdn.microsoft.com/stevelasker/2018/03/01/docker-tagging-best-practices-for-tagging-and-versioning-docker-images/) for Docker image tagging

- [ ] Pulling secrets from Azure Key Vault and [injecting them during the release pipeline](https://docs.microsoft.com/en-us/vsts/build-release/concepts/library/variable-groups?view=vsts)

- [ ] How to do integration testing?

- [ ] Provision the infrastructure using ARM templates in the `infrastructure` folder

## Cloning this repository

This repository uses [submodules](https://github.com/blog/2104-working-with-submodules).

Git expects us to explicitly ask it to download the submodule's content. You can use `git submodule update --init --recursive` here as well, but if you're cloning this repository for the first time, you can use a modified clone command to ensure you download everything, including any submodules:

```sh
git clone --recursive git@github.com:sabbour/aks-gitops.git
```
