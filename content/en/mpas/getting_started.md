---
title: "Get Started with MPAS"
description: ""
lead: ""
date: 2023-09-12T10:37:58+01:00
lastmod: 2023-09-12T10:37:58+01:00
draft: true
images: []
weight: 101
toc: true
---

This tutorial shows you how to bootstrap MPAS to a Kubernetes cluster and deploy
a simple application.

## Prerequisites

- A Kubernetes cluster
- A GitHub access token with `repo` scope
- kubectl

## Objectives

- Bootstrap MPAS to a Kubernetes cluster
- Deploy a simple application

## Install the MPAS CLI

The MPAS CLI is the primary tool for interacting with MPAS. It can be used to
bootstrap MPAS to a Kubernetes cluster.

To install the MPAS CLI using `brew`:

```bash
brew install open-component-model/tap/mpas
```

For other installation methods, see the [installation guide](/mpas/overview/installation/).

## Bootstrap MPAS

### Export your GitHub access token

The MPAS CLI uses your GitHub access token to authenticate with GitHub. To create a
GitHub access token, see the [GitHub documentation](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token).

```bash
export GITHUB_TOKEN=<your-github-access-token>
export GITHUB_USER=<your-username>
```

### Bootstrap MPAS

To bootstrap MPAS to your Kubernetes cluster, run the following command. If nothing is specified it will use the KUBECONFIG specified in the user's environment. It is also possible to specify a dedicated config using the --kubeconfig option. 

```bash
mpas bootstrap github \
  --owner=$GITHUB_USER \
  --repository=mpas-bootstrap \
  --path=./clusters/my-cluster \
  --dev \
  --personal
```

This command will create a new Github repository called `mpas-bootstrap` and bootstrap
MPAS to your Kubernetes cluster. The following components will be installed:
- [Flux](https://fluxcd.io/docs/components/): A Kubernetes operator that will
  install and manage the other components.
- [ocm-controller](https://github.com/open-component-model/ocm-controller): A Kubernetes controller 
  that enables the automated deployment of software components using the Open Component Model and Flux.
- [git-controller](https://github.com/open-component-model/git-controller): A
  Kubernetes controller that will create pull requests in the target Github repository
  when changes are made to the cluster.
- [replication-controller](https://github.com/open-component-model/replication-controller): A Kubernetes controller that 
  keeps keep component versions in the cluster up-to-date with a version defined by the consumer in the `ComponentSubscription` resource. 
- [mpas-product-controller](https://github.com/open-component-model/mpas-product-controller): A Kubernetes controller, responsible for creating a product. Reconciles the `Product` resource.
- [mpas-project-controller](https://github.com/open-component-model/mpas-project-controller): A Kubernetes controller responsible for bootstrapping a whole project. Creates relevant access credentials, service accounts, roles and the main GitOps repository and 
reconciles the `Project` resource.

The output of the bootstrap is similar to the following:

```bash
Running mpas bootstrap ...
 ✓   Preparing Management repository mpas-bootstrap
 ✓   Fetching bootstrap component from ghcr.io/open-component-model/mpas-bootstrap-component
 ✓   Installing flux with version v2.1.0
 ✓   Generating git-controller manifest with version v0.7.1
 ✓   Generating mpas-product-controller manifest with version v0.3.3
 ✓   Generating mpas-project-controller manifest with version v0.1.2
 ✓   Generating ocm-controller manifest with version v0.12.2
 ✓   Generating replication-controller manifest with version v0.6.2
 ✓   Waiting for components to be ready

Bootstrap completed successfully!
```

After completing the bootstrap process, the target github repository will contain
yaml manifests for the components to be installed on the cluster and Flux will
apply all of them to get the components installed. Furthermore the installed `Flux` components will
be configured to watch the target github repository for changes in the path `./clusters/my-cluster`.

#### Registry certificate

The `--dev` flag used in the bootstrap command above will bootstrap MPAS in development mode, which means that a self-signed
certificate will be used for the MPAS components to communicate with the internal `oci` registry.

You may want to provide your own certificate for production use, for example by using [cert-manager](https://cert-manager.io/docs/usage/certificate/).
The certificate should be named `ocm-registry-tls-certs` and should be placed in the `mpas-system`
and `ocm-system` namespaces. You can use [syncing-secrets-across-namespaces](https://cert-manager.io/docs/tutorials/syncing-secrets-across-namespaces/) guide to sync the certificate between namespaces.

#### Clone the git repository

Clone the `mpas-bootstrap` repository to your local machine:

```sh
git clone https://github.com/$GITHUB_USER/mpas-bootstrap
cd mpas-bootstrap
```

### Deploy podinfo application

The [podinfo application](https://github.com/stefanprodan/podinfo) has been packaged
as an OCM component and can be retrieved from [Github](ghcr.io/open-component-model/podinfo).

1. Create a secret containing your GitHub credentials that will be used by MPAS to
create your project repository.

```bash
kubectl create secret generic \
  github-access \
  --from-literal=username=$GITHUB_USER \
  --from-literal=password=$GITHUB_TOKEN \
  -n mpas-system
```

2. Create a project that will contain the podinfo application.

Let's create a directory for the project:

```bash
mkdir -p ./clusters/my-cluster/podinfo
````

Then, create a `project.yaml` file in the `./clusters/my-cluster/podinfo` directory:

```bash
cat <<EOF >> ./clusters/my-cluster/podinfo/project.yaml
apiVersion: mpas.ocm.software/v1alpha1
kind: Project
metadata:
  name: podinfo-application
  namespace: mpas-system
spec:
  flux:
    interval: 1h
  git:
    provider: github
    owner: $GITHUB_USER
    isOrganization: false
    visibility: public
    maintainers:
    - $GITHUB_USER
    existingRepositoryPolicy: adopt
    defaultBranch: main
    credentials:
      secretRef:
        name: github-access
    commitTemplate:
      email: <MY_EMAIL>
      message: Initializing Project repository
      name: mpas-admin
  prune: true
EOF
```

Then, apply the project to the cluster in a gitOps fashion:

```bash
git add --all && git commit -m "Add podinfo project" && git push
```

`Flux` will detect the changes and apply the project to the cluster.

This will create in the cluster a `namespace` for the project, a `serviceaccount`, and RBAC.
It will also create a GitHub repository for the project, and configure `Flux` to manage the project's resources.

3. Add the needed secrets to the namespace

`Flux` is used to deploy all workloads in a gitOps way. In order for `Flux` to access the internal
registry, we have to provide the certificate to use `https`.

```bash
kubectl get secret ocm-registry-tls-certs --namespace=mpas-system -o yaml | sed 's/namespace: .*/namespace: mpas-podinfo-application/' | kubectl apply -f -
```

We also need a secret in the project namespace that will be used to communicate with github:

```bash
kubectl create secret generic \
  github-access \
  --from-literal=username=$GITHUB_USER \
  --from-literal=password=$GITHUB_TOKEN \
  -n mpas-podinfo-application
```

**Note** The credentials should have access to GitHub packages.

As part of step 2, a `serviceaccount` was created for the project. We will use this service account
to provide the necessary permissions to pull from the `ghcr` registry.

First, create a secret containing the credentials for the service account:

```bash
kubectl create secret docker-registry github-registry-key --docker-server=ghcr.io \
  --docker-username=$GITHUB_USER --docker-password=$GITHUB_TOKEN \
  --docker-email=<MY_EMAIL> -n mpas-podinfo-application
```

Then, patch the service account to use the secret:

```bash
kubectl patch serviceaccount mpas-podinfo-application -p '{"imagePullSecrets": [{"name": "github-registry-key"}]}' \
  -n mpas-podinfo-application
```

4. Clone the project repository

```bash
git clone https://github.com/$GITHUB_USER/mpas-podinfo-application
cd mpas-podinfo-application
```

5. Add the podinfo component subscription

Create a file under `./subscriptions/` that will contains the subscription declaration.

```bash
cat <<EOF >> ./subscriptions/podinfo.yaml
apiVersion: delivery.ocm.software/v1alpha1
kind: ComponentSubscription
metadata:
  name: podinfo-subscription
  namespace: mpas-podinfo-application
spec:
  interval: 30s
  component: mpas.ocm.software/podinfo
  semver: ">=v1.0.0"
  source:
    url: ghcr.io/open-component-model/mpas
    secretRef:
      name: github-access
  destination:
    url: ghcr.io/$GITHUB_USER
    secretRef:
      name: github-access
EOF
```

Then, apply the `ComponentSubscription` to the project in a gitOps fashion:

```bash
git add --all && git commit -m "Add podinfo subscription" && git push
```

`Flux` will detect the changes and apply the subscription to the cluster.

This will replicate the product referenced by the `ComponentSubscription` `spec.component` field from
defined registry in the `spec.source.url` to the `spec.destination.url` registry.

6. Add a target for the podinfo application

The target will define where the application will be installed


```bash
cat <<EOF >> ./targets/podinfo.yaml
apiVersion: mpas.ocm.software/v1alpha1
kind: Target
metadata:
  name: podinfo-kubernetes-target
  namespace: mpas-podinfo-application
  labels:
    target.mpas.ocm.software/ingress-enabled: "true" # This label is defined by the component that will use it to select an appropriate target to deploy to.
spec:
  type: kubernetes
  access:
    targetNamespace: podinfo
EOF
```

Then, apply the `Target` to the project in a gitOps fashion:

```bash
git add --all && git commit -m "Add a target for podinfo" && git push
```

`Flux` will detect the changes and apply the target to the cluster.

1. Deploy the podinfo application

In order to deploy the podinfo application, we need to create a `ProductDeploymentGenerator` resource:

```bash
cat <<EOF >> ./generators/podinfo.yaml
apiVersion: mpas.ocm.software/v1alpha1
kind: ProductDeploymentGenerator
metadata:
  name: podinfo
  namespace: mpas-podinfo-application
spec:
  interval: 1m
  serviceAccountName: mpas-podinfo-application
  subscriptionRef:
    name: podinfo-subscription
    namespace: mpas-podinfo-application
EOF
```

Then, apply the `ProductDeploymentGenerator` to the project in a gitOps fashion:

```bash
git add --all && git commit -m "Add podinfo deployment generator" && git push
```

`Flux` will detect the changes and apply the resource to the cluster.

This will create a pull request in the project repository with the `ProductDeployment` resource
that will deploy the podinfo application.

Go to the project repository and retrieve the pull request. 
It should contain a `ProductDeployment` declaration that provides the configuration and
all steps needed to deploy the product, as well as a `values.yaml` file. The `values` file
contains values that should be used to configure the different resources that are part of
the product to be deployed. There is a check that should pass before merging the pull request.

Once the pull request is merged, `Flux` will detect the changes and deploy the application to the cluster.

After a moment the `ProductDeployment` should be deployed successfully.
It is possible to verify this with the command:

```bash
k describe productdeployment -n mpas-podinfo-application  
```

The result should look something like:

```bash
Name:         podinfo
Namespace:    mpas-podinfo-application
Labels:       kustomize.toolkit.fluxcd.io/name=mpas-podinfo-application-products
              kustomize.toolkit.fluxcd.io/namespace=mpas-system
API Version:  mpas.ocm.software/v1alpha1
Kind:         ProductDeployment
Metadata:
...
Status:
  Conditions:
    Last Transition Time:  2023-09-14T10:14:41Z
    Message:               Reconciliation success
    Observed Generation:   1
    Reason:                Succeeded
    Status:                True
    Type:                  Ready
  Observed Generation:     1
```

The application is deployed in the `mpas-podinfo-application` namespace.